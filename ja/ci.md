title: "OpenClaw CI パイプラインのドキュメントとジョブ概要"
description: "OpenClaw CI パイプライン、スマートスコーピング、ジョブの目的、チェック、テスト、リリースのためのローカル同等コマンドについて学びます。"
keywords: ["ci パイプライン", "継続的インテグレーション", "スマートスコーピング", "ci ジョブ", "ローカル開発コマンド", "pnpm check", "blacksmith runner", "コントリビュートワークフロー"]
---

  コントリビューティング

  
# CI パイプライン

CI は `main` へのすべてのプッシュとすべてのプルリクエストで実行されます。ドキュメントやネイティブコードのみが変更された場合、高コストなジョブをスキップするスマートスコーピングを使用します。

## ジョブ概要

| ジョブ | 目的 | 実行条件 |
| --- | --- | --- |
| `docs-scope` | ドキュメントのみの変更を検出 | 常時 |
| `changed-scope` | どの領域が変更されたか検出 (node/macos/android/windows) | ドキュメント以外のPR |
| `check` | TypeScript 型チェック、リント、フォーマット | `main` へのプッシュ、または Node 関連の変更を含むPR |
| `check-docs` | Markdown リント + 壊れたリンクチェック | ドキュメントが変更された場合 |
| `code-analysis` | LOC 閾値チェック (1000行) | PRのみ |
| `secrets` | 漏洩したシークレットを検出 | 常時 |
| `build-artifacts` | dist を一度ビルドし、他のジョブと共有 | ドキュメント以外、Node 変更あり |
| `release-check` | npm pack 内容の検証 | ビルド後 |
| `checks` | Node/Bun テスト + プロトコルチェック | ドキュメント以外、Node 変更あり |
| `checks-windows` | Windows 固有のテスト | ドキュメント以外、Windows 関連の変更あり |
| `macos` | Swift リント/ビルド/テスト + TS テスト | macOS 変更を含むPR |
| `android` | Gradle ビルド + テスト | ドキュメント以外、Android 変更あり |

## フェイルファストの順序

ジョブは、高コストなジョブが実行される前に安価なチェックが失敗するように順序付けられています:

1.  `docs-scope` + `code-analysis` + `check` (並列実行、〜1-2分)
2.  `build-artifacts` (上記の完了待ち)
3.  `checks`, `checks-windows`, `macos`, `android` (ビルド完了待ち)

スコープロジックは `scripts/ci-changed-scope.mjs` にあり、`src/scripts/ci-changed-scope.test.ts` のユニットテストでカバーされています。

## ランナー

| ランナー | ジョブ |
| --- | --- |
| `blacksmith-16vcpu-ubuntu-2404` | スコープ検出を含むほとんどの Linux ジョブ |
| `blacksmith-32vcpu-windows-2025` | `checks-windows` |
| `macos-latest` | `macos`, `ios` |

## ローカル同等コマンド

```bash
pnpm check          # 型チェック + リント + フォーマット
pnpm test           # vitest テスト
pnpm check:docs     # ドキュメントフォーマット + リント + 壊れたリンク
pnpm release:check  # npm pack の検証
```

[Pi 開発ワークフロー](./pi-dev.md)[ドキュメントハブ](./start/hubs.md)

---