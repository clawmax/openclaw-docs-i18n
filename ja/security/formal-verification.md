

  セキュリティ

  
# 形式検証 (セキュリティモデル)

このページは、OpenClaw の **形式的セキュリティモデル** (現在は TLA+/TLC、必要に応じて追加) を追跡します。

> 注: 古いリンクの一部は、以前のプロジェクト名を参照している場合があります。

**目標 (北極星):** OpenClaw が、明示的な仮定の下で、意図したセキュリティポリシー (認可、セッション分離、ツールゲーティング、設定ミスの安全性) を強制することを、機械的にチェック可能な論拠として提供すること。**現在の内容:** 実行可能で、攻撃者を想定した **セキュリティ回帰テストスイート**:

-   各主張には、有限状態空間上で実行可能なモデル検査があります。
-   多くの主張には、現実的なバグクラスに対する反例トレースを生成する **ネガティブモデル** がペアで用意されています。

**現在の限界:** 「OpenClaw が全ての点で安全である」ことや、完全な TypeScript 実装が正しいことの証明ではありません。

## モデルの所在

モデルは別のリポジトリで管理されています: [vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models)。

## 重要な注意点

-   これらは **モデル** であり、完全な TypeScript 実装ではありません。モデルとコードの間に乖離が生じる可能性があります。
-   結果は TLC によって探索される状態空間によって制限されます。「緑」は、モデル化された仮定と範囲を超えたセキュリティを意味しません。
-   一部の主張は、明示的な環境仮定 (例: 正しいデプロイ、正しい設定入力) に依存しています。

## 結果の再現

現在、結果はモデルリポジトリをローカルにクローンして TLC を実行することで再現できます (下記参照)。将来のバージョンでは以下を提供する可能性があります:

-   公開アーティファクト (反例トレース、実行ログ) 付きの CI 実行モデル
-   小規模で範囲が限定されたチェックのための、ホスト型「このモデルを実行」ワークフロー

始め方:

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# Java 11+ が必要です (TLC は JVM 上で動作します)。
# このリポジトリは固定バージョンの `tla2tools.jar` (TLA+ ツール) を同梱し、`bin/tlc` と Make ターゲットを提供します。

make <target>
```

### ゲートウェイ公開とオープンゲートウェイ設定ミス

**主張:** 認証なしでループバックを超えてバインドすると、リモートからの侵害が可能になる可能性があり / 公開範囲が増大する。トークン/パスワードは (モデルの仮定に従い) 未認証の攻撃者をブロックする。

-   成功 (緑) 実行:
    -   `make gateway-exposure-v2`
    -   `make gateway-exposure-v2-protected`
-   失敗 (予期される赤):
    -   `make gateway-exposure-v2-negative`

詳細: モデルリポジトリ内の `docs/gateway-exposure-matrix.md` も参照してください。

### Nodes.run パイプライン (最高リスクの機能)

**主張:** `nodes.run` は (a) ノードコマンド許可リストと宣言されたコマンド、および (b) 設定時にライブ承認を必要とする。承認は (モデル内で) リプレイを防ぐためにトークン化される。

-   成功 (緑) 実行:
    -   `make nodes-pipeline`
    -   `make approvals-token`
-   失敗 (予期される赤):
    -   `make nodes-pipeline-negative`
    -   `make approvals-token-negative`

### ペアリングストア (DM ゲーティング)

**主張:** ペアリングリクエストは TTL と保留中リクエストの上限を尊重する。

-   成功 (緑) 実行:
    -   `make pairing`
    -   `make pairing-cap`
-   失敗 (予期される赤):
    -   `make pairing-negative`
    -   `make pairing-cap-negative`

### イングレスゲーティング (メンション + 制御コマンドバイパス)

**主張:** メンションを必要とするグループコンテキストでは、未認可の「制御コマンド」はメンションゲーティングをバイパスできない。

-   成功 (緑):
    -   `make ingress-gating`
-   失敗 (予期される赤):
    -   `make ingress-gating-negative`

### ルーティング/セッションキー分離

**主張:** 異なるピアからの DM は、明示的にリンク/設定されない限り、同じセッションに折りたたまれない。

-   成功 (緑):
    -   `make routing-isolation`
-   失敗 (予期される赤):
    -   `make routing-isolation-negative`

## v1++: 追加の範囲限定モデル (並行性、リトライ、トレース正確性)

これらは、現実世界の障害モード (非アトミックな更新、リトライ、メッセージファンアウト) に関する忠実度を高めるフォローオンモデルです。

### ペアリングストアの並行性 / べき等性

**主張:** ペアリングストアは、インターリーブ下でも `MaxPending` とべき等性を強制すべきである (つまり、「チェックしてから書き込み」はアトミック / ロックされている必要がある。リフレッシュは重複を作成すべきではない)。意味すること:

-   並行リクエスト下では、チャネルごとの `MaxPending` を超えることはできない。
-   同じ `(channel, sender)` に対する繰り返しのリクエスト/リフレッシュは、重複するライブ保留行を作成すべきではない。
-   成功 (緑) 実行:
    -   `make pairing-race` (アトミック/ロックされた上限チェック)
    -   `make pairing-idempotency`
    -   `make pairing-refresh`
    -   `make pairing-refresh-race`
-   失敗 (予期される赤):
    -   `make pairing-race-negative` (非アトミックな開始/コミット上限競合)
    -   `make pairing-idempotency-negative`
    -   `make pairing-refresh-negative`
    -   `make pairing-refresh-race-negative`

### イングレストレース相関 / べき等性

**主張:** インジェストは、ファンアウト全体でトレース相関を保持し、プロバイダーのリトライ下でべき等であるべき。意味すること:

-   1つの外部イベントが複数の内部メッセージになる場合、各部分は同じトレース/イベントIDを保持する。
-   リトライによって二重処理が発生しない。
-   プロバイダーイベントIDが欠落している場合、重複排除は安全なキー (例: トレースID) にフォールバックし、異なるイベントをドロップしないようにする。
-   成功 (緑):
    -   `make ingress-trace`
    -   `make ingress-trace2`
    -   `make ingress-idempotency`
    -   `make ingress-dedupe-fallback`
-   失敗 (予期される赤):
    -   `make ingress-trace-negative`
    -   `make ingress-trace2-negative`
    -   `make ingress-idempotency-negative`
    -   `make ingress-dedupe-fallback-negative`

### ルーティング dmScope 優先順位 + identityLinks

**主張:** ルーティングは、デフォルトでは DM セッションを分離したままにし、明示的に設定された場合にのみセッションを折りたたむ必要がある (チャネル優先順位 + アイデンティティリンク)。意味すること:

-   チャネル固有の dmScope オーバーライドは、グローバルデフォルトよりも優先されなければならない。
-   identityLinks は、明示的にリンクされたグループ内でのみ折りたたまれ、無関係なピア間では折りたたまれない。
-   成功 (緑):
    -   `make routing-precedence`
    -   `make routing-identitylinks`
-   失敗 (予期される赤):
    -   `make routing-precedence-negative`
    -   `make routing-identitylinks-negative`

[Tailscale](../gateway/tailscale.md)[README](./README.md)