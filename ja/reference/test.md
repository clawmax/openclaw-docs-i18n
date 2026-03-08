title: "OpenClaw AI テストガイド: コマンド、ベンチマーク、Docker"
description: "OpenClaw AI のテスト実行方法を学びます。ユニットテスト、統合テスト、ライブテストのほか、カバレッジ、ベンチマーク、Docker ベースの E2E テストのコマンドを紹介します。"
keywords: ["openclaw テスト", "vitest コマンド", "docker e2e テスト", "テストカバレッジ", "cli ベンチマーク", "ゲートウェイ統合", "pnpm test", "モデルレイテンシベンチ"]
---

  リリースノート

  
# テスト

-   完全なテストキット（スイート、ライブ、Docker）: [テスト](../help/testing.md)
-   `pnpm test:force`: デフォルトの制御ポートを保持している残留ゲートウェイプロセスを強制終了し、分離されたゲートウェイポートで完全な Vitest スイートを実行します。これにより、サーバーテストが実行中のインスタンスと衝突しません。以前のゲートウェイ実行によりポート 18789 が占有されている場合に使用します。
-   `pnpm test:coverage`: ユニットスイートを V8 カバレッジで実行します（`vitest.unit.config.ts` 経由）。グローバルしきい値は、行/分岐/関数/ステートメントの 70% です。カバレッジ対象からは、統合が中心のエントリーポイント（CLI 配線、ゲートウェイ/Telegram ブリッジ、webchat 静的サーバー）を除外し、ユニットテスト可能なロジックに焦点を当てます。
-   Node 24+ での `pnpm test`: OpenClaw は自動的に Vitest の `vmForks` を無効化し、`ERR_VM_MODULE_LINK_FAILURE` / `module is already linked` を回避するために `forks` を使用します。動作は `OPENCLAW_TEST_VM_FORKS=0|1` で強制できます。
-   `pnpm test`: デフォルトでは、高速なコアユニットレーンを実行し、ローカルでの迅速なフィードバックを提供します。
-   `pnpm test:channels`: チャネル中心のスイートを実行します。
-   `pnpm test:extensions`: 拡張機能/プラグインのスイートを実行します。
-   ゲートウェイ統合: `OPENCLAW_TEST_INCLUDE_GATEWAY=1 pnpm test` または `pnpm test:gateway` でオプトインします。
-   `pnpm test:e2e`: ゲートウェイのエンドツーエンドスモークテストを実行します（マルチインスタンス WS/HTTP/ノードペアリング）。デフォルトでは `vitest.e2e.config.ts` で `vmForks` + 適応型ワーカーを使用します。`OPENCLAW_E2E_WORKERS=` で調整し、詳細ログには `OPENCLAW_E2E_VERBOSE=1` を設定します。
-   `pnpm test:live`: プロバイダーのライブテストを実行します（minimax/zai）。API キーと `LIVE=1`（またはプロバイダー固有の `*_LIVE_TEST=1`）が必要で、スキップ解除されます。

## ローカル PR ゲート

ローカル PR のランド/ゲートチェックでは、以下を実行します:

-   `pnpm check`
-   `pnpm build`
-   `pnpm test`
-   `pnpm check:docs`

`pnpm test` が負荷の高いホストで不安定な場合、回帰と判断する前に一度再実行し、その後 `pnpm vitest run <path/to/test>` で問題を切り分けてください。メモリ制約のあるホストでは、以下を使用します:

-   `OPENCLAW_TEST_PROFILE=low OPENCLAW_TEST_SERIAL_GATEWAY=1 pnpm test`

## モデルレイテンシベンチ（ローカルキー）

スクリプト: [`scripts/bench-model.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-model.ts) 使用方法:

-   `source ~/.profile && pnpm tsx scripts/bench-model.ts --runs 10`
-   オプション環境変数: `MINIMAX_API_KEY`, `MINIMAX_BASE_URL`, `MINIMAX_MODEL`, `ANTHROPIC_API_KEY`
-   デフォルトプロンプト: 「Reply with a single word: ok. No punctuation or extra text.」

最終実行（2025-12-31, 20 回）:

-   minimax 中央値 1279ms (最小 1114, 最大 2431)
-   opus 中央値 2454ms (最小 1224, 最大 3170)

## CLI 起動ベンチ

スクリプト: [`scripts/bench-cli-startup.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-cli-startup.ts) 使用方法:

-   `pnpm tsx scripts/bench-cli-startup.ts`
-   `pnpm tsx scripts/bench-cli-startup.ts --runs 12`
-   `pnpm tsx scripts/bench-cli-startup.ts --entry dist/entry.js --timeout-ms 45000`

以下のコマンドをベンチマークします:

-   `--version`
-   `--help`
-   `health --json`
-   `status --json`
-   `status`

出力には、各コマンドの平均、p50、p95、最小/最大、および終了コード/シグナルの分布が含まれます。

## オンボーディング E2E (Docker)

Docker はオプションです。これはコンテナ化されたオンボーディングスモークテストにのみ必要です。クリーンな Linux コンテナ内での完全なコールドスタートフロー:

```
scripts/e2e/onboard-docker.sh
```

このスクリプトは、疑似 TTY 経由で対話型ウィザードを駆動し、設定/ワークスペース/セッションファイルを検証した後、ゲートウェイを起動して `openclaw health` を実行します。

## QR インポートスモーク (Docker)

Docker 内の Node 22+ で `qrcode-terminal` が正常にロードされることを確認します:

```bash
pnpm test:docker:qr
```

[リリースチェックリスト](./RELEASING.md)[Kilo ゲートウェイ統合](../design/kilo-gateway-integration.md)