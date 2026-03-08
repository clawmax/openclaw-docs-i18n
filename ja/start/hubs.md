

  ドキュメントメタ

  
# ドキュメントハブ

> **ℹ️** OpenClaw が初めての方は、[はじめに](./getting-started.md)から始めてください。

 これらのハブを使用して、左側のナビゲーションに表示されない詳細解説やリファレンスドキュメントを含む、すべてのページを発見してください。

## はじめに

-   [インデックス](../home.md)
-   [はじめに](./getting-started.md)
-   [クイックスタート](./quickstart.md)
-   [オンボーディング](./onboarding.md)
-   [ウィザード](./wizard.md)
-   [セットアップ](./setup.md)
-   [ダッシュボード (ローカルゲートウェイ)](http://127.0.0.1:18789/)
-   [ヘルプ](../help.md)
-   [ドキュメントディレクトリ](./docs-directory.md)
-   [設定](../gateway/configuration.md)
-   [設定例](../gateway/configuration-examples.md)
-   [OpenClaw アシスタント](./openclaw.md)
-   [ショーケース](./showcase.md)
-   [背景設定](./lore.md)

## インストール + 更新

-   [Docker](../install/docker.md)
-   [Nix](../install/nix.md)
-   [更新 / ロールバック](../install/updating.md)
-   [Bun ワークフロー (実験的)](../install/bun.md)

## コアコンセプト

-   [アーキテクチャ](../concepts/architecture.md)
-   [機能](../concepts/features.md)
-   [ネットワークハブ](../network.md)
-   [エージェントランタイム](../concepts/agent.md)
-   [エージェントワークスペース](../concepts/agent-workspace.md)
-   [メモリ](../concepts/memory.md)
-   [エージェントループ](../concepts/agent-loop.md)
-   [ストリーミング + チャンキング](../concepts/streaming.md)
-   [マルチエージェントルーティング](../concepts/multi-agent.md)
-   [コンパクション](../concepts/compaction.md)
-   [セッション](../concepts/session.md)
-   [セッション剪定](../concepts/session-pruning.md)
-   [セッションツール](../concepts/session-tool.md)
-   [キュー](../concepts/queue.md)
-   [スラッシュコマンド](../tools/slash-commands.md)
-   [RPC アダプター](../reference/rpc.md)
-   [TypeBox スキーマ](../concepts/typebox.md)
-   [タイムゾーン処理](../concepts/timezone.md)
-   [プレゼンス](../concepts/presence.md)
-   [ディスカバリー + トランスポート](../gateway/discovery.md)
-   [Bonjour](../gateway/bonjour.md)
-   [チャネルルーティング](../channels/channel-routing.md)
-   [グループ](../channels/groups.md)
-   [グループメッセージ](../channels/group-messages.md)
-   [モデルフェイルオーバー](../concepts/model-failover.md)
-   [OAuth](../concepts/oauth.md)

## プロバイダー + イングレス

-   [チャットチャネルハブ](../channels.md)
-   [モデルプロバイダーハブ](../providers/models.md)
-   [WhatsApp](../channels/whatsapp.md)
-   [Telegram](../channels/telegram.md)
-   [Slack](../channels/slack.md)
-   [Discord](../channels/discord.md)
-   [Mattermost](../channels/mattermost.md) (プラグイン)
-   [Signal](../channels/signal.md)
-   [BlueBubbles (iMessage)](../channels/bluebubbles.md)
-   [iMessage (レガシー)](../channels/imessage.md)
-   [位置情報解析](../channels/location.md)
-   [WebChat](../web/webchat.md)
-   [Webhooks](../automation/webhook.md)
-   [Gmail Pub/Sub](../automation/gmail-pubsub.md)

## ゲートウェイ + 運用

-   [ゲートウェイ運用マニュアル](../gateway.md)
-   [ネットワークモデル](../gateway/network-model.md)
-   [ゲートウェイペアリング](../gateway/pairing.md)
-   [ゲートウェイロック](../gateway/gateway-lock.md)
-   [バックグラウンドプロセス](../gateway/background-process.md)
-   [ヘルス](../gateway/health.md)
-   [ハートビート](../gateway/heartbeat.md)
-   [ドクター](../gateway/doctor.md)
-   [ロギング](../gateway/logging.md)
-   [サンドボックス化](../gateway/sandboxing.md)
-   [ダッシュボード](../web/dashboard.md)
-   [コントロールUI](../web/control-ui.md)
-   [リモートアクセス](../gateway/remote.md)
-   [リモートゲートウェイ README](../gateway/remote-gateway-readme.md)
-   [Tailscale](../gateway/tailscale.md)
-   [セキュリティ](../gateway/security.md)
-   [トラブルシューティング](../gateway/troubleshooting.md)

## ツール + 自動化

-   [ツールサーフェス](../tools.md)
-   [OpenProse](../prose.md)
-   [CLI リファレンス](../cli.md)
-   [Exec ツール](../tools/exec.md)
-   [PDF ツール](../tools/pdf.md)
-   [昇格モード](../tools/elevated.md)
-   [Cron ジョブ](../automation/cron-jobs.md)
-   [Cron 対 Heartbeat](../automation/cron-vs-heartbeat.md)
-   [思考 + 詳細出力](../tools/thinking.md)
-   [モデル](../concepts/models.md)
-   [サブエージェント](../tools/subagents.md)
-   [エージェント送信 CLI](../tools/agent-send.md)
-   [ターミナルUI](../web/tui.md)
-   [ブラウザ制御](../tools/browser.md)
-   [ブラウザ (Linux トラブルシューティング)](../tools/browser-linux-troubleshooting.md)
-   [投票](../automation/poll.md)

## ノード、メディア、音声

-   [ノード概要](../nodes.md)
-   [カメラ](../nodes/camera.md)
-   [画像](../nodes/images.md)
-   [オーディオ](../nodes/audio.md)
-   [位置情報コマンド](../nodes/location-command.md)
-   [音声起動](../nodes/voicewake.md)
-   [トークモード](../nodes/talk.md)

## プラットフォーム

-   [プラットフォーム概要](../platforms.md)
-   [macOS](../platforms/macos.md)
-   [iOS](../platforms/ios.md)
-   [Android](../platforms/android.md)
-   [Windows (WSL2)](../platforms/windows.md)
-   [Linux](../platforms/linux.md)
-   [Web サーフェス](../web.md)

## macOS コンパニオンアプリ (上級者向け)

-   [macOS 開発環境設定](../platforms/mac/dev-setup.md)
-   [macOS メニューバー](../platforms/mac/menu-bar.md)
-   [macOS 音声起動](../platforms/mac/voicewake.md)
-   [macOS 音声オーバーレイ](../platforms/mac/voice-overlay.md)
-   [macOS WebChat](../platforms/mac/webchat.md)
-   [macOS キャンバス](../platforms/mac/canvas.md)
-   [macOS 子プロセス](../platforms/mac/child-process.md)
-   [macOS ヘルス](../platforms/mac/health.md)
-   [macOS アイコン](../platforms/mac/icon.md)
-   [macOS ロギング](../platforms/mac/logging.md)
-   [macOS 権限](../platforms/mac/permissions.md)
-   [macOS リモート](../platforms/mac/remote.md)
-   [macOS 署名](../platforms/mac/signing.md)
-   [macOS リリース](../platforms/mac/release.md)
-   [macOS ゲートウェイ (launchd)](../platforms/mac/bundled-gateway.md)
-   [macOS XPC](../platforms/mac/xpc.md)
-   [macOS スキル](../platforms/mac/skills.md)
-   [macOS Peekaboo](../platforms/mac/peekaboo.md)

## ワークスペース + テンプレート

-   [スキル](../tools/skills.md)
-   [ClawHub](../tools/clawhub.md)
-   [スキル設定](../tools/skills-config.md)
-   [デフォルト AGENTS](../reference/AGENTS.default.md)
-   [テンプレート: AGENTS](../reference/templates/AGENTS.md)
-   [テンプレート: BOOTSTRAP](../reference/templates/BOOTSTRAP.md)
-   [テンプレート: HEARTBEAT](../reference/templates/HEARTBEAT.md)
-   [テンプレート: IDENTITY](../reference/templates/IDENTITY.md)
-   [テンプレート: SOUL](../reference/templates/SOUL.md)
-   [テンプレート: TOOLS](../reference/templates/TOOLS.md)
-   [テンプレート: USER](../reference/templates/USER.md)

## 実験 (探索的)

-   [オンボーディング設定プロトコル](../experiments/onboarding-config-protocol.md)
-   [研究: メモリ](../experiments/research/memory.md)
-   [モデル設定の探求](../experiments/proposals/model-config.md)

## プロジェクト

-   [クレジット](../reference/credits.md)

## テスト + リリース

-   [テスト](../reference/test.md)
-   [リリースチェックリスト](../reference/RELEASING.md)
-   [デバイスモデル](../reference/device-models.md)

[CI パイプライン](../ci.md)[ドキュメントディレクトリ](./docs-directory.md)

---