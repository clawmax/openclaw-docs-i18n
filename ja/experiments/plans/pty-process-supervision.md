

  実験

  
# PTYおよびプロセス監視計画

## 1\. 問題と目標

以下にわたる長時間実行コマンド実行のための、一つの信頼性の高いライフサイクルが必要です:

-   `exec` フォアグラウンド実行
-   `exec` バックグラウンド実行
-   `process` フォローアップアクション (`poll`, `log`, `send-keys`, `paste`, `submit`, `kill`, `remove`)
-   CLIエージェントランナーのサブプロセス

目標は単にPTYをサポートすることではありません。目標は、安全でないプロセスマッチングヒューリスティックなしに、予測可能な所有権、キャンセル、タイムアウト、およびクリーンアップを実現することです。

## 2\. 範囲と境界

-   実装は `src/process/supervisor` 内に内部で保持します。
-   これのために新しいパッケージを作成しないでください。
-   実用的な範囲で現在の動作互換性を保持します。
-   ターミナルリプレイやtmuxスタイルのセッション永続化への範囲拡大は行いません。

## 3\. このブランチで実装済み

### スーパーバイザーのベースラインは既に存在

-   スーパーバイザーモジュールは `src/process/supervisor/*` 下に配置済みです。
-   ExecランタイムとCLIランナーは既にスーパーバイザーのspawnとwaitを経由するようルーティングされています。
-   レジストリのファイナライズは冪等です。

### このパスで完了した項目

1.  明示的なPTYコマンド契約

-   `SpawnInput` は `src/process/supervisor/types.ts` で判別共用体になりました。
-   PTY実行では、汎用の `argv` を再利用する代わりに `ptyCommand` が必要です。
-   スーパーバイザーは `src/process/supervisor/supervisor.ts` でargvの結合からPTYコマンド文字列を再構築しなくなりました。
-   Execランタイムは `src/agents/bash-tools.exec-runtime.ts` で `ptyCommand` を直接渡すようになりました。

2.  プロセスレイヤーの型の分離

-   スーパーバイザーの型は、エージェントからの `SessionStdin` をインポートしなくなりました。
-   プロセスローカルなstdin契約は `src/process/supervisor/types.ts` (`ManagedRunStdin`) に存在します。
-   アダプターはプロセスレベルの型のみに依存するようになりました:
    -   `src/process/supervisor/adapters/child.ts`
    -   `src/process/supervisor/adapters/pty.ts`

3.  プロセスツールのライフサイクル所有権の改善

-   `src/agents/bash-tools.process.ts` は、最初にスーパーバイザーを通じてキャンセルを要求するようになりました。
-   `process kill/remove` は、スーパーバイザールックアップが失敗した場合にプロセスツリーによるフォールバック終了を使用するようになりました。
-   `remove` は、終了要求後すぐに実行中のセッションエントリを削除することで、決定論的な削除動作を維持します。

4.  単一ソースのウォッチドッグデフォルト

-   `src/agents/cli-watchdog-defaults.ts` に共有デフォルトを追加しました。
-   `src/agents/cli-backends.ts` は共有デフォルトを利用します。
-   `src/agents/cli-runner/reliability.ts` も同じ共有デフォルトを利用します。

5.  不要なヘルパーのクリーンアップ

-   `src/agents/bash-tools.shared.ts` から未使用の `killSession` ヘルパーパスを削除しました。

6.  直接スーパーバイザーパスのテスト追加

-   `src/agents/bash-tools.process.supervisor.test.ts` を追加し、スーパーバイザーキャンセルを経由するkillとremoveのルーティングをカバーしました。

7.  信頼性ギャップの修正完了

-   `src/agents/bash-tools.process.ts` は、スーパーバイザールックアップが失敗した場合に、実際のOSレベルのプロセス終了にフォールバックするようになりました。
-   `src/process/supervisor/adapters/child.ts` は、デフォルトのキャンセル/タイムアウトkillパスにプロセスツリー終了セマンティクスを使用するようになりました。
-   `src/process/kill-tree.ts` に共有プロセスツーティリティを追加しました。

8.  PTY契約のエッジケースカバレッジ追加

-   `src/process/supervisor/supervisor.pty-command.test.ts` を追加し、逐語的なPTYコマンド転送と空コマンド拒否をテストします。
-   `src/process/supervisor/adapters/child.test.ts` を追加し、子アダプターキャンセルにおけるプロセスツリーkill動作をテストします。

## 4\. 残りのギャップと決定事項

### 信頼性の状況

このパスで必要な2つの信頼性ギャップは現在閉じられています:

-   `process kill/remove` は、スーパーバイザールックアップが失敗した場合に実際のOS終了フォールバックを持つようになりました。
-   子キャンセル/タイムアウトは、デフォルトkillパスにプロセスツリーkillセマンティクスを使用するようになりました。
-   両方の動作に対して回帰テストが追加されました。

### 耐久性と起動時の調整

再起動時の動作は、明示的にインメモリライフサイクルのみと定義されました。

