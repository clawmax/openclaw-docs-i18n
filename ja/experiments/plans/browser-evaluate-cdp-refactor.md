

  実験

  
# ブラウザ Evaluate CDP リファクタリング

## 背景

`act:evaluate` は、ユーザーが提供した JavaScript をページ内で実行します。現在は Playwright (`page.evaluate` または `locator.evaluate`) を介して実行されています。Playwright はページごとに CDP コマンドを直列化するため、固まった、または長時間実行される evaluate はページコマンドキューをブロックし、そのタブでの後続のすべてのアクションが「固まった」ように見える原因となります。PR #13498 は実用的な安全策（制限付き evaluate、中止伝播、ベストエフォートでの回復）を追加しました。このドキュメントは、`act:evaluate` を Playwright から本質的に分離し、固まった evaluate が通常の Playwright 操作を妨げないようにする、より大規模なリファクタリングについて説明します。

## 目標

-   `act:evaluate` が同じタブでの後続のブラウザアクションを恒久的にブロックできないようにする。
-   タイムアウトがエンドツーエンドで単一の信頼できる情報源となり、呼び出し元が予算を頼りにできるようにする。
-   HTTP とプロセス内ディスパッチの両方で、中止とタイムアウトが同じ方法で扱われるようにする。
-   Playwright からすべてを切り替えることなく、evaluate のための要素ターゲティングをサポートする。
-   既存の呼び出し元とペイロードに対する後方互換性を維持する。

## 非目標

-   すべてのブラウザアクション（クリック、タイプ、待機など）を CDP 実装に置き換えること。
-   PR #13498 で導入された既存の安全策を削除すること（有用なフォールバックとして残す）。
-   既存の `browser.evaluateEnabled` ゲートを超える新しい安全でない機能を導入すること。
-   evaluate のためのプロセス分離（ワーカープロセス/スレッド）を追加すること。このリファクタリング後も回復困難な固まった状態が見られる場合、それはフォローアップのアイデアとする。

## 現在のアーキテクチャ（固まる理由）

大まかに説明すると：

-   呼び出し元は `act:evaluate` をブラウザ制御サービスに送信する。
-   ルートハンドラは JavaScript を実行するために Playwright を呼び出す。
-   Playwright はページコマンドを直列化するため、終了しない evaluate はキューをブロックする。
-   固まったキューは、そのタブでの後続のクリック/タイプ/待機操作がハングしているように見えることを意味する。

## 提案アーキテクチャ

### 1. デッドライン伝播

単一の予算概念を導入し、そこからすべてを導出する：

-   呼び出し元が `timeoutMs`（または将来のデッドライン）を設定する。
-   外部リクエストのタイムアウト、ルートハンドラロジック、およびページ内の実行予算はすべて同じ予算を使用し、直列化オーバーヘッドに必要な小さな余裕を持たせる。
-   中止は `AbortSignal` としてどこにでも伝播され、キャンセルが一貫性を持つようにする。

実装方針：

-   小さなヘルパー（例：`createBudget({ timeoutMs, signal })`）を追加し、以下を返す：
    -   `signal`: リンクされた AbortSignal
    -   `deadlineAtMs`: 絶対的なデッドライン
    -   `remainingMs()`: 子操作のための残り予算
-   このヘルパーを以下で使用する：
    -   `src/browser/client-fetch.ts`（HTTP およびプロセス内ディスパッチ）
    -   `src/node-host/runner.ts`（プロキシパス）
    -   ブラウザアクション実装（Playwright および CDP）

### 2. 分離された Evaluate エンジン（CDP パス）

Playwright のページごとのコマンドキューを共有しない、CDP ベースの evaluate 実装を追加する。重要な特性は、evaluate トランスポートが別個の WebSocket 接続であり、ターゲットにアタッチされた別個の CDP セッションであること。実装方針：

-   新しいモジュール、例：`src/browser/cdp-evaluate.ts` を追加し、以下を行う：
    -   設定された CDP エンドポイント（ブラウザレベルのソケット）に接続する。
    -   `Target.attachToTarget({ targetId, flatten: true })` を使用して `sessionId` を取得する。
    -   以下を実行する：
        -   ページレベルの evaluate の場合は `Runtime.evaluate`、または
        -   要素 evaluate の場合は `DOM.resolveNode` と `Runtime.callFunctionOn`。
    -   タイムアウトまたは中止時：
        -   セッションに対してベストエフォートで `Runtime.terminateExecution` を送信する。
        -   WebSocket を閉じ、明確なエラーを返す。

