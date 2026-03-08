

  ガイド

  
# パーソナルアシスタント セットアップ

OpenClawは、**Pi**エージェントのためのWhatsApp + Telegram + Discord + iMessageゲートウェイです。プラグインでMattermostも追加可能です。このガイドは「パーソナルアシスタント」セットアップ、つまり常時稼働するあなた専用のエージェントとして振る舞う、専用のWhatsApp番号の設定について説明します。

## ⚠️ 安全第一

あなたはエージェントを以下のような立場に置こうとしています：

-   マシン上でコマンドを実行する（Piツールの設定による）
-   ワークスペース内のファイルを読み書きする
-   WhatsApp/Telegram/Discord/Mattermost（プラグイン）経由でメッセージを送信する

まずは控えめに始めましょう：

-   常に `channels.whatsapp.allowFrom` を設定してください（個人のMacで世界に公開された状態で実行しない）。
-   アシスタント用に専用のWhatsApp番号を使用してください。
-   ハートビートは現在、デフォルトで30分ごとに実行されます。設定を信頼するまでは、`agents.defaults.heartbeat.every: "0m"` を設定して無効にしてください。

## 前提条件

-   OpenClawがインストールされ、オンボーディング済み — まだ完了していない場合は[はじめに](./getting-started.md)を参照
-   アシスタント用の2つ目の電話番号（SIM/eSIM/プリペイド）

## 2台の電話番号を使うセットアップ（推奨）

望ましいのはこれです：個人のWhatsAppをOpenClawにリンクすると、あなた宛のすべてのメッセージが「エージェントへの入力」になってしまいます。それは通常、望むところではありません。

## 5分間クイックスタート

1.  WhatsApp Webをペアリング（QRコードを表示；アシスタント用の電話でスキャン）：

```bash
openclaw channels login
```

2.  ゲートウェイを起動（実行したままにします）：

```bash
openclaw gateway --port 18789
```

3.  最小限の設定を `~/.openclaw/openclaw.json` に記述：

```json
{
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

許可リストに登録した電話からアシスタント番号にメッセージを送信します。オンボーディングが完了すると、ダッシュボードが自動的に開き、クリーンな（トークン化されていない）リンクが表示されます。認証を求められた場合は、`gateway.auth.token` のトークンをControl UIの設定に貼り付けてください。後で再度開くには：`openclaw dashboard`。

## エージェントにワークスペースを与える（AGENTS）

OpenClawは、操作指示と「記憶」をそのワークスペースディレクトリから読み取ります。デフォルトでは、OpenClawは `~/.openclaw/workspace` をエージェントワークスペースとして使用し、セットアップ時または最初のエージェント実行時に自動的に作成します（スターター `AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md` も作成）。`BOOTSTRAP.md` はワークスペースが真新しい場合にのみ作成されます（削除後は再作成されません）。`MEMORY.md` はオプション（自動作成されません）；存在する場合、通常セッションで読み込まれます。サブエージェントセッションでは `AGENTS.md` と `TOOLS.md` のみが注入されます。ヒント：このフォルダをOpenClawの「記憶」として扱い、gitリポジトリ（理想的にはプライベート）にして、`AGENTS.md` + メモリファイルをバックアップしてください。gitがインストールされている場合、真新しいワークスペースは自動的に初期化されます。

```bash
openclaw setup
```

完全なワークスペースレイアウト + バックアップガイド：[エージェントワークスペース](../concepts/agent-workspace.md) メモリワークフロー：[メモリ](../concepts/memory.md) オプション：`agents.defaults.workspace` で別のワークスペースを選択可能（`~` をサポート）。

```json
{
  agent: {
    workspace: "~/.openclaw/workspace",
  },
}
```

すでにリポジトリから独自のワークスペースファイルを提供している場合は、ブートストラップファイルの作成を完全に無効にできます：

```json
{
  agent: {
    skipBootstrap: true,
  },
}
```

## 「アシスタント」にするための設定

OpenClawはデフォルトで良いアシスタント設定になっていますが、通常は以下を調整したいでしょう：

-   `SOUL.md` 内のペルソナ/指示
-   思考のデフォルト（必要であれば）
-   ハートビート（信頼できたら）

例：

```json
{
  logging: { level: "info" },
  agent: {
    model: "anthropic/claude-opus-4-6",
    workspace: "~/.openclaw/workspace",
    thinkingDefault: "high",
    timeoutSeconds: 1800,
    // 最初は0から始め、後で有効化。
    heartbeat: { every: "0m" },
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true },
      },
    },
  },
  routing: {
    groupChat: {
      mentionPatterns: ["@openclaw", "openclaw"],
    },
  },
  session: {
    scope: "per-sender",
    resetTriggers: ["/new", "/reset"],
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 10080,
    },
  },
}
```

## セッションとメモリ

-   セッションファイル：`~/.openclaw/agents//sessions/{{SessionId}}.jsonl`
-   セッションメタデータ（トークン使用量、最終ルートなど）：`~/.openclaw/agents//sessions/sessions.json`（レガシー：`~/.openclaw/sessions/sessions.json`）
-   `/new` または `/reset` はそのチャットの新しいセッションを開始します（`resetTriggers` で設定可能）。単独で送信された場合、エージェントはリセットを確認する短い挨拶で応答します。
-   `/compact [instructions]` はセッションコンテキストを圧縮し、残りのコンテキスト予算を報告します。