-   `reconcileOrphans()` は設計により、`src/process/supervisor/supervisor.ts` でno-opのままです。
-   アクティブな実行はプロセス再起動後に回復されません。
-   この境界は、部分的な永続化のリスクを避けるために、この実装パスでは意図的なものです。

### 保守性に関するフォローアップ

1.  `src/agents/bash-tools.exec-runtime.ts` 内の `runExecProcess` は依然として複数の責任を扱っており、フォローアップで焦点を絞ったヘルパーに分割できます。

## 5\. 実装計画

必要な信頼性と契約項目の実装パスは完了しました。完了項目:

-   `process kill/remove` の実際の終了フォールバック
-   子アダプターデフォルトkillパスのためのプロセスツリーキャンセル
-   フォールバックkillと子アダプターkillパスの回帰テスト
-   明示的な `ptyCommand` 下でのPTYコマンドエッジケーステスト
-   設計による `reconcileOrphans()` no-opを伴う明示的なインメモリ再起動境界

オプションのフォローアップ:

-   動作のずれなしに `runExecProcess` を焦点を絞ったヘルパーに分割

## 6\. ファイルマップ

### プロセススーパーバイザー

-   `src/process/supervisor/types.ts` は判別spawn入力とプロセスローカルstdin契約で更新されました。
-   `src/process/supervisor/supervisor.ts` は明示的な `ptyCommand` を使用するよう更新されました。
-   `src/process/supervisor/adapters/child.ts` と `src/process/supervisor/adapters/pty.ts` はエージェントの型から分離されました。
-   `src/process/supervisor/registry.ts` の冪等ファイナライズは変更なく保持されました。

### Execおよびプロセス統合

-   `src/agents/bash-tools.exec-runtime.ts` はPTYコマンドを明示的に渡し、フォールバックパスを保持するよう更新されました。
-   `src/agents/bash-tools.process.ts` はスーパーバイザー経由でキャンセルし、実際のプロセスツリーフォールバック終了を行うよう更新されました。
-   `src/agents/bash-tools.shared.ts` から直接killヘルパーパスを削除しました。

### CLI信頼性

-   `src/agents/cli-watchdog-defaults.ts` が共有ベースラインとして追加されました。
-   `src/agents/cli-backends.ts` と `src/agents/cli-runner/reliability.ts` は同じデフォルトを利用するようになりました。

## 7\. このパスでの検証実行

単体テスト:

-   `pnpm vitest src/process/supervisor/registry.test.ts`
-   `pnpm vitest src/process/supervisor/supervisor.test.ts`
-   `pnpm vitest src/process/supervisor/supervisor.pty-command.test.ts`
-   `pnpm vitest src/process/supervisor/adapters/child.test.ts`
-   `pnpm vitest src/agents/cli-backends.test.ts`
-   `pnpm vitest src/agents/bash-tools.exec.pty-cleanup.test.ts`
-   `pnpm vitest src/agents/bash-tools.process.poll-timeout.test.ts`
-   `pnpm vitest src/agents/bash-tools.process.supervisor.test.ts`
-   `pnpm vitest src/process/exec.test.ts`

E2Eターゲット:

-   `pnpm vitest src/agents/cli-runner.test.ts`
-   `pnpm vitest run src/agents/bash-tools.exec.pty-fallback.test.ts src/agents/bash-tools.exec.background-abort.test.ts src/agents/bash-tools.process.send-keys.test.ts`

型チェック注意:

-   このリポジトリでは `pnpm build` (および完全なlint/docsゲートの場合は `pnpm check`) を使用してください。`pnpm tsgo` に言及している古いメモは廃止されています。

## 8\. 保持された運用保証

-   Exec環境の強化動作は変更されていません。
-   承認と許可リストフローは変更されていません。
-   出力サニタイズと出力キャップは変更されていません。
-   PTYアダプターは依然として、強制killとリスナー破棄時のwait解決を保証します。

## 9\. 完了の定義

1.  スーパーバイザーは管理対象実行のライフサイクル所有者です。
2.  PTY spawnは、argv再構築なしの明示的なコマンド契約を使用します。
3.  プロセスレイヤーは、スーパーバイザーstdin契約に対してエージェントレイヤーの型依存を持ちません。
4.  ウォッチドッグデフォルトは単一ソースです。
5.  対象の単体およびe2eテストはグリーンのままです。
6.  再起動耐久性境界は明示的に文書化されるか、完全に実装されています。

## 10\. まとめ

このブランチは、一貫性がありより安全な監視構造を持つようになりました:

-   明示的なPTY契約
-   よりクリーンなプロセス階層化
-   プロセス操作のためのスーパーバイザー主導のキャンセルパス
-   スーパーバイザールックアップ失敗時の実際のフォールバック終了
-   子実行デフォルトkillパスのためのプロセスツリーキャンセル
-   統一されたウォッチドッグデフォルト
-   明示的なインメモリ再起動境界 (このパスでは再起動をまたがる孤児調整なし)

[OpenResponses Gateway Plan](./openresponses-gateway.md)[Session Binding Channel Agnostic Plan](./session-binding-channel-agnostic.md)