注意点：

-   これでも JavaScript はページ内で実行されるため、終了には副作用がある可能性がある。利点は、Playwright キューを妨げず、CDP セッションを終了することでトランスポート層でキャンセル可能であること。

### 3. 参照ストーリー（完全な書き換えなしでの要素ターゲティング）

難しい部分は要素ターゲティングです。CDP は DOM ハンドルまたは `backendDOMNodeId` を必要としますが、現在ほとんどのブラウザアクションはスナップショットからの参照に基づく Playwright ロケーターを使用しています。推奨されるアプローチ：既存の参照を保持するが、オプションで CDP で解決可能な ID を付与する。

#### 3.1 保存された参照情報の拡張

保存されたロール参照メタデータを拡張し、オプションで CDP ID を含める：

-   現在：`{ role, name, nth }`
-   提案：`{ role, name, nth, backendDOMNodeId?: number }`

これにより、既存の Playwright ベースのアクションがすべて機能し続け、`backendDOMNodeId` が利用可能な場合に CDP evaluate が同じ `ref` 値を受け入れられるようになります。

#### 3.2 スナップショット作成時に backendDOMNodeId を設定

ロールスナップショットを生成する際：

1.  現在と同じように既存のロール参照マップ（role, name, nth）を生成する。
2.  CDP を介して AX ツリーを取得し（`Accessibility.getFullAXTree`）、同じ重複処理ルールを使用して `(role, name, nth) -> backendDOMNodeId` の並列マップを計算する。
3.  現在のタブの保存された参照情報に ID をマージし戻す。

参照のマッピングが失敗した場合は、`backendDOMNodeId` を未定義のままにする。これにより、この機能はベストエフォートで安全にロールアウトできる。

#### 3.3 参照付きでの Evaluate 動作

`act:evaluate` において：

-   `ref` が存在し、`backendDOMNodeId` を持つ場合、CDP を介して要素 evaluate を実行する。
-   `ref` が存在するが `backendDOMNodeId` を持たない場合、Playwright パスにフォールバックする（安全策付き）。

オプションのエスケープハッチ：

-   高度な呼び出し元（およびデバッグ用）のために、リクエスト形状を拡張して `backendDOMNodeId` を直接受け入れられるようにし、`ref` を主要インターフェースとして維持する。

### 4. 最後の手段の回復パスを保持する

CDP evaluate を使用しても、タブや接続を妨げる他の方法があります。既存の回復メカニズム（実行終了 + Playwright 切断）を、以下の場合の最後の手段として保持する：

-   レガシー呼び出し元
-   CDP アタッチがブロックされている環境
-   予期しない Playwright のエッジケース

## 実装計画（単一イテレーション）

### 成果物

-   Playwright のページごとのコマンドキューの外部で実行される、CDP ベースの evaluate エンジン。
-   呼び出し元とハンドラが一貫して使用する、単一のエンドツーエンドのタイムアウト/中止予算。
-   要素 evaluate のためにオプションで `backendDOMNodeId` を運ぶことができる参照メタデータ。
-   `act:evaluate` は可能な場合は CDP エンジンを優先し、不可能な場合は Playwright にフォールバックする。
-   固まった evaluate が後続のアクションを妨げないことを証明するテスト。
-   失敗とフォールバックを可視化するログ/メトリクス。

### 実装チェックリスト

1.  共有「予算」ヘルパーを追加し、`timeoutMs` + 上流の `AbortSignal` を以下にリンクする：
    -   単一の `AbortSignal`
    -   絶対的なデッドライン
    -   下流操作のための `remainingMs()` ヘルパー
2.  すべての呼び出し元パスを更新してこのヘルパーを使用し、`timeoutMs` がどこでも同じ意味を持つようにする：
    -   `src/browser/client-fetch.ts`（HTTP およびプロセス内ディスパッチ）
    -   `src/node-host/runner.ts`（ノードプロキシパス）
    -   `/act` を呼び出す CLI ラッパー（`browser evaluate` に `--timeout-ms` を追加）
