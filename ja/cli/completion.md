

  CLI コマンド

  
# completion

シェル補完スクリプトを生成し、オプションでシェルプロファイルにインストールします。

## 使用方法

```bash
openclaw completion
openclaw completion --shell zsh
openclaw completion --install
openclaw completion --shell fish --install
openclaw completion --write-state
openclaw completion --shell bash --write-state
```

## オプション

-   `-s, --shell `: ターゲットシェル (`zsh`, `bash`, `powershell`, `fish`; デフォルト: `zsh`)
-   `-i, --install`: シェルプロファイルにソース行を追加して補完をインストール
-   `--write-state`: 補完スクリプトを標準出力に表示せずに `$OPENCLAW_STATE_DIR/completions` に書き込む
-   `-y, --yes`: インストール確認プロンプトをスキップ

## 注記

-   `--install` は、小さな「OpenClaw Completion」ブロックをシェルプロファイルに書き込み、キャッシュされたスクリプトを指すようにします。
-   `--install` または `--write-state` を指定しない場合、コマンドはスクリプトを標準出力に表示します。
-   補完生成はコマンドツリーを積極的に読み込むため、ネストされたサブコマンドも含まれます。

[clawbot](./clawbot.md)[config](./config.md)

---