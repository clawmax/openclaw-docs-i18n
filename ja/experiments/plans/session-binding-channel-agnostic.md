

  実験

  
# セッションバインディング チャネル非依存計画

## 概要

このドキュメントは、長期的なチャネル非依存セッションバインディングモデルと、次の実装イテレーションの具体的な範囲を定義します。目標:

-   サブエージェントにバインドされたセッションのルーティングをコア機能とする
-   チャネル固有の動作はアダプターに保持する
-   通常のDiscord動作における後退を避ける

## この計画が存在する理由

現在の動作は以下を混在させています:

-   完了コンテンツポリシー
-   宛先ルーティングポリシー
-   Discord固有の詳細

これにより、以下のようなエッジケースが発生しました:

-   同時実行時のメインおよびスレッドへの重複配信
-   再利用されたバインディングマネージャーでの古いトークン使用
-   Webhook送信に対するアクティビティ計測の欠落

## イテレーション1の範囲

このイテレーションは意図的に限定されています。

### 1. チャネル非依存のコアインターフェースを追加

バインディングとルーティングのためのコアタイプとサービスインターフェースを追加します。提案されるコアタイプ:

```bash
export type BindingTargetKind = "subagent" | "session";
export type BindingStatus = "active" | "ending" | "ended";

export type ConversationRef = {
  channel: string;
  accountId: string;
  conversationId: string;
  parentConversationId?: string;
};

export type SessionBindingRecord = {
  bindingId: string;
  targetSessionKey: string;
  targetKind: BindingTargetKind;
  conversation: ConversationRef;
  status: BindingStatus;
  boundAt: number;
  expiresAt?: number;
  metadata?: Record<string, unknown>;
};
```

コアサービス契約:

```bash
export interface SessionBindingService {
  bind(input: {
    targetSessionKey: string;
    targetKind: BindingTargetKind;
    conversation: ConversationRef;
    metadata?: Record<string, unknown>;
    ttlMs?: number;
  }): Promise<SessionBindingRecord>;

  listBySession(targetSessionKey: string): SessionBindingRecord[];
  resolveByConversation(ref: ConversationRef): SessionBindingRecord | null;
  touch(bindingId: string, at?: number): void;
  unbind(input: {
    bindingId?: string;
    targetSessionKey?: string;
    reason: string;
  }): Promise<SessionBindingRecord[]>;
}
```

### 2. サブエージェント完了用の単一コア配信ルーターを追加

完了イベントのための単一の宛先解決パスを追加します。ルーター契約:

```bash
export interface BoundDeliveryRouter {
  resolveDestination(input: {
    eventKind: "task_completion";
    targetSessionKey: string;
    requester?: ConversationRef;
    failClosed: boolean;
  }): {
    binding: SessionBindingRecord | null;
    mode: "bound" | "fallback";
    reason: string;
  };
}
```

このイテレーションでは:

-   `task_completion` のみがこの新しいパスを通じてルーティングされる
-   他のイベント種別に対する既存のパスはそのまま維持される

### 3. Discordをアダプターとして維持

Discordは最初のアダプター実装として残ります。アダプターの責任:

-   スレッド会話の作成/再利用
-   Webhookまたはチャネル送信を介したバインドされたメッセージの送信
-   スレッド状態の検証 (アーカイブ/削除済み)
-   アダプターメタデータのマッピング (Webhook ID, スレッドID)

### 4. 現在既知の正確性の問題を修正

このイテレーションで必要なもの:

-   既存のスレッドバインディングマネージャーを再利用する際のトークン使用量の更新
-   WebhookベースのDiscord送信に対する送信アクティビティの記録
-   セッションモード完了に対してバインドされたスレッド宛先が選択された場合の、暗黙的なメインチャネルへのフォールバックを停止

### 5. 現在のランタイム安全デフォルトを維持

スレッドバインドされたスポーンが無効なユーザーに対する動作変更はありません。デフォルトは維持:

-   `channels.discord.threadBindings.spawnSubagentSessions = false`

結果:

-   通常のDiscordユーザーは現在の動作を維持
-   新しいコアパスは、有効な場合のバインドされたセッション完了ルーティングにのみ影響

## イテレーション1に含まれないもの

明示的に延期されるもの:

-   ACPバインディングターゲット (`targetKind: "acp"`)
-   Discordを超える新しいチャネルアダプター
-   すべての配信パスのグローバルな置き換え (`spawn_ack`, 将来の `subagent_message`)
-   プロトコルレベルの変更
-   すべてのバインディング永続化のためのストア移行/バージョニング再設計

ACPに関する注記:

-   インターフェース設計はACPの余地を残している
-   ACP実装はこのイテレーションでは開始されない

## ルーティング不変条件

これらの不変条件はイテレーション1で必須です。

-   宛先選択とコンテンツ生成は別々のステップである
-   セッションモード完了がアクティブなバインドされた宛先に解決された場合、配信はその宛先をターゲットとしなければならない
-   バインドされた宛先からメインチャネルへの隠れたリルートはない
-   フォールバック動作は明示的かつ観測可能でなければならない

## 互換性とロールアウト

互換性目標:

-   スレッドバインドスポーンがオフのユーザーに対して後退がない
-   このイテレーションでは非Discordチャネルに変更はない

ロールアウト:

1.  現在の機能ゲートの背後にインターフェースとルーターを実装。
2.  Discord完了モードのバインドされた配信をルーター経由でルーティング。
3.  非バインドフローのためのレガシーパスを維持。
4.  対象を絞ったテストとカナリアランタイムログで検証。

## イテレーション1で必要なテスト

必要な単体および統合カバレッジ:

-   マネージャートークンのローテーションは、マネージャー再利用後に最新のトークンを使用する
-   Webhook送信はチャネルアクティビティタイムスタンプを更新する
-   同じリクエスターチャネル内の2つのアクティブなバインドされたセッションがメインチャネルに重複配信しない
-   バインドされたセッションモード実行の完了は、スレッド宛先のみに解決する
-   無効化されたスポーンフラグはレガシー動作を変更しないまま維持する

## 提案される実装ファイル

コア:

-   `src/infra/outbound/session-binding-service.ts` (新規)
-   `src/infra/outbound/bound-delivery-router.ts` (新規)
-   `src/agents/subagent-announce.ts` (完了宛先解決統合)

Discordアダプターとランタイム:

-   `src/discord/monitor/thread-bindings.manager.ts`
-   `src/discord/monitor/reply-delivery.ts`
-   `src/discord/send.outbound.ts`

テスト:

-   `src/discord/monitor/provider*.test.ts`
-   `src/discord/monitor/reply-delivery.test.ts`
-   `src/agents/subagent-announce.format.test.ts`

## イテレーション1の完了基準

-   コアインターフェースが存在し、完了ルーティングのために配線されている
-   上記の正確性修正がテスト付きでマージされている
-   セッションモードバインド実行において、メインおよびスレッドへの重複完了配信がない
-   バインドスポーンが無効なデプロイメントに対して動作変更がない
-   ACPは明示的に延期されたままである

[PTYおよびプロセス監視計画](./pty-process-supervision.md)[ワークスペースメモリ研究](../research/memory.md)