3.  `src/browser/cdp-evaluate.ts` を実装する：
    -   ブラウザレベルの CDP ソケットに接続する
    -   `Target.attachToTarget` で `sessionId` を取得する
    -   ページ evaluate のために `Runtime.evaluate` を実行する
    -   要素 evaluate のために `DOM.resolveNode` + `Runtime.callFunctionOn` を実行する
    -   タイムアウト/中止時：ベストエフォートで `Runtime.terminateExecution` を実行し、ソケットを閉じる
4.  保存されたロール参照メタデータを拡張し、オプションで `backendDOMNodeId` を含める：
    -   Playwright アクションのための既存の `{ role, name, nth }` 動作を保持する
    -   CDP 要素ターゲティングのための `backendDOMNodeId?: number` を追加する
5.  スナップショット作成中に `backendDOMNodeId` を設定する（ベストエフォート）：
    -   CDP を介して AX ツリーを取得する（`Accessibility.getFullAXTree`）
    -   `(role, name, nth) -> backendDOMNodeId` を計算し、保存された参照マップにマージする
    -   マッピングが曖昧または欠落している場合、ID を未定義のままにする
6.  `act:evaluate` ルーティングを更新する：
    -   `ref` がない場合：常に CDP evaluate を使用する
    -   `ref` が `backendDOMNodeId` に解決される場合：CDP 要素 evaluate を使用する
    -   それ以外の場合：Playwright evaluate にフォールバックする（依然として制限付きで中止可能）
7.  既存の「最後の手段」回復パスをデフォルトパスではなくフォールバックとして保持する。
8.  テストを追加する：
    -   固まった evaluate が予算内でタイムアウトし、次のクリック/タイプが成功する
    -   中止が evaluate をキャンセルし（クライアント切断またはタイムアウト）、後続のアクションのブロックを解除する
    -   マッピング失敗が Playwright に適切にフォールバックする
9.  可観測性を追加する：
    -   evaluate 継続時間とタイムアウトカウンター
    -   terminateExecution の使用状況
    -   フォールバック率（CDP -> Playwright）とその理由

### 受け入れ基準

-   意図的にハングさせた `act:evaluate` が呼び出し元の予算内で返り、後続のアクションのためにタブを妨げない。
-   `timeoutMs` が CLI、エージェントツール、ノードプロキシ、およびプロセス内呼び出し全体で一貫して動作する。
-   `ref` が `backendDOMNodeId` にマッピングできる場合、要素 evaluate は CDP を使用する。それ以外の場合、フォールバックパスは依然として制限付きで回復可能である。

## テスト計画

-   単体テスト：
    -   ロール参照と AX ツリーノード間の `(role, name, nth)` マッチングロジック。
    -   予算ヘルパーの動作（余裕、残り時間の計算）。
-   統合テスト：
    -   CDP evaluate タイムアウトが予算内で返り、次のアクションをブロックしない。
    -   中止が evaluate をキャンセルし、ベストエフォートで終了をトリガーする。
-   契約テスト：
    -   `BrowserActRequest` と `BrowserActResponse` の互換性が維持されることを確認する。

## リスクと軽減策

-   マッピングが不完全：
    -   軽減策：ベストエフォートのマッピング、Playwright evaluate へのフォールバック、デバッグツールの追加。
-   `Runtime.terminateExecution` に副作用がある：
    -   軽減策：タイムアウト/中止時のみ使用し、エラー内で動作を文書化する。
-   追加のオーバーヘッド：
    -   軽減策：AX ツリーはスナップショットが要求されたときのみ取得し、ターゲットごとにキャッシュし、CDP セッションを短命に保つ。
-   拡張機能リレーの制限：
    -   軽減策：ページごとのソケットが利用できない場合はブラウザレベルのアタッチ API を使用し、現在の Playwright パスをフォールバックとして保持する。

## 未解決の質問

-   新しいエンジンを `playwright`、`cdp`、または `auto` として設定可能にするべきか？
-   高度なユーザー向けに新しい「nodeRef」形式を公開するか、それとも `ref` のみを保持するか？
-   フレームスナップショットとセレクタースコープ付きスナップショットは、AX マッピングにどのように参加すべきか？

[Unified Runtime Streaming Refactor Plan](./acp-unified-streaming-refactor.md)[OpenResponses Gateway Plan](./openresponses-gateway.md)