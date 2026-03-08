

  拡張機能

  
# OpenProse

OpenProseは、AIセッションをオーケストレーションするためのポータブルでマークダウンファーストのワークフロー形式です。OpenClawでは、OpenProseスキルパックと`/prose`スラッシュコマンドをインストールするプラグインとして提供されます。プログラムは`.prose`ファイルに記述され、明示的な制御フローで複数のサブエージェントを生成できます。公式サイト: [https://www.prose.md](https://www.prose.md)

## できること

-   明示的な並列処理によるマルチエージェントの調査と統合。
-   繰り返し可能で承認セーフなワークフロー（コードレビュー、インシデントトリアージ、コンテンツパイプライン）。
-   サポートされているエージェントランタイム間で実行可能な再利用可能な`.prose`プログラム。

## インストールと有効化

バンドルされたプラグインはデフォルトで無効になっています。OpenProseを有効にします:

```bash
openclaw plugins enable open-prose
```

プラグインを有効にした後は、Gatewayを再起動してください。開発/ローカルチェックアウトの場合: `openclaw plugins install ./extensions/open-prose` 関連ドキュメント: [プラグイン](./tools/plugin.md), [プラグインマニフェスト](./plugins/manifest.md), [スキル](./tools/skills.md)。

## スラッシュコマンド

OpenProseは、ユーザーが呼び出せるスキルコマンドとして`/prose`を登録します。これはOpenProse VMの指示にルーティングされ、内部でOpenClawツールを使用します。一般的なコマンド:

```bash
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

## 例: シンプルな.proseファイル

```bash
# 2つのエージェントが並列で実行される調査と統合。

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```

## ファイルの場所

OpenProseはワークスペース内の`.prose/`以下に状態を保持します:

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/
└── agents/
```

ユーザーレベルの永続的なエージェントは以下に配置されます:

```
~/.prose/agents/
```

## 状態モード

OpenProseは複数の状態バックエンドをサポートしています:

-   **filesystem** (デフォルト): `.prose/runs/...`
-   **in-context**: 一時的、小規模プログラム向け
-   **sqlite** (実験的): `sqlite3`バイナリが必要
-   **postgres** (実験的): `psql`と接続文字列が必要

注意点:

-   sqlite/postgresはオプトインで実験的な機能です。
-   postgresの認証情報はサブエージェントのログに流れるため、専用の最小権限のDBを使用してください。

## リモートプログラム

`/prose run <handle/slug>`は`https://p.prose.md//`に解決されます。直接のURLはそのまま取得されます。これは`web_fetch`ツール（またはPOSTの場合は`exec`）を使用します。

## OpenClawランタイムマッピング

OpenProseプログラムはOpenClawのプリミティブにマッピングされます:

| OpenProse 概念 | OpenClaw ツール |
| --- | --- |
| セッション生成 / タスクツール | `sessions_spawn` |
| ファイル読み書き | `read` / `write` |
| Web取得 | `web_fetch` |

これらのツールがツール許可リストでブロックされている場合、OpenProseプログラムは失敗します。[スキル設定](./tools/skills-config.md)を参照してください。

## セキュリティと承認

`.prose`ファイルはコードのように扱ってください。実行前にレビューします。副作用を制御するには、OpenClawのツール許可リストと承認ゲートを使用します。決定論的で承認ゲート付きのワークフローについては、[Lobster](./tools/lobster.md)と比較してください。

[プラグイン エージェント ツール](./plugins/agent-tools.md)[フック](./automation/hooks.md)