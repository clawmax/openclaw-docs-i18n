

  ホスティングとデプロイメント

  
# VPSホスティング

このハブは、サポートされているVPS/ホスティングガイドへのリンクを提供し、クラウドデプロイメントがどのように機能するかを高レベルで説明します。

## プロバイダーを選ぶ

-   **Railway** (ワンクリック + ブラウザセットアップ): [Railway](./install/railway.md)
-   **Northflank** (ワンクリック + ブラウザセットアップ): [Northflank](./install/northflank.md)
-   **Oracle Cloud (Always Free)**: [Oracle](./platforms/oracle.md) — $0/月 (Always Free、ARM; キャパシティ/サインアップが扱いにくい場合あり)
-   **Fly.io**: [Fly.io](./install/fly.md)
-   **Hetzner (Docker)**: [Hetzner](./install/hetzner.md)
-   **GCP (Compute Engine)**: [GCP](./install/gcp.md)
-   **exe.dev** (VM + HTTPSプロキシ): [exe.dev](./install/exe-dev.md)
-   **AWS (EC2/Lightsail/無料枠)**: こちらもうまく動作します。ビデオガイド: [https://x.com/techfrenAJ/status/2014934471095812547](https://x.com/techfrenAJ/status/2014934471095812547)

## クラウドセットアップの仕組み

-   **ゲートウェイはVPS上で動作**し、状態とワークスペースを所有します。
-   あなたはラップトップ/携帯電話から**コントロールUI**または**Tailscale/SSH**経由で接続します。
-   VPSを信頼できる情報源として扱い、状態とワークスペースを**バックアップ**してください。
-   安全なデフォルト設定: ゲートウェイをループバック上で維持し、SSHトンネルまたはTailscale Serve経由でアクセスします。`lan`/`tailnet`にバインドする場合は、`gateway.auth.token`または`gateway.auth.password`を要求してください。

リモートアクセス: [Gateway remote](./gateway/remote.md)  
プラットフォームハブ: [Platforms](./platforms.md)

## VPS上の共有会社エージェント

これは、ユーザーが1つの信頼境界内にいる場合（例えば1つの会社のチーム）、かつエージェントが業務専用である場合に有効なセットアップです。

-   専用のランタイム（VPS/VM/コンテナ + 専用OSユーザー/アカウント）上で維持してください。
-   そのランタイムに個人のApple/Googleアカウントや個人のブラウザ/パスワードマネージャープロファイルでサインインしないでください。
-   ユーザー同士が敵対的な関係にある場合は、ゲートウェイ/ホスト/OSユーザーごとに分割してください。

セキュリティモデルの詳細: [Security](./gateway/security.md)

## VPSでノードを使用する

ゲートウェイをクラウドに置いたまま、ローカルデバイス（Mac/iOS/Android/ヘッドレス）上で**ノード**をペアリングできます。ノードはローカルの画面/カメラ/キャンバスと`system.run`機能を提供し、ゲートウェイはクラウドに残ります。ドキュメント: [Nodes](./nodes.md), [Nodes CLI](./cli/nodes.md)

## 小型VMおよびARMホストの起動チューニング

低電力VM（またはARMホスト）でCLIコマンドの動作が遅いと感じる場合は、Nodeのモジュールコンパイルキャッシュを有効にしてください:

```
grep -q 'NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache' ~/.bashrc || cat >> ~/.bashrc <<'EOF'
export NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
mkdir -p /var/tmp/openclaw-compile-cache
export OPENCLAW_NO_RESPAWN=1
EOF
source ~/.bashrc
```

-   `NODE_COMPILE_CACHE`は、繰り返し実行されるコマンドの起動時間を改善します。
-   `OPENCLAW_NO_RESPAWN=1`は、自己再起動パスによる追加の起動オーバーヘッドを回避します。
-   最初のコマンド実行でキャッシュがウォームアップされ、その後の実行はより高速になります。
-   Raspberry Piの詳細については、[Raspberry Pi](./platforms/raspberry-pi.md)を参照してください。

### systemdチューニングチェックリスト (オプション)

`systemd`を使用するVMホストの場合、以下を検討してください:

-   安定した起動パスのためにサービス環境変数を追加:
    -   `OPENCLAW_NO_RESPAWN=1`
    -   `NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache`
-   再起動動作を明示的に設定:
    -   `Restart=always`
    -   `RestartSec=2`
    -   `TimeoutStartSec=90`
-   状態/キャッシュパスにはSSDバックアップディスクを優先し、ランダムI/Oによるコールドスタートのペナルティを軽減します。

例:

```bash
sudo systemctl edit openclaw
```

```ini
[Service]
Environment=OPENCLAW_NO_RESPAWN=1
Environment=NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
Restart=always
RestartSec=2
TimeoutStartSec=90
```

`Restart=`ポリシーが自動復旧にどのように役立つか: [systemd can automate service recovery](https://www.redhat.com/en/blog/systemd-automate-recovery).

[アンインストール](./install/uninstall.md)[Fly.io](./install/fly.md)