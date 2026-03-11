

  ホーム

  
# OpenClaw

  ![](../images/openclaw-logo-text-dark.png)
  ![](../images/openclaw-logo-text.png)

> *"角質除去！角質除去！"* — 宇宙ザリガニ、おそらく

**WhatsApp、Telegram、Discord、iMessage などを横断する AI エージェント用の OS 非依存ゲートウェイ。** メッセージを送信すると、ポケットからエージェントの応答が返ってきます。プラグインで Mattermost などが追加可能です。

  
  
  

## OpenClaw とは？

OpenClaw は、あなたのお気に入りのチャットアプリ — WhatsApp、Telegram、Discord、iMessage など — を Pi のような AI コーディングエージェントに接続する**セルフホスト型ゲートウェイ**です。あなた自身のマシン（またはサーバー）上で単一のゲートウェイプロセスを実行し、それがメッセージングアプリと常時利用可能な AI アシスタントとの間の架け橋となります。

**対象ユーザーは？** データの制御を諦めたり、ホスト型サービスに依存することなく、どこからでもメッセージを送れる個人用 AI アシスタントを望む開発者やパワーユーザーです。

**何が違うのか？**

- **セルフホスト**: あなたのハードウェア、あなたのルールで実行
- **マルチチャネル**: 単一のゲートウェイで WhatsApp、Telegram、Discord などを同時に提供
- **エージェントネイティブ**: ツール使用、セッション、メモリ、マルチエージェントルーティングに対応したコーディングエージェント向けに構築
- **オープンソース**: MIT ライセンス、コミュニティ駆動

**必要なものは？** Node 22+、選択したプロバイダーの API キー、そして 5 分間。最高の品質とセキュリティのためには、利用可能な最新世代の最強モデルを使用してください。

## 仕組み

ゲートウェイは、セッション、ルーティング、チャネル接続に関する唯一の信頼できる情報源です。

## 主な機能

  - **マルチチャネルゲートウェイ** — 単一のゲートウェイプロセスで WhatsApp、Telegram、Discord、iMessage に対応。

  - **プラグインチャネル** — 拡張パッケージで Mattermost などを追加。

  - **マルチエージェントルーティング** — エージェント、ワークスペース、または送信者ごとに分離されたセッション。

  - **メディアサポート** — 画像、音声、ドキュメントの送受信。

  - **Web コントロール UI** — チャット、設定、セッション、ノード用のブラウザダッシュボード。

  

## クイックスタート

  ### OpenClaw をインストール

```bash
npm install -g openclaw@latest
```

  

  ### オンボーディングしてサービスをインストール

```bash
openclaw onboard --install-daemon
```

  

  ### WhatsApp をペアリングしてゲートウェイを起動

```bash
openclaw channels login
openclaw gateway --port 18789
```

  

完全なインストールと開発環境のセットアップが必要ですか？ [クイックスタート](./start/quickstart.md) を参照してください。

## ダッシュボード

ゲートウェイ起動後、ブラウザのコントロール UI を開きます。

- ローカルデフォルト: [http://127.0.0.1:18789/](http://127.0.0.1:18789/)
- リモートアクセス: [Web サーフェス](./web.md) および [Tailscale](./gateway/tailscale.md)

  ![](../images/whatsapp-openclaw.jpg)

## 設定（オプション）

設定は `~/.openclaw/openclaw.json` に保存されます。

- **何もしない**場合、OpenClaw は RPC モードのバンドルされた Pi バイナリを送信者ごとのセッションで使用します。
- ロックダウンしたい場合は、`channels.whatsapp.allowFrom` と（グループ用の）メンションルールから始めてください。

例:

```json
{
  "channels": {
    "whatsapp": {
      "allowFrom": ["+15555550123"],
      "groups": { "*": { "requireMention": true } }
    }
  },
  "messages": { "groupChat": { "mentionPatterns": ["@openclaw"] } }
}
```

## ここから始める

  
  
  
  
  
  

## 詳細情報

  
  
  
  
  

