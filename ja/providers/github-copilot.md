title: "OpenClawでGitHub Copilotをモデルプロバイダーとして使用する方法"
description: "GitHub CopilotをOpenClawと統合する2つの方法（組み込みプロバイダーとCopilot Proxyプラグイン）を学びます。セットアップコマンドと設定を含みます。"
keywords: ["github copilot", "openclaw", "ai コーディングアシスタント", "モデルプロバイダー", "copilot proxy", "cli セットアップ", "auth login", "gpt-4o"]
---

  プロバイダー

  
# GitHub Copilot

## GitHub Copilotとは？

GitHub Copilotは、GitHubのAIコーディングアシスタントです。GitHubアカウントとプランに応じてCopilotモデルへのアクセスを提供します。OpenClawでは、Copilotをモデルプロバイダーとして2つの異なる方法で使用できます。

## OpenClawでCopilotを使用する2つの方法

### 1) 組み込みGitHub Copilotプロバイダー (github-copilot)

ネイティブのデバイスログインフローを使用してGitHubトークンを取得し、OpenClaw実行時にそれをCopilot APIトークンと交換します。これはVS Codeを必要としないため、**デフォルト**で最もシンプルな方法です。

### 2) Copilot Proxyプラグイン (copilot-proxy)

**Copilot Proxy** VS Code拡張機能をローカルブリッジとして使用します。OpenClawはプロキシの`/v1`エンドポイントと通信し、そこで設定したモデルリストを使用します。既にVS CodeでCopilot Proxyを実行している場合、またはそれを経由する必要がある場合に選択してください。プラグインを有効にし、VS Code拡張機能を実行し続ける必要があります。モデルプロバイダーとしてGitHub Copilotを使用します (`github-copilot`)。ログインコマンドはGitHubデバイスフローを実行し、認証プロファイルを保存し、そのプロファイルを使用するように設定を更新します。

## CLIセットアップ

```bash
openclaw models auth login-github-copilot
```

URLにアクセスしてワンタイムコードを入力するよう促されます。完了するまでターミナルを開いたままにしてください。

### オプションフラグ

```bash
openclaw models auth login-github-copilot --profile-id github-copilot:work
openclaw models auth login-github-copilot --yes
```

## デフォルトモデルの設定

```bash
openclaw models set github-copilot/gpt-4o
```

### 設定スニペット

```json
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } },
}
```

## 注意事項

-   対話型TTYが必要です。ターミナルで直接実行してください。
-   Copilotモデルの利用可否はプランに依存します。モデルが拒否された場合は、別のIDを試してください（例: `github-copilot/gpt-4.1`）。
-   ログインはGitHubトークンを認証プロファイルストアに保存し、OpenClaw実行時にCopilot APIトークンと交換します。

[Deepgram](./deepgram.md)[Hugging Face (Inference)](./huggingface.md)