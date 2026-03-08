

  概念と内部構造

  
# タイムゾーン

OpenClawはタイムスタンプを標準化し、モデルが**単一の基準時間**を認識するようにします。

## メッセージエンベロープ（デフォルトはローカル）

受信メッセージは以下のようなエンベロープでラップされます：

```
[Provider ... 2026-01-05 16:26 PST] メッセージ本文
```

エンベロープ内のタイムスタンプは**デフォルトでホストローカル**であり、分単位の精度です。これは以下の設定で上書きできます：

```json
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | IANA タイムゾーン
      envelopeTimestamp: "on", // "on" | "off"
      envelopeElapsed: "on", // "on" | "off"
    },
  },
}
```

-   `envelopeTimezone: "utc"` はUTCを使用します。
-   `envelopeTimezone: "user"` は `agents.defaults.userTimezone` を使用します（設定されていない場合はホストのタイムゾーンにフォールバック）。
-   固定オフセットには明示的なIANAタイムゾーン（例：`"Europe/Vienna"`）を使用します。
-   `envelopeTimestamp: "off"` はエンベロープヘッダーから絶対タイムスタンプを削除します。
-   `envelopeElapsed: "off"` は経過時間の接尾辞（`+2m` スタイル）を削除します。

### 例

**ローカル（デフォルト）：**

```
[Signal Alice +1555 2026-01-18 00:19 PST] こんにちは
```

**固定タイムゾーン：**

```
[Signal Alice +1555 2026-01-18 06:19 GMT+1] こんにちは
```

**経過時間：**

```
[Signal Alice +1555 +2m 2026-01-18T05:19Z] フォローアップ
```

## ツールペイロード（生のプロバイダーデータ + 正規化フィールド）

ツール呼び出し（`channels.discord.readMessages`、`channels.slack.readMessages` など）は**生のプロバイダータイムスタンプ**を返します。一貫性のために正規化フィールドも付加します：

-   `timestampMs` (UTCエポックミリ秒)
-   `timestampUtc` (ISO 8601 UTC文字列)

生のプロバイダーフィールドは保持されます。

## システムプロンプト用のユーザータイムゾーン

モデルにユーザーのローカルタイムゾーンを伝えるには、`agents.defaults.userTimezone` を設定します。設定されていない場合、OpenClawは**実行時のホストタイムゾーン**を解決します（設定ファイルへの書き込みは行われません）。

```json
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

システムプロンプトには以下が含まれます：

-   ローカル時間とタイムゾーンを含む `現在の日付と時刻` セクション
-   `時間形式: 12時間制` または `24時間制`

プロンプトの形式は `agents.defaults.timeFormat` (`auto` | `12` | `24`) で制御できます。完全な動作と例については [日付と時刻](../date-time.md) を参照してください。

[利用状況の追跡](./usage-tracking.md)[クレジット](../reference/credits.md)