title: "OpenClaw 開発チャネル Stable Beta Dev インストールガイド"
description: "npm dist-tag または git を使用して、OpenClaw の stable、beta、dev アップデートチャネルを切り替える方法を学びます。プラグインソースの管理と、タグ付けのベストプラクティスについて理解します。"
keywords: ["openclaw", "開発チャネル", "アップデートチャネル", "npm dist-tag", "ベータ版インストール", "git checkout", "安定版チャネル", "開発版チャネル"]
---

  上級者向け

  
# 開発チャネル

最終更新日: 2026-01-21 OpenClaw は3つのアップデートチャネルを提供しています:

-   **stable**: npm dist-tag `latest`。
-   **beta**: npm dist-tag `beta` (テスト中のビルド)。
-   **dev**: `main` ブランチの最新状態 (git)。npm dist-tag: `dev` (公開時)。

ビルドを **beta** に配信し、テストを行った後、バージョン番号を変更せずに **審査済みのビルドを `latest` に昇格** します — npm インストールの信頼できる情報源は dist-tag です。

## チャネルの切り替え

Git checkout:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

-   `stable`/`beta` は、最新の一致するタグをチェックアウトします (多くの場合、同じタグです)。
-   `dev` は `main` ブランチに切り替え、アップストリームにリベースします。

npm/pnpm グローバルインストール:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

これは対応する npm dist-tag (`latest`, `beta`, `dev`) を介して更新します。`--channel` で**明示的に**チャネルを切り替えると、OpenClaw はインストール方法も調整します:

-   `dev` は git チェックアウト (デフォルト `~/openclaw`、`OPENCLAW_GIT_DIR` で上書き可能) を確実に行い、それを更新し、そのチェックアウトからグローバル CLI をインストールします。
-   `stable`/`beta` は、一致する dist-tag を使用して npm からインストールします。

ヒント: stable と dev を並行して使用したい場合は、2つのクローンを保持し、ゲートウェイを stable のものに向けてください。

## プラグインとチャネル

`openclaw update` でチャネルを切り替えると、OpenClaw はプラグインソースも同期します:

-   `dev` は、git チェックアウトからのバンドル済みプラグインを優先します。
-   `stable` と `beta` は、npm インストールされたプラグインパッケージを復元します。

## タグ付けのベストプラクティス

-   git チェックアウトで使用したいリリースにタグを付けます (stable には `vYYYY.M.D`、beta には `vYYYY.M.D-beta.N`)。
-   `vYYYY.M.D.beta.N` も互換性のために認識されますが、`-beta.N` を推奨します。
-   従来の `vYYYY.M.D-` タグは、stable (非 beta) として引き続き認識されます。
-   タグは不変に保ちます: タグを移動または再利用しないでください。
-   npm dist-tag は、npm インストールの信頼できる情報源として残ります:
    -   `latest` → stable
    -   `beta` → 候補ビルド
    -   `dev` → main スナップショット (オプション)

## macOS アプリの提供状況

Beta および dev ビルドには、**macOS アプリリリースが含まれていない場合があります**。問題ありません:

-   git タグと npm dist-tag は引き続き公開できます。
-   リリースノートや変更履歴で「このベータ版には macOS ビルドはありません」と明記してください。

[Northflank にデプロイする](./northflank.md)

---