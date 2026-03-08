

  設定

  
# ブロードキャストグループ

**ステータス:** 実験的  
**バージョン:** 2026.1.9で追加

## 概要

ブロードキャストグループを使用すると、複数のエージェントが同じメッセージを同時に処理して応答できます。これにより、単一のWhatsAppグループまたはDMで連携する専門エージェントチームを作成できます — すべて1つの電話番号を使用します。現在の範囲: **WhatsAppのみ** (ウェブチャネル)。ブロードキャストグループは、チャネルの許可リストとグループアクティベーションルールの後に評価されます。WhatsAppグループでは、これはOpenClawが通常応答するとき（例えば、メンション時、グループ設定に依存）にブロードキャストが発生することを意味します。

## ユースケース

### 1\. 専門エージェントチーム

原子的で焦点を絞った責任を持つ複数のエージェントをデプロイ:

```
グループ: "開発チーム"
エージェント:
  - CodeReviewer (コードスニペットをレビュー)
  - DocumentationBot (ドキュメントを生成)
  - SecurityAuditor (脆弱性をチェック)
  - TestGenerator (テストケースを提案)
```

各エージェントは同じメッセージを処理し、専門的な視点を提供します。

### 2\. 多言語サポート

```
グループ: "国際サポート"
エージェント:
  - Agent_EN (英語で応答)
  - Agent_DE (ドイツ語で応答)
  - Agent_ES (スペイン語で応答)
```

### 3\. 品質保証ワークフロー

```
グループ: "カスタマーサポート"
エージェント:
  - SupportAgent (回答を提供)
  - QAAgent (品質をレビュー、問題が見つかった場合のみ応答)
```

### 4\. タスク自動化

```
グループ: "プロジェクト管理"
エージェント:
  - TaskTracker (タスクデータベースを更新)
  - TimeLogger (費やした時間を記録)
  - ReportGenerator (サマリーを作成)
```

## 設定

### 基本設定

トップレベルに `broadcast` セクションを追加します (`bindings` の隣)。キーはWhatsAppピアIDです:

-   グループチャット: グループJID (例: `120363403215116621@g.us`)
-   DM: E.164電話番号 (例: `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**結果:** OpenClawがこのチャットで応答するとき、3つのエージェントすべてが実行されます。

### 処理戦略

エージェントがメッセージを処理する方法を制御:

#### 並列 (デフォルト)

すべてのエージェントが同時に処理:

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

#### 順次

エージェントは順番に処理 (前のエージェントが終了するのを待つ):

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

### 完全な例

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "コードレビュワー",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "セキュリティ監査人",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "ドキュメントジェネレーター",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

## 仕組み

### メッセージフロー

1.  **受信メッセージ** がWhatsAppグループに到着
2.  **ブロードキャストチェック**: システムはピアIDが `broadcast` にあるかチェック
3.  **ブロードキャストリストにある場合**:
    -   リストされたすべてのエージェントがメッセージを処理
    -   各エージェントは独自のセッションキーと分離されたコンテキストを持つ
    -   エージェントは並列（デフォルト）または順次で処理
4.  **ブロードキャストリストにない場合**:
    -   通常のルーティングが適用される（最初に一致するバインディング）

注意: ブロードキャストグループは、チャネルの許可リストやグループアクティベーションルール（メンション/コマンドなど）をバイパスしません。処理対象となるメッセージに対して*どのエージェントが実行されるか*を変更するだけです。

### セッション分離

ブロードキャストグループ内の各エージェントは完全に分離された状態を維持します:

-   **セッションキー** (`agent:alfred:whatsapp:group:120363...` 対 `agent:baerbel:whatsapp:group:120363...`)
-   **会話履歴** (エージェントは他のエージェントのメッセージを見ない)
-   **ワークスペース** (設定されている場合は別々のサンドボックス)
-   **ツールアクセス** (異なる許可/拒否リスト)
-   **メモリ/コンテキスト** (別々のIDENTITY.md、SOUL.mdなど)
-   **グループコンテキストバッファ** (コンテキストに使用される最近のグループメッセージ) はピアごとに共有されるため、すべてのブロードキャストエージェントはトリガー時に同じコンテキストを見る

これにより、各エージェントは以下を持つことができます:

-   異なる人格
-   異なるツールアクセス (例: 読み取り専用 対 読み書き)
-   異なるモデル (例: opus 対 sonnet)
-   異なるスキルがインストール済み

### 例: 分離されたセッション

グループ `120363403215116621@g.us` にエージェント `["alfred", "baerbel"]` がいる場合: **Alfredのコンテキスト:**

```yaml
セッション: agent:alfred:whatsapp:group:120363403215116621@g.us
履歴: [ユーザーメッセージ, alfredの以前の応答]
ワークスペース: /Users/pascal/openclaw-alfred/
ツール: read, write, exec
```

**Bärbelのコンテキスト:**

```yaml
セッション: agent:baerbel:whatsapp:group:120363403215116621@g.us
履歴: [ユーザーメッセージ, baerbelの以前の応答]
ワークスペース: /Users/pascal/openclaw-baerbel/
ツール: read only
```

## ベストプラクティス

### 1\. エージェントを焦点化する

各エージェントを単一の明確な責任で設計:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **良い:** 各エージェントに1つの仕事  
❌ **悪い:** 1つの汎用的な「dev-helper」エージェント

### 2\. 説明的な名前を使用する

各エージェントの役割を明確に:

```json
{
  "agents": {
    "security-scanner": { "name": "セキュリティスキャナー" },
    "code-formatter": { "name": "コードフォーマッター" },
    "test-generator": { "name": "テストジェネレーター" }
  }
}
```

### 3\. 異なるツールアクセスを設定する

エージェントに必要なツールのみを与える:

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] } // 読み取り専用
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] } // 読み書き
    }
  }
}
```

### 4\. パフォーマンスを監視する

多くのエージェントを使用する場合、以下を考慮:

-   速度のために `"strategy": "parallel"` (デフォルト) を使用
-   ブロードキャストグループを5-10エージェントに制限
-   より単純なエージェントには高速なモデルを使用

### 5\. 失敗を適切に処理する

エージェントは独立して失敗します。1つのエージェントのエラーは他のエージェントをブロックしません:

```
メッセージ → [エージェント A ✓, エージェント B ✗ エラー, エージェント C ✓]
結果: エージェントAとCが応答、エージェントBはエラーをログに記録
```

## 互換性

### プロバイダー

ブロードキャストグループは現在以下で動作:

-   ✅ WhatsApp (実装済み)
-   🚧 Telegram (計画中)
-   🚧 Discord (計画中)
-   🚧 Slack (計画中)

### ルーティング

ブロードキャストグループは既存のルーティングと併用可能:

```json
{
  "bindings": [
    {
      "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } },
      "agentId": "alfred"
    }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

-   `GROUP_A`: alfredのみ応答 (通常ルーティング)
-   `GROUP_B`: agent1 と agent2 が応答 (ブロードキャスト)

**優先順位:** `broadcast` は `bindings` より優先されます。

## トラブルシューティング

### エージェントが応答しない

**確認事項:**

1.  エージェントIDが `agents.list` に存在する
2.  ピアIDの形式が正しい (例: `120363403215116621@g.us`)
3.  エージェントが拒否リストにない

**デバッグ:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

### 1つのエージェントのみが応答する

**原因:** ピアIDが `bindings` にはあるが `broadcast` にない可能性があります。  
**修正:** ブロードキャスト設定に追加するか、バインディングから削除します。

### パフォーマンスの問題

**多くのエージェントで遅い場合:**

-   グループごとのエージェント数を減らす
-   軽量なモデルを使用 (opusの代わりにsonnet)
-   サンドボックスの起動時間を確認

## 例

### 例1: コードレビューチーム

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      {
        "id": "code-formatter",
        "workspace": "~/agents/formatter",
        "tools": { "allow": ["read", "write"] }
      },
      {
        "id": "security-scanner",
        "workspace": "~/agents/security",
        "tools": { "allow": ["read", "exec"] }
      },
      {
        "id": "test-coverage",
        "workspace": "~/agents/testing",
        "tools": { "allow": ["read", "exec"] }
      },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**ユーザーが送信:** コードスニペット  
**応答:**

-   code-formatter: 「インデントを修正し、型ヒントを追加しました」
-   security-scanner: 「⚠️ 12行目にSQLインジェクションの脆弱性があります」
-   test-coverage: 「カバレッジは45%、エラーケースのテストが不足しています」
-   docs-checker: 「関数 `process_data` のdocstringが不足しています」

### 例2: 多言語サポート

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

## APIリファレンス

### 設定スキーマ

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

### フィールド

-   `strategy` (オプション): エージェントの処理方法
    -   `"parallel"` (デフォルト): すべてのエージェントが同時に処理
    -   `"sequential"`: エージェントは配列順に処理
-   `[peerId]`: WhatsAppグループJID、E.164番号、またはその他のピアID
    -   値: メッセージを処理すべきエージェントIDの配列

## 制限事項

1.  **最大エージェント数:** 厳密な制限はないが、10以上のエージェントは遅くなる可能性あり
2.  **共有コンテキスト:** エージェントは互いの応答を見ない (設計による)
3.  **メッセージ順序:** 並列応答は任意の順序で到着する可能性あり
4.  **レート制限:** すべてのエージェントがWhatsAppのレート制限にカウントされる

## 将来の拡張

計画されている機能:

-   [ ] 共有コンテキストモード (エージェントが互いの応答を見る)
-   [ ] エージェント調整 (エージェントが互いに信号を送れる)
-   [ ] 動的エージェント選択 (メッセージ内容に基づいてエージェントを選択)
-   [ ] エージェント優先度 (一部のエージェントが他のエージェントより先に応答)

## 関連項目

-   [マルチエージェント設定](../tools/multi-agent-sandbox-tools.md)
-   [ルーティング設定](./channel-routing.md)
-   [セッション管理](../concepts/session.md)

[グループ](./groups.md)[チャネルルーティング](./channel-routing.md)