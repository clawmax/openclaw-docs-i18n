

  テンプレート

  
# デフォルト AGENTS.md

## 初回実行（推奨）

OpenClawはエージェント用に専用のワークスペースディレクトリを使用します。デフォルト: `~/.openclaw/workspace` (`agents.defaults.workspace` で設定可能)。

1.  ワークスペースを作成（まだ存在しない場合）:

```bash
mkdir -p ~/.openclaw/workspace
```

2.  デフォルトのワークスペーステンプレートをワークスペースにコピー:

```bash
cp docs/reference/templates/AGENTS.md ~/.openclaw/workspace/AGENTS.md
cp docs/reference/templates/SOUL.md ~/.openclaw/workspace/SOUL.md
cp docs/reference/templates/TOOLS.md ~/.openclaw/workspace/TOOLS.md
```

3.  オプション: パーソナルアシスタントスキル名簿が必要な場合は、AGENTS.mdをこのファイルで置き換え:

```bash
cp docs/reference/AGENTS.default.md ~/.openclaw/workspace/AGENTS.md
```

4.  オプション: `agents.defaults.workspace` を設定して別のワークスペースを選択（`~` をサポート）:

```json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

## 安全デフォルト

-   ディレクトリやシークレットをチャットにダンプしない。
-   明示的に指示されない限り、破壊的なコマンドを実行しない。
-   外部メッセージングサーフェスに部分的な/ストリーミング返信を送信しない（最終返信のみ）。

## セッション開始（必須）

-   `SOUL.md`、`USER.md`、`memory.md`、および `memory/` 内の今日と昨日のファイルを読み込む。
-   応答する前に実行すること。

## ソウル（必須）

-   `SOUL.md` はアイデンティティ、トーン、境界を定義する。最新の状態を保つ。
-   `SOUL.md` を変更した場合は、ユーザーに伝える。
-   各セッションは新規インスタンス。継続性はこれらのファイルに存在する。

## 共有スペース（推奨）

-   あなたはユーザーの声ではない。グループチャットや公開チャンネルでは注意する。
-   プライベートデータ、連絡先情報、内部メモを共有しない。

## メモリシステム（推奨）

-   日次ログ: `memory/YYYY-MM-DD.md`（必要に応じて `memory/` を作成）。
-   長期記憶: 永続的な事実、好み、決定事項用の `memory.md`。
-   セッション開始時に、存在すれば今日 + 昨日 + `memory.md` を読み込む。
-   記録するもの: 決定事項、好み、制約、未解決事項。
-   明示的に要求されない限り、シークレットは避ける。

## ツールとスキル

-   ツールはスキル内に存在。必要に応じて各スキルの `SKILL.md` に従う。
-   環境固有のメモは `TOOLS.md`（スキル用メモ）に保管。

## バックアップのヒント（推奨）

このワークスペースをClawdの「メモリ」として扱う場合は、gitリポジトリ（理想的にはプライベート）にして、`AGENTS.md` とメモリファイルがバックアップされるようにする。

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md
git commit -m "Add Clawd workspace"
# オプション: プライベートリモートを追加 + プッシュ
```

## OpenClawの機能

-   WhatsAppゲートウェイ + Piコーディングエージェントを実行し、アシスタントがチャットの読み書き、コンテキストの取得、ホストMac経由でのスキル実行を可能にする。
-   macOSアプリは権限（画面録画、通知、マイク）を管理し、バンドルされたバイナリ経由で `openclaw` CLIを公開する。
-   ダイレクトチャットはデフォルトでエージェントの `main` セッションに統合。グループは `agent:::group:` として分離されたまま（ルーム/チャンネル: `agent:::channel:`）。ハートビートはバックグラウンドタスクを維持。
-   キャンバスUIはフルスクリーンでネイティブオーバーレイを実行。重要なコントロールを左上/右上/下端に配置しない。レイアウトに明示的なガターを追加し、セーフエリアインセットに依存しない。

## コアスキル（設定 → スキルで有効化）

-   **mcporter** — 外部スキルバックエンドを管理するためのツールサーバーランタイム/CLI。
-   **Peekaboo** — オプションのAIビジョン分析付きの高速macOSスクリーンショット。
-   **camsnap** — RTSP/ONVIFセキュリティカメラからフレーム、クリップ、またはモーションアラートをキャプチャ。
-   **oracle** — セッション再生とブラウザ制御機能を備えたOpenAI対応エージェントCLI。
-   **eightctl** — ターミナルから睡眠を制御。
-   **imsg** — iMessage & SMSの送信、読み取り、ストリーミング。
-   **wacli** — WhatsApp CLI: 同期、検索、送信。
-   **discord** — Discordアクション: リアクション、ステッカー、投票。ターゲットには `user:` または `channel:` を使用（単なる数値IDは曖昧）。
-   **gog** — Google Suite CLI: Gmail、カレンダー、ドライブ、連絡先。
-   **spotify-player** — 検索/キュー/再生制御のためのターミナルSpotifyクライアント。
-   **sag** — ElevenLabs音声合成、macスタイルのsay UX。デフォルトでスピーカーにストリーミング。
-   **Sonos CLI** — スクリプトからSonosスピーカーを制御（発見/ステータス/再生/音量/グループ化）。
-   **blucli** — スクリプトからBluOSプレーヤーを再生、グループ化、自動化。
-   **OpenHue CLI** — シーンとオートメーションのためのPhilips Hue照明制御。
-   **OpenAI Whisper** — クイックディクテーションとボイスメール文字起こしのためのローカル音声認識。
-   **Gemini CLI** — 高速なQ&AのためのターミナルからのGoogle Geminiモデル。
-   **agent-tools** — オートメーションとヘルパースクリプトのためのユーティリティツールキット。

## 使用上の注意

-   スクリプト作成には `openclaw` CLIを優先。macアプリが権限を処理。
-   スキルタブからインストールを実行。バイナリが既に存在する場合はボタンが非表示になる。
-   アシスタントがリマインダーのスケジュール、受信トレイの監視、カメラキャプチャのトリガーをできるように、ハートビートを有効に保つ。
-   ブラウザ駆動の検証には、OpenClaw管理のChromeプロファイルを使用した `openclaw browser`（タブ/ステータス/スクリーンショット）を使用。
-   DOM検査には、`openclaw browser eval|query|dom|snapshot` を使用（機械出力が必要な場合は `--json`/`--out` を使用）。
-   インタラクションには、`openclaw browser click|type|hover|drag|select|upload|press|wait|navigate|back|evaluate|run` を使用（click/typeにはスナップショット参照が必要。CSSセレクターには `evaluate` を使用）。

[デバイスモデルデータベース](./device-models.md)[AGENTS.md テンプレート](./templates/AGENTS.md)