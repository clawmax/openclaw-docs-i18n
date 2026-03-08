title: "Telegram、WhatsApp、Discord、MS Teams向け自動化投票"
description: "CLI、RPC、Agentツールを使用してTelegram、WhatsApp、Discord、MS Teamsで投票を作成する方法を学びます。チャネル固有のオプションと例を含みます。"
keywords: ["自動化投票", "telegram 投票", "whatsapp 投票", "discord 投票", "msteams 投票", "openclaw cli", "poll api", "マルチチャネル投票"]
---

  自動化

  
# 投票

## 対応チャネル

-   Telegram
-   WhatsApp (webチャネル)
-   Discord
-   MS Teams (Adaptive Cards)

## CLI

```bash
# Telegram
openclaw message poll --channel telegram --target 123456789 \
  --poll-question "Ship it?" --poll-option "Yes" --poll-option "No"
openclaw message poll --channel telegram --target -1001234567890:topic:42 \
  --poll-question "Pick a time" --poll-option "10am" --poll-option "2pm" \
  --poll-duration-seconds 300

# WhatsApp
openclaw message poll --target +15555550123 \
  --poll-question "Lunch today?" --poll-option "Yes" --poll-option "No" --poll-option "Maybe"
openclaw message poll --target 123456789@g.us \
  --poll-question "Meeting time?" --poll-option "10am" --poll-option "2pm" --poll-option "4pm" --poll-multi

# Discord
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Snack?" --poll-option "Pizza" --poll-option "Sushi"
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Plan?" --poll-option "A" --poll-option "B" --poll-duration-hours 48

# MS Teams
openclaw message poll --channel msteams --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" --poll-option "Pizza" --poll-option "Sushi"
```

オプション:

-   `--channel`: `whatsapp` (デフォルト), `telegram`, `discord`, または `msteams`
-   `--poll-multi`: 複数オプションの選択を許可
-   `--poll-duration-hours`: Discord専用 (省略時はデフォルト24時間)
-   `--poll-duration-seconds`: Telegram専用 (5-600秒)
-   `--poll-anonymous` / `--poll-public`: Telegram専用の投票表示設定

## Gateway RPC

メソッド: `poll` パラメータ:

-   `to` (string, 必須)
-   `question` (string, 必須)
-   `options` (string\[\], 必須)
-   `maxSelections` (number, オプション)
-   `durationHours` (number, オプション)
-   `durationSeconds` (number, オプション, Telegram専用)
-   `isAnonymous` (boolean, オプション, Telegram専用)
-   `channel` (string, オプション, デフォルト: `whatsapp`)
-   `idempotencyKey` (string, 必須)

## チャネルごとの違い

-   Telegram: 2-10個の選択肢。`threadId` または `:topic:` ターゲットによるフォーラムトピックをサポート。`durationHours` の代わりに `durationSeconds` を使用、5-600秒に制限。匿名投票と公開投票をサポート。
-   WhatsApp: 2-12個の選択肢、`maxSelections` は選択肢の数以内である必要あり、`durationHours` は無視。
-   Discord: 2-10個の選択肢、`durationHours` は1-768時間に制限 (デフォルト24時間)。`maxSelections > 1` は複数選択を有効化；Discordは厳密な選択数制限をサポートしません。
-   MS Teams: Adaptive Card投票 (OpenClaw管理)。ネイティブの投票APIはなし；`durationHours` は無視。

## Agentツール (Message)

`message` ツールを `poll` アクション (`to`, `pollQuestion`, `pollOption`, オプションで `pollMulti`, `pollDurationHours`, `channel`) と共に使用。Telegramの場合、ツールは `pollDurationSeconds`, `pollAnonymous`, `pollPublic` も受け付けます。投票作成には `action: "poll"` を使用。`action: "send"` で渡される投票フィールドは拒否されます。注記: Discordには「正確にN個選択」モードはありません；`pollMulti` は複数選択にマッピングされます。Teamsの投票はAdaptive Cardsとしてレンダリングされ、投票を `~/.openclaw/msteams-polls.json` に記録するためにゲートウェイのオンライン状態が必要です。

[Gmail PubSub](./gmail-pubsub.md)[認証監視](./auth-monitoring.md)