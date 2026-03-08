title: "OpenClaw ループ検出ツールの設定とセットアップガイド"
description: "エージェントが反復的で非生産的なツール呼び出しパターンに陥るのを防ぐための、OpenClawのループ検出ツールの設定と有効化方法について学びます。"
keywords: ["ループ検出", "openclaw ツール", "エージェントガードレール", "ツール呼び出しパターン", "サーキットブレーカー", "エージェント設定", "反復ループ", "ポーリング検出"]
---

  組み込みツール

  
# ツールループ検出

OpenClawは、エージェントが繰り返しのツール呼び出しパターンに陥るのを防ぐことができます。このガードは**デフォルトで無効**です。厳格な設定では正当な繰り返し呼び出しもブロックする可能性があるため、必要な場合にのみ有効化してください。

## この機能の目的

-   進捗のない反復的なシーケンスを検出します。
-   高頻度で結果のないループ（同じツール、同じ入力、繰り返されるエラー）を検出します。
-   既知のポーリングツールに対する特定の繰り返し呼び出しパターンを検出します。

## 設定ブロック

グローバルデフォルト:

```json
{
  tools: {
    loopDetection: {
      enabled: false,
      historySize: 30,
      warningThreshold: 10,
      criticalThreshold: 20,
      globalCircuitBreakerThreshold: 30,
      detectors: {
        genericRepeat: true,
        knownPollNoProgress: true,
        pingPong: true,
      },
    },
  },
}
```

エージェントごとの上書き設定（オプション）:

```json
{
  agents: {
    list: [
      {
        id: "safe-runner",
        tools: {
          loopDetection: {
            enabled: true,
            warningThreshold: 8,
            criticalThreshold: 16,
          },
        },
      },
    ],
  },
}
```

### フィールドの動作

-   `enabled`: マスタースイッチ。`false`の場合、ループ検出は実行されません。
-   `historySize`: 分析のために保持される直近のツール呼び出しの数。
-   `warningThreshold`: パターンを警告のみとして分類する前の閾値。
-   `criticalThreshold`: 反復ループパターンをブロックする閾値。
-   `globalCircuitBreakerThreshold`: グローバルな進捗なしブレーカーの閾値。
-   `detectors.genericRepeat`: 同じツール＋同じパラメータの繰り返しパターンを検出します。
-   `detectors.knownPollNoProgress`: 状態変化のない既知のポーリング様パターンを検出します。
-   `detectors.pingPong`: 交互に行われるピンポンパターンを検出します。

## 推奨セットアップ

-   `enabled: true`で開始し、デフォルトは変更しない。
-   閾値は `warningThreshold < criticalThreshold < globalCircuitBreakerThreshold` の順序で維持する。
-   誤検知が発生した場合:
    -   `warningThreshold` および/または `criticalThreshold` を引き上げる
    -   （オプションで）`globalCircuitBreakerThreshold` を引き上げる
    -   問題を引き起こしている検出器のみを無効化する
    -   厳格さを緩和するために `historySize` を減らす

## ログと期待される動作

ループが検出されると、OpenClawはループイベントを報告し、深刻度に応じて次のツールサイクルをブロックまたは抑制します。これにより、通常のツールアクセスを維持しながら、暴走するトークン消費やロックアップからユーザーを保護します。

-   まず警告と一時的な抑制を優先します。
-   反復的な証拠が蓄積された場合にのみ、エスカレーションします。

## 注意事項

-   `tools.loopDetection` はエージェントレベルの上書き設定とマージされます。
-   エージェントごとの設定は、グローバル値を完全に上書きまたは拡張します。
-   設定が存在しない場合、ガードレールはオフのままです。

[Lobster](./lobster.md)[Reactions](./reactions.md)