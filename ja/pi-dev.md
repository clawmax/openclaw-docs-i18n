

  開発者セットアップ

  
# Pi開発ワークフロー

このガイドは、OpenClawにおけるPi統合を扱うための適切なワークフローをまとめたものです。

## 型チェックとリンター

-   型チェックとビルド: `pnpm build`
-   リンター: `pnpm lint`
-   フォーマットチェック: `pnpm format`
-   プッシュ前の完全なゲート: `pnpm lint && pnpm build && pnpm test`

## Piテストの実行

Vitestを使用してPiに焦点を当てたテストセットを直接実行します:

```bash
pnpm test -- \
  "src/agents/pi-*.test.ts" \
  "src/agents/pi-embedded-*.test.ts" \
  "src/agents/pi-tools*.test.ts" \
  "src/agents/pi-settings.test.ts" \
  "src/agents/pi-tool-definition-adapter*.test.ts" \
  "src/agents/pi-extensions/**/*.test.ts"
```

ライブプロバイダーの演習を含めるには:

```
OPENCLAW_LIVE_TEST=1 pnpm test -- src/agents/pi-embedded-runner-extraparams.live.test.ts
```

これにより、主要なPiユニットスイートがカバーされます:

-   `src/agents/pi-*.test.ts`
-   `src/agents/pi-embedded-*.test.ts`
-   `src/agents/pi-tools*.test.ts`
-   `src/agents/pi-settings.test.ts`
-   `src/agents/pi-tool-definition-adapter.test.ts`
-   `src/agents/pi-extensions/*.test.ts`

## 手動テスト

推奨されるフロー:

-   ゲートウェイを開発モードで実行:
    -   `pnpm gateway:dev`
-   エージェントを直接トリガー:
    -   `pnpm openclaw agent --message "Hello" --thinking low`
-   対話型デバッグにTUIを使用:
    -   `pnpm tui`

ツール呼び出しの動作については、`read`または`exec`アクションを促し、ツールストリーミングとペイロード処理を確認できるようにします。

## クリーンスレートリセット

状態はOpenClaw状態ディレクトリの下に保存されます。デフォルトは`~/.openclaw`です。`OPENCLAW_STATE_DIR`が設定されている場合は、そのディレクトリを使用してください。すべてをリセットするには:

-   `openclaw.json` (設定用)
-   `credentials/` (認証プロファイルとトークン用)
-   `agents//sessions/` (エージェントセッション履歴用)
-   `agents//sessions.json` (セッションインデックス用)
-   `sessions/` (レガシーパスが存在する場合)
-   `workspace/` (空のワークスペースが必要な場合)

セッションのみをリセットしたい場合は、そのエージェントの`agents//sessions/`と`agents//sessions.json`を削除します。再認証を避けたい場合は`credentials/`は保持してください。

## 参考資料

-   [https://docs.openclaw.ai/testing](https://docs.openclaw.ai/testing)
-   [https://docs.openclaw.ai/start/getting-started](https://docs.openclaw.ai/start/getting-started)

[セットアップ](./start/setup.md)[CIパイプライン](./ci.md)

---