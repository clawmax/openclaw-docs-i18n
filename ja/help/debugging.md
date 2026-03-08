title: "ストリーミング出力と環境のためのOpenClawデバッグヘルパー"
description: "ランタイムオーバーライド、開発プロファイル、生ストリームロギング、および高速反復のためのゲートウェイウォッチモードを使用してOpenClawをデバッグする方法を学びます。"
keywords: ["openclaw デバッグ", "ストリーミング出力 デバッグ", "ランタイムオーバーライド", "開発プロファイル", "ゲートウェイウォッチモード", "生ストリームロギング", "pi-mono ロギング", "環境設定"]
---

  環境とデバッグ

  
# デバッグ

このページでは、特にプロバイダーが通常のテキストに推論を混ぜる場合の、ストリーミング出力のためのデバッグヘルパーについて説明します。

## ランタイムデバッグオーバーライド

チャットで `/debug` を使用して、**ランタイムのみ**の設定オーバーライド（ディスクではなくメモリ）を設定します。 `/debug` はデフォルトで無効です； `commands.debug: true` で有効にします。これは `openclaw.json` を編集せずに不明瞭な設定を切り替える必要がある場合に便利です。例：

```bash
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug unset messages.responsePrefix
/debug reset
```

`/debug reset` はすべてのオーバーライドをクリアし、ディスク上の設定に戻ります。

## ゲートウェイウォッチモード

高速な反復開発のために、ファイルウォッチャー下でゲートウェイを実行します：

```bash
pnpm gateway:watch
```

これは以下にマッピングされます：

```bash
node --watch-path src --watch-path tsconfig.json --watch-path package.json --watch-preserve-output scripts/run-node.mjs gateway --force
```

`gateway:watch` の後に任意のゲートウェイCLIフラグを追加すると、各再起動時に渡されます。

## 開発プロファイル + 開発ゲートウェイ (—dev)

開発プロファイルを使用して状態を分離し、デバッグ用の安全で使い捨て可能なセットアップを起動します。 **2つ**の `--dev` フラグがあります：

-   **グローバル `--dev` (プロファイル):** `~/.openclaw-dev` 下で状態を分離し、ゲートウェイポートをデフォルトで `19001` に設定します（派生ポートはこれに合わせてシフトします）。
-   **`gateway --dev`: ゲートウェイに、欠落している場合にデフォルト設定 + ワークスペースを自動作成するよう指示します**（そしてBOOTSTRAP.mdをスキップします）。

推奨フロー（開発プロファイル + 開発ブートストラップ）：

```bash
pnpm gateway:dev
OPENCLAW_PROFILE=dev openclaw tui
```

まだグローバルインストールがない場合は、CLIを `pnpm openclaw ...` 経由で実行してください。これが行うこと：

1.  **プロファイル分離** (グローバル `--dev`)
    -   `OPENCLAW_PROFILE=dev`
    -   `OPENCLAW_STATE_DIR=~/.openclaw-dev`
    -   `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
    -   `OPENCLAW_GATEWAY_PORT=19001` (ブラウザ/キャンバスはそれに応じてシフト)
2.  **開発ブートストラップ** (`gateway --dev`)
    -   欠落している場合に最小限の設定を書き込みます (`gateway.mode=local`, ループバックにバインド)。
    -   `agent.workspace` を開発ワークスペースに設定します。
    -   `agent.skipBootstrap=true` を設定します (BOOTSTRAP.mdなし)。
    -   ワークスペースファイルが欠落している場合にシードします: `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`。
    -   デフォルトアイデンティティ: **C3‑PO** (プロトコルドロイド)。
    -   開発モードではチャネルプロバイダーをスキップします (`OPENCLAW_SKIP_CHANNELS=1`)。

リセットフロー（新規開始）：

```bash
pnpm gateway:dev:reset
```

注意: `--dev` は**グローバル**なプロファイルフラグであり、一部のランナーによって消費されます。明示的に指定する必要がある場合は、環境変数形式を使用してください：

```
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

`--reset` は設定、認証情報、セッション、および開発ワークスペースを（`rm` ではなく `trash` を使用して）ワイプし、デフォルトの開発セットアップを再作成します。ヒント：非開発ゲートウェイがすでに実行されている場合（launchd/systemd）、まず停止してください：

```bash
openclaw gateway stop
```

## 生ストリームロギング (OpenClaw)

OpenClawは、フィルタリング/フォーマット前の**生のアシスタントストリーム**をログに記録できます。これは、推論がプレーンテキストの差分として（または別々の思考ブロックとして）到着しているかどうかを確認する最良の方法です。CLI経由で有効にします：

```bash
pnpm gateway:watch --raw-stream
```

オプションのパスオーバーライド：

```bash
pnpm gateway:watch --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl
```

同等の環境変数：

```
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

デフォルトファイル: `~/.openclaw/logs/raw-stream.jsonl`

## 生チャンクロギング (pi-mono)

ブロックに解析される前の**生のOpenAI互換チャンク**をキャプチャするために、pi-monoは別のロガーを公開しています：

```
PI_RAW_STREAM=1
```

オプションのパス：

```
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

デフォルトファイル: `~/.pi-mono/logs/raw-openai-completions.jsonl`

> 注意: これはpi-monoの `openai-completions` プロバイダーを使用するプロセスによってのみ出力されます。

## 安全性に関する注意

-   生ストリームログには、完全なプロンプト、ツール出力、およびユーザーデータが含まれる可能性があります。
-   ログはローカルに保持し、デバッグ後に削除してください。
-   ログを共有する場合は、まずシークレットと個人識別情報を削除してください。

[環境変数](./environment.md)[テスト](./testing.md)