## ハートビート（プロアクティブモード）

デフォルトでは、OpenClawは30分ごとにハートビートを実行し、次のプロンプトを使用します：`HEARTBEAT.md が存在する場合は読み取ってください（ワークスペースコンテキスト）。それを厳密に従ってください。以前のチャットからの古いタスクを推測したり繰り返したりしないでください。注意が必要なものが何もない場合は、HEARTBEAT_OK と返信してください。` 無効にするには `agents.defaults.heartbeat.every: "0m"` を設定します。

-   `HEARTBEAT.md` が存在しても実質的に空の場合（空白行と `# 見出し` のようなマークダウンヘッダーのみ）、OpenClawはAPIコールを節約するためにハートビート実行をスキップします。
-   ファイルが存在しない場合、ハートビートは依然として実行され、モデルが何をすべきかを決定します。
-   エージェントが `HEARTBEAT_OK` で応答した場合（オプションで短いパディング付き；`agents.defaults.heartbeat.ackMaxChars` を参照）、OpenClawはそのハートビートのアウトバウンド配信を抑制します。
-   デフォルトでは、DM形式の `user:` ターゲットへのハートビート配信は許可されています。ハートビート実行をアクティブに保ちながら直接ターゲットへの配信を抑制するには、`agents.defaults.heartbeat.directPolicy: "block"` を設定します。
-   ハートビートは完全なエージェントターンを実行します — 間隔が短いほど多くのトークンを消費します。

```json
{
  agent: {
    heartbeat: { every: "30m" },
  },
}
```

## メディアの送受信

受信した添付ファイル（画像/音声/文書）は、テンプレートを介してコマンドに表示できます：

-   `{{MediaPath}}`（ローカルの一時ファイルパス）
-   `{{MediaUrl}}`（疑似URL）
-   `{{Transcript}}`（音声文字起こしが有効な場合）

エージェントからの送信添付ファイル：`MEDIA:<path-or-url>` を単独の行に記述（スペースなし）。例：

```
スクリーンショットです。
MEDIA:https://example.com/screenshot.png
```

OpenClawはこれらを抽出し、テキストと一緒にメディアとして送信します。

## 運用チェックリスト

```bash
openclaw status          # ローカルステータス（認証情報、セッション、キューイングされたイベント）
openclaw status --all    # 完全な診断（読み取り専用、貼り付け可能）
openclaw status --deep   # ゲートウェイヘルスプローブを追加（Telegram + Discord）
openclaw health --json   # ゲートウェイヘルススナップショット（WS）
```

ログは `/tmp/openclaw/` 以下に保存されます（デフォルト：`openclaw-YYYY-MM-DD.log`）。

## 次のステップ

-   WebChat：[WebChat](../web/webchat.md)
-   ゲートウェイ運用：[ゲートウェイ ランブック](../gateway.md)
-   Cron + ウェイクアップ：[Cronジョブ](../automation/cron-jobs.md)
-   macOSメニューバーコンパニオン：[OpenClaw macOSアプリ](../platforms/macos.md)
-   iOSノードアプリ：[iOSアプリ](../platforms/ios.md)
-   Androidノードアプリ：[Androidアプリ](../platforms/android.md)
-   Windowsステータス：[Windows (WSL2)](../platforms/windows.md)
-   Linuxステータス：[Linuxアプリ](../platforms/linux.md)
-   セキュリティ：[セキュリティ](../gateway/security.md)

[オンボーディング：macOSアプリ](./onboarding.md)[CLIリファレンス](./wizard-cli-reference.md)