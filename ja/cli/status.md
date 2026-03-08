

  CLI コマンド

  
# status

チャネルとセッションの診断。

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

注記:

-   `--deep` はライブプローブ (WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal) を実行します。
-   出力には、複数のエージェントが設定されている場合のエージェントごとのセッションストアが含まれます。
-   概要には、利用可能な場合、Gateway + ノードホストサービスのインストール/ランタイムステータスが含まれます。
-   概要には、更新チャネル + git SHA (ソースチェックアウト用) が含まれます。
-   更新情報は概要に表示されます。利用可能な更新がある場合、status は `openclaw update` を実行するヒントを表示します ([更新](../install/updating.md) を参照)。
-   読み取り専用のステータスサーフェス (`status`, `status --json`, `status --all`) は、可能な場合、対象となる設定パスのためにサポートされている SecretRefs を解決します。
-   サポートされているチャネル SecretRef が設定されているが、現在のコマンドパスで利用できない場合、status は読み取り専用のままとなり、クラッシュする代わりに劣化した出力を報告します。人間向けの出力には「設定されたトークンがこのコマンドパスで利用できません」などの警告が表示され、JSON出力には `secretDiagnostics` が含まれます。

[skills](./skills.md)[system](./system.md)

---