---
title: "インターネット隔離環境でのインストール"
weight: 60
---

このガイドではインターネット隔離環境にノードが作成されていて、インターネット踏み台サーバ上にセキュアなDockerプライベートレジストリが用意されていることを前提としています。

# インストールの概要

1. [イメージディレクトリの準備](#イメージディレクトリの準備)
2. [レジストリYAMLの作成](#レジストリYAMLの作成)
3. [K3のインストール](#K3sのインストール)

### イメージディレクトリの準備
実行したいバージョンのK3sの[リリース](https://github.com/rancher/k3s/releases)ページから、利用するアーキテクチャのイメージtarファイルを入手します。

K3sを起動する前に、各ノードでtarファイルを`images`ディレクトリに配置します。次の例のようにします:

```sh
sudo mkdir -p /var/lib/rancher/k3s/agent/images/
sudo cp ./k3s-airgap-images-$ARCH.tar /var/lib/rancher/k3s/agent/images/
```

### レジストリYAMLの作成
registries.yamlファイルを`/etc/rancher/k3s/registries.yaml`に作成します。ここにk3sがプライベートレジストリに接続するための必要な詳細情報を設定します。
接続する前に必要な情報のregistries.yamlファイルは次のようになります:

```
---
mirrors:
  customreg:
    endpoint:
      - "https://ip-to-server:5000"
configs:
  customreg:
    auth:
      username: xxxxxx # this is the registry username
      password: xxxxxx # this is the registry password
    tls:
      cert_file: <path to the cert file used in the registry>
      key_file:  <path to the key file used in the registry>
      ca_file: <path to the ca file used in the registry>
```

注意：現時点では、セキュアレジストリ(独自CAのSSL)のみK3sでサポートします。

### K3sのインストール

[リリース](https://github.com/rancher/k3s/releases)ページからK3sバイナリを取得します。これは、インターネット隔離環境のイメージtarと同じバージョンです。
同様に、https://get.k3s.io からK3sインストールスクリプトをダウンロードします。

バイナリーをそれぞれのノードの`/usr/local/bin`に配置します。
インストールスクリプトをそれぞれのノードに配置して、ファイル名を`install.sh`にします。

各サーバにK3をインストールします:

```
INSTALL_K3S_SKIP_DOWNLOAD=true ./install.sh
```

それぞれエージェントをインストールします:

```
INSTALL_K3S_SKIP_DOWNLOAD=true K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken ./install.sh
```

注意点は、`myserver`をサーバーのIPまたは有効なDNSに置き換えること、`mynodetoken`をサーバーからのノードトークンに置き換えることです。
サーバーのノードトークンは、`/var/lib/rancher/k3s/server/node-token` に保存されています。

>**注意:** K3sにはkubeletsの為に`--resolv-conf`フラグも用意しています。これを使うとインターネット隔離環境でDNSを設定するのに便利です。

# アップグレード

エアギャップ環境の場合、以下の方法でアップグレードします:

1. [リリース](https://github.com/rancher/k3s/releases)ページから、アップグレードしたいバージョンのK3sのインターネット隔離環境用イメージ(tarファイル)をダウンロードします。全てのノードの`/var/lib/rancher/k3s/agent/images/`に保存して、古いtarファイルは削除します。
2. 全てのノードの`/usr/local/bin`にある古いK3sバイナリーをコピーして置き換えて、https://get.k3s.ioから(できるだけ新しい最新の)をダウンロードしてインストールスクリプトを上書きします。そうして、旧環境と同じ環境変数をインストール時に指定した状態で、スクリプトを再度実行してください。
3. K3sサービスを再起動(インストーラーで再起動しなかった場合)してください。
