

  拡張機能

  
# Zalo Personal プラグイン

Zalo Personal は、ネイティブの `zca-js` を使用して通常の Zalo ユーザーアカウントを自動化するプラグインを介して OpenClaw をサポートします。

> **警告:** 非公式の自動化はアカウントの停止/利用停止につながる可能性があります。自己責任でご利用ください。

## 命名規則

チャネル ID は `zalouser` です。これは**個人の Zalo ユーザーアカウント**（非公式）を自動化することを明確にするためです。将来の公式 Zalo API 統合の可能性のために、`zalo` は予約済みとして保持します。

## 実行環境

このプラグインは**Gateway プロセス内で実行されます**。リモート Gateway を使用する場合は、**Gateway を実行しているマシン**にインストール/設定し、その後 Gateway を再起動してください。外部の `zca`/`openzca` CLI バイナリは必要ありません。

## インストール

### オプション A: npm からインストール

```bash
openclaw plugins install @openclaw/zalouser
```

その後、Gateway を再起動してください。

### オプション B: ローカルフォルダからインストール（開発用）

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

その後、Gateway を再起動してください。

## 設定

チャネル設定は `channels.zalouser` の下にあります（`plugins.entries.*` ではありません）:

```json
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

## CLI

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```

## エージェントツール

ツール名: `zalouser` アクション: `send`, `image`, `link`, `friends`, `groups`, `me`, `status` チャネルメッセージアクションは、メッセージリアクション用の `react` もサポートします。

[Voice Call プラグイン](./voice-call.md)[プラグインマニフェスト](./manifest.md)