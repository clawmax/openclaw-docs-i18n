

  CLI コマンド

  
# update

OpenClaw を安全に更新し、安定版/ベータ版/開発版チャンネルを切り替えます。**npm/pnpm** 経由でインストールした場合（グローバルインストール、git メタデータなし）、更新は [Updating](../install/updating.md) で説明されているパッケージマネージャーのフローで行われます。

## 使用方法

```bash
openclaw update
openclaw update status
openclaw update wizard
openclaw update --channel beta
openclaw update --channel dev
openclaw update --tag beta
openclaw update --dry-run
openclaw update --no-restart
openclaw update --json
openclaw --update
```

## オプション

-   `--no-restart`: 更新成功後に Gateway サービスの再起動をスキップします。
-   `--channel <stable|beta|dev>`: 更新チャンネルを設定します（git + npm; 設定に保存されます）。
-   `--tag <dist-tag|version>`: この更新のみの npm ディストリビューションフラグまたはバージョンを上書きします。
-   `--dry-run`: 設定の書き込み、インストール、プラグインの同期、再起動を行わずに、計画された更新アクション（チャンネル/タグ/ターゲット/再起動フロー）をプレビューします。
-   `--json`: 機械可読な `UpdateRunResult` JSON を出力します。
-   `--timeout `: ステップごとのタイムアウト（デフォルトは 1200 秒）。

注: ダウングレードには確認が必要です。古いバージョンは設定を破損する可能性があるためです。

## update status

アクティブな更新チャンネルと git タグ/ブランチ/SHA（ソースチェックアウトの場合）に加え、更新の可用性を表示します。

```bash
openclaw update status
openclaw update status --json
openclaw update status --timeout 10
```

オプション:

-   `--json`: 機械可読なステータス JSON を出力します。
-   `--timeout `: チェックのタイムアウト（デフォルトは 3 秒）。

## update wizard

更新チャンネルを選択し、更新後に Gateway を再起動するかどうかを確認する対話型フローです（デフォルトは再起動します）。git チェックアウトがない状態で `dev` を選択すると、作成するオプションが表示されます。

## 動作内容

チャンネルを明示的に切り替える場合（`--channel ...`）、OpenClaw はインストール方法も合わせて維持します:

-   `dev` → git チェックアウトを確実に行い（デフォルト: `~/openclaw`、`OPENCLAW_GIT_DIR` で上書き可能）、それを更新し、そのチェックアウトからグローバル CLI をインストールします。
-   `stable`/`beta` → 一致するディストリビューションフラグを使用して npm からインストールします。

Gateway コアの自動更新機能（設定で有効化されている場合）は、この同じ更新パスを再利用します。

## Git チェックアウトフロー

チャンネル:

-   `stable`: 最新の非ベータタグをチェックアウトし、ビルド + doctor を実行します。
-   `beta`: 最新の `-beta` タグをチェックアウトし、ビルド + doctor を実行します。
-   `dev`: `main` ブランチをチェックアウトし、フェッチ + リベースを実行します。

概要:

1.  クリーンなワークツリーが必要です（未コミットの変更がない状態）。
2.  選択したチャンネル（タグまたはブランチ）に切り替えます。
3.  アップストリームをフェッチします（dev のみ）。
4.  dev のみ: 一時的なワークツリーで事前チェック（lint + TypeScript ビルド）を実行します。先端のコミットが失敗した場合、最新のクリーンビルドを見つけるために最大 10 コミット遡ります。
5.  選択したコミットにリベースします（dev のみ）。
6.  依存関係をインストールします（pnpm 推奨; npm フォールバック）。
7.  ビルド + Control UI をビルドします。
8.  最終的な「安全な更新」チェックとして `openclaw doctor` を実行します。
9.  プラグインをアクティブなチャンネルに同期します（dev はバンドルされた拡張機能を使用; stable/beta は npm を使用）し、npm インストールされたプラグインを更新します。

## \--update ショートハンド

`openclaw --update` は `openclaw update` に書き換えられます（シェルやランチャースクリプトで便利です）。

## 関連項目

-   `openclaw doctor`（git チェックアウトでは最初に更新を実行するオプションを提供）
-   [開発チャンネル](../install/development-channels.md)
-   [更新](../install/updating.md)
-   [CLI リファレンス](../cli.md)

[uninstall](./uninstall.md)[voicecall](./voicecall.md)