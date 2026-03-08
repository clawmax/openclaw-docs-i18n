title: "OpenClaw Gateway ロギング設定と運用ガイド"
description: "OpenClaw gateway でのファイルおよびコンソールロギングの設定、ログレベルの設定、WebSocket ログの管理、機密データのための編集（リダクション）の使用方法について学びます。"
keywords: ["openclaw ロギング", "ゲートウェイログ", "ログ設定", "websocket ログ", "コンソールキャプチャ", "ログ編集", "json ログ", "ログレベル"]
---

  設定と運用

  
# ロギング

ユーザー向けの概要（CLI + コントロールUI + 設定）については、[/logging](../logging.md) を参照してください。OpenClaw には2つのログ「表面」があります：

-   **コンソール出力**（ターミナル / デバッグUIで見えるもの）。
-   **ファイルログ**（JSON 行）ゲートウェイロガーによって書き込まれるもの。

## ファイルベースのロガー

-   デフォルトのローテーションログファイルは `/tmp/openclaw/` の下にあります（1日1ファイル）： `openclaw-YYYY-MM-DD.log`
    -   日付はゲートウェイホストのローカルタイムゾーンを使用します。
-   ログファイルのパスとレベルは `~/.openclaw/openclaw.json` で設定できます：
    -   `logging.file`
    -   `logging.level`

ファイル形式は1行に1つのJSONオブジェクトです。コントロールUIの「ログ」タブは、ゲートウェイ（`logs.tail`）経由でこのファイルを末尾監視します。CLIでも同様のことができます：

```bash
openclaw logs --follow
```

**詳細出力とログレベル**

-   **ファイルログ**は `logging.level` によってのみ制御されます。
-   `--verbose` は**コンソールの詳細度**（およびWSログスタイル）にのみ影響し、ファイルログレベルを上げることは**ありません**。
-   詳細出力のみの詳細をファイルログにキャプチャするには、`logging.level` を `debug` または `trace` に設定します。

## コンソールキャプチャ

CLIは `console.log/info/warn/error/debug/trace` をキャプチャし、それらをファイルログに書き込みながら、stdout/stderrにも出力します。コンソールの詳細度は以下で独立して調整できます：

-   `logging.consoleLevel` (デフォルト `info`)
-   `logging.consoleStyle` (`pretty` | `compact` | `json`)

## ツールサマリー編集

詳細なツールサマリー（例：`🛠️ Exec: ...`）は、コンソールストリームに到達する前に機密トークンをマスクできます。これは**ツールのみ**であり、ファイルログは変更しません。

-   `logging.redactSensitive`: `off` | `tools` (デフォルト: `tools`)
-   `logging.redactPatterns`: 正規表現文字列の配列（デフォルトを上書き）
    -   生の正規表現文字列（自動 `gi`）、またはカスタムフラグが必要な場合は `/pattern/flags` を使用します。
    -   マッチした部分は、最初の6文字と最後の4文字を保持してマスクされます（長さ >= 18の場合）。それ以外の場合は `***` になります。
    -   デフォルトは、一般的なキー割り当て、CLIフラグ、JSONフィールド、Bearerヘッダー、PEMブロック、一般的なトークンプレフィックスをカバーします。

## ゲートウェイ WebSocket ログ

ゲートウェイはWebSocketプロトコルログを2つのモードで出力します：

-   **通常モード（`--verbose` なし）**: 「興味深い」RPC結果のみが出力されます：
    -   エラー (`ok=false`)
    -   遅い呼び出し（デフォルトしきい値: `>= 50ms`）
    -   パースエラー
-   **詳細モード（`--verbose`）**: すべてのWSリクエスト/レスポンストラフィックを出力します。

### WS ログスタイル

`openclaw gateway` はゲートウェイごとのスタイル切り替えをサポートします：

-   `--ws-log auto` (デフォルト): 通常モードは最適化済み；詳細モードはコンパクト出力を使用
-   `--ws-log compact`: 詳細時にコンパクト出力（ペアのリクエスト/レスポンス）
-   `--ws-log full`: 詳細時にフルなフレームごとの出力
-   `--compact`: `--ws-log compact` のエイリアス

例：

```bash
# 最適化済み（エラー/遅延のみ）
openclaw gateway

# すべてのWSトラフィックを表示（ペア）
openclaw gateway --verbose --ws-log compact

# すべてのWSトラフィックを表示（フルメタデータ）
openclaw gateway --verbose --ws-log full
```

## コンソールフォーマット（サブシステムロギング）

コンソールフォーマッタは**TTYを認識**し、一貫したプレフィックス付きの行を出力します。サブシステムロガーは出力をグループ化し、スキャンしやすく保ちます。動作：

-   **サブシステムプレフィックス** が各行に付きます（例：`[gateway]`, `[canvas]`, `[tailscale]`）
-   **サブシステムカラー**（サブシステムごとに安定）とレベルカラーリング
-   **出力がTTYの場合、または環境がリッチターミナルに見える場合にカラー**（`TERM`/`COLORTERM`/`TERM_PROGRAM`）、`NO_COLOR` を尊重
-   **短縮されたサブシステムプレフィックス**: 先頭の `gateway/` + `channels/` を削除し、最後の2セグメントを保持（例：`whatsapp/outbound`）
-   **サブシステムによるサブロガー**（自動プレフィックス + 構造化フィールド `{ subsystem }`）
-   **`logRaw()`** QR/UX出力用（プレフィックスなし、フォーマットなし）
-   **コンソールスタイル**（例：`pretty | compact | json`）
-   **コンソールログレベル** はファイルログレベルとは別（ファイルは `logging.level` が `debug`/`trace` に設定されている場合、完全な詳細を保持）
-   **WhatsApp メッセージ本文** は `debug` レベルでログ記録されます（`--verbose` を使用して表示）

これにより、既存のファイルログを安定させながら、対話型出力をスキャンしやすく保ちます。

[Doctor](./doctor.md)[Gateway Lock](./gateway-lock.md)

---