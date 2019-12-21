---
title: "クイックスタートガイド"
weight: 10
---

>**注意:** このガイドでは、デフォルトオプションを使用してクラスタをすばやく起動する方法について説明します。[インストールセクション](../installation)では、K3の設定方法について詳しく説明しています。

> Kubernetesは初めてですか？Kubernetesの基本的なことを概説した素晴らしいチュートリアルが公式ドキュメントの[こちら](https://kubernetes.io/docs/tutorials/kubernetes-basics/)にあります。

インストールスクリプト
--------------
systemdまたはopenrcベースのシステムにサービスとしてインストールするのに便利なK3sをインストールするスクリプトがあります。このスクリプトは、https://get.k3s.io からダウンロードすることができます。この方法でK3sをインストールするには、以下のように動かします:
```bash
curl -sfL https://get.k3s.io | sh -
```

このインストールの実行後:

* K3sサービスは、ノードの再起動後、またはプロセスがクラッシュまたは強制終了した場合に自動的に再起動するように設定されます。
* `kubectl`、`crictl`、`ctr`、`k3s-killall`と`k3s-uninstall.sh` などの追加ユーティリティがインストールされます。
* kubeconfigファイルは`/etc/rancher/k3s/k3s.yaml`に書き込まれ、K3sがインストールしたkubectlで自動的に使用されます。

ワーカーノードにインストールしてクラスタに追加するには、環境変数 `K3S_URL` および `K3S_TOKEN` を使用してインストールスクリプトを実行します。Workerノードを追加する方法は以下の通りです:

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -
```
`K3S_URL`パラメータを設定すると、K3がワーカーモードで実行されます。K3sエージェントは、指定されたURLでリッスンするK3sサーバーに登録されます。`K3S_TOKEN`に使用する値は、サーバーノードの`/var/lib/rancher/k3s/server/node-token`に格納されます。

注意：各マシンには一意のホスト名が必須です。マシンのホスト名が一意でない場合は、`K3S_NODE_NAME`環境変数を渡し、各ノードに有効な一意のホスト名を指定します。
