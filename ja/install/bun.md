

  その他のインストール方法

  
# Bun (実験的)

目標: **Bun** を使用してこのリポジトリを実行する (オプション、WhatsApp/Telegram には推奨されません)。pnpm ワークフローから逸脱しないようにします。⚠️ **Gateway ランタイム (WhatsApp/Telegram) には推奨されません** (バグが発生します)。本番環境では Node を使用してください。

## ステータス

-   Bun は、TypeScript を直接実行するためのオプションのローカルランタイムです (`bun run …`, `bun --watch …`)。
-   `pnpm` はビルドのデフォルトであり、完全にサポートされています (また、一部のドキュメントツールで使用されています)。
-   Bun は `pnpm-lock.yaml` を使用できず、それを無視します。

## インストール

デフォルト:

```bash
bun install
```

注: `bun.lock`/`bun.lockb` は gitignore されているため、どちらの方法でもリポジトリに変更は加わりません。*ロックファイルの書き込みを一切行わない* 場合は:

```bash
bun install --no-save
```

## ビルド / テスト (Bun)

```bash
bun run build
bun run vitest run
```

## Bun ライフサイクルスクリプト (デフォルトでブロック)

Bun は、明示的に信頼されない限り (`bun pm untrusted` / `bun pm trust`)、依存関係のライフサイクルスクリプトをブロックする場合があります。このリポジトリでは、一般的にブロックされるスクリプトは必要ありません:

-   `@whiskeysockets/baileys` `preinstall`: Node のメジャーバージョンが >= 20 であることを確認します (私たちは Node 22+ を実行しています)。
-   `protobufjs` `postinstall`: 互換性のないバージョンスキームに関する警告を出力します (ビルド成果物はありません)。

これらのスクリプトが必要な実際のランタイム問題が発生した場合は、明示的に信頼してください:

```bash
bun pm trust @whiskeysockets/baileys protobufjs
```

## 注意点

-   一部のスクリプトは依然として pnpm をハードコードしています (例: `docs:build`, `ui:*`, `protocol:check`)。当面はそれらを pnpm 経由で実行してください。

[Ansible](./ansible.md)[アップデート](./updating.md)