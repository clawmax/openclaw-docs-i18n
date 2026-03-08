

  リリースノート

  
# リリースチェックリスト

リポジトリのルートで `pnpm` (Node 22+) を使用します。タグ付け/公開前に作業ツリーをクリーンに保ってください。

## オペレーターのトリガー

オペレーターが「リリース」と言ったら、すぐにこの事前チェックを行います（ブロックされない限り追加の質問は不要）：

-   このドキュメントと `docs/platforms/mac/release.md` を読む。
-   `~/.profile` から環境変数を読み込み、`SPARKLE_PRIVATE_KEY_FILE` + App Store Connect 変数が設定されていることを確認する（SPARKLE\_PRIVATE\_KEY\_FILE は `~/.profile` に存在するはず）。
-   必要に応じて `~/Library/CloudStorage/Dropbox/Backup/Sparkle` から Sparkle キーを使用する。

1.  **バージョン & メタデータ**

-   [ ]  `package.json` のバージョンを更新する（例: `2026.1.29`）。
-   [ ]  `pnpm plugins:sync` を実行して、拡張機能パッケージのバージョンと変更履歴を同期させる。
-   [ ]  [`src/version.ts`](https://github.com/openclaw/openclaw/blob/main/src/version.ts) の CLI/バージョン文字列と [`src/web/session.ts`](https://github.com/openclaw/openclaw/blob/main/src/web/session.ts) の Baileys ユーザーエージェントを更新する。
-   [ ]  パッケージメタデータ（名前、説明、リポジトリ、キーワード、ライセンス）と `bin` マップが `openclaw` に対して [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs) を指していることを確認する。
-   [ ]  依存関係が変更された場合は、`pnpm install` を実行して `pnpm-lock.yaml` を最新の状態にする。

2.  **ビルド & 成果物**

-   [ ]  A2UI の入力が変更された場合は、`pnpm canvas:a2ui:bundle` を実行し、更新された [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js) があればコミットする。
-   [ ]  `pnpm run build` (`dist/` を再生成する)。
-   [ ]  npm パッケージの `files` に必要なすべての `dist/*` フォルダ（特にヘッドレスノード + ACP CLI 用の `dist/node-host/**` と `dist/acp/**`）が含まれていることを確認する。
-   [ ]  `dist/build-info.json` が存在し、期待される `commit` ハッシュを含んでいることを確認する（CLI バナーは npm インストール時にこれを使用する）。
-   [ ]  オプション: ビルド後に `npm pack --pack-destination /tmp` を実行し、tarball の内容を確認して GitHub リリース用に手元に置いておく（**コミットしない**）。

3.  **変更履歴 & ドキュメント**

-   [ ]  `CHANGELOG.md` をユーザー向けのハイライトで更新する（ファイルがなければ作成する）；エントリはバージョン順に厳密に降順で保つ。
-   [ ]  README の例やフラグが現在の CLI の動作（特に新しいコマンドやオプション）と一致していることを確認する。

4.  **検証**

-   [ ]  `pnpm build`
-   [ ]  `pnpm check`
-   [ ]  `pnpm test` (またはカバレッジ出力が必要な場合は `pnpm test:coverage`)
-   [ ]  `pnpm release:check` (npm pack の内容を検証)
-   [ ]  `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke` (Docker インストールスモークテスト、高速パス；リリース前に必須)
    -   直前の npm リリースが壊れていることがわかっている場合は、プリインストールステップで `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<last-good-version>` または `OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1` を設定する。
-   [ ]  (オプション) フルインストーラースモーク（非root + CLI カバレッジを追加）: `pnpm test:install:smoke`
-   [ ]  (オプション) インストーラー E2E (Docker, `curl -fsSL https://openclaw.ai/install.sh | bash` を実行し、オンボーディング後、実際のツール呼び出しを実行):
    -   `pnpm test:install:e2e:openai` (`OPENAI_API_KEY` が必要)
    -   `pnpm test:install:e2e:anthropic` (`ANTHROPIC_API_KEY` が必要)
    -   `pnpm test:install:e2e` (両方のキーが必要；両方のプロバイダーを実行)
-   [ ]  (オプション) 送信/受信パスに影響を与える変更を行った場合は、Web ゲートウェイをスポットチェックする。

5.  **macOS アプリ (Sparkle)**

-   [ ]  macOS アプリをビルド + 署名し、配布用に zip 化する。
-   [ ]  Sparkle アプキャストを生成し（[`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh) 経由の HTML ノート）、`appcast.xml` を更新する。
-   [ ]  アプリ zip（およびオプションの dSYM zip）を GitHub リリースに添付する準備をしておく。
-   [ ]  正確なコマンドと必要な環境変数については [macOS リリース](../platforms/mac/release.md) に従う。
    -   `APP_BUILD` は数値で単調増加する必要がある（`-beta` なし）。これにより Sparkle がバージョンを正しく比較できる。
    -   公証する場合は、App Store Connect API 環境変数から作成された `openclaw-notary` キーチェーンプロファイルを使用する（[macOS リリース](../platforms/mac/release.md) を参照）。

6.  **公開 (npm)**

-   [ ]  git ステータスがクリーンであることを確認し、必要に応じてコミットしてプッシュする。
-   [ ]  必要に応じて `npm login` (2FA を確認)。
-   [ ]  `npm publish --access public` (プレリリースの場合は `--tag beta` を使用)。
-   [ ]  レジストリを確認: `npm view openclaw version`, `npm view openclaw dist-tags`, `npx -y openclaw@X.Y.Z --version` (または `--help`)。

### トラブルシューティング (2.0.0-beta2 リリースからのメモ)

-   **npm pack/publish がハングする、または巨大な tarball が生成される**: `dist/OpenClaw.app` 内の macOS アプリバンドル（およびリリース zip）がパッケージに取り込まれてしまう。`package.json` の `files` で公開内容をホワイトリスト化して修正する（dist サブディレクトリ、docs、skills を含め、アプリバンドルは除外）。`npm pack --dry-run` で `dist/OpenClaw.app` がリストされていないことを確認する。
-   **dist-tags の npm auth web ループ**: レガシー認証を使用して OTP プロンプトを表示させる:
    -   `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest`
-   **`npx` 検証が `ECOMPROMISED: Lock compromised` で失敗する**: 新しいキャッシュで再試行する:
    -   `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version`
-   **遅延修正後にタグを再ポイントする必要がある**: タグを強制更新してプッシュし、GitHub リリースのアセットがまだ一致していることを確認する:
    -   `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7.  **GitHub リリース + アプキャスト**

-   [ ]  タグを付けてプッシュ: `git tag vX.Y.Z && git push origin vX.Y.Z` (または `git push --tags`)。
-   [ ]  `vX.Y.Z` の GitHub リリースを作成/更新し、**タイトルを `openclaw X.Y.Z`** とする（単なるタグ名ではない）。本文にはそのバージョンの**完全な**変更履歴セクション（ハイライト + 変更点 + 修正点）を含め、インラインで（リンクのみではなく）、**本文内でタイトルを繰り返してはならない**。
-   [ ]  成果物を添付: `npm pack` tarball (オプション), `OpenClaw-X.Y.Z.zip`, `OpenClaw-X.Y.Z.dSYM.zip` (生成された場合)。
-   [ ]  更新された `appcast.xml` をコミットしてプッシュする（Sparkle は main からフィードを取得する）。
-   [ ]  クリーンな一時ディレクトリ（`package.json` なし）から、`npx -y openclaw@X.Y.Z send --help` を実行して、インストール/CLI エントリーポイントが機能することを確認する。
-   [ ]  リリースノートを告知/共有する。

## プラグイン公開スコープ (npm)

`@openclaw/*` スコープの下で公開するのは、**既存の npm プラグイン**のみです。npm にないバンドル済みプラグインは、**ディスクツリーのみ**に留めます（`extensions/**` で出荷される）。リストを導出する手順:

1.  `npm search @openclaw --json` を実行し、パッケージ名を取得する。
2.  `extensions/*/package.json` の名前と比較する。
3.  **共通部分**（すでに npm にあるもの）のみを公開する。

現在の npm プラグインリスト（必要に応じて更新）:

-   @openclaw/bluebubbles
-   @openclaw/diagnostics-otel
-   @openclaw/discord
-   @openclaw/feishu
-   @openclaw/lobster
-   @openclaw/matrix
-   @openclaw/msteams
-   @openclaw/nextcloud-talk
-   @openclaw/nostr
-   @openclaw/voice-call
-   @openclaw/zalo
-   @openclaw/zalouser

リリースノートでは、**デフォルトではオンになっていない新しいオプションのバンドル済みプラグイン**（例: `tlon`）も明記する必要があります。

[クレジット](./credits.md)[テスト](./test.md)