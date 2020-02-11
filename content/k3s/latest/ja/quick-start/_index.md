---
title: "クイックスタートガイド"
weight: 10
---

この章では、デフォルトのオプションを使用してクラスタをすばやく起動する方法について説明します。K3sの設定方法については、[インストール](../installation)の章で詳しく説明しています。

K3sのコンポーネントがそれぞれどのように連携するかについては、[アーキテクチャ]({{<baseurl>}}/k3s/latest/en/architecture/#high-availability-with-an-external-db)の章を参照してください。

> Kubernetesについてよく知らない場合、Kubernetesの基本的なことを概説した素晴らしいチュートリアルが[公式ドキュメント](https://kubernetes.io/docs/tutorials/kubernetes-basics/)にあります。

インストールスクリプト
--------------
K3sには便利なインストールスクリプトがあり、それを使ってsystemdやopenrcベースのシステムにサービスとしてインストールできます。このスクリプトは、https://get.k3s.io からダウンロードできます。インストールスクリプトを使ってK3sをインストールする場合、以下のように実行します:
```bash
curl -sfL https://get.k3s.io | sh -
```

インストール後:

* K3sサービスが、ノードの再起動時、プロセスがクラッシュまたは強制終了した場合に自動的に再起動するようなります。
* `kubectl`、`crictl`、`ctr`、`k3s-killall`と`k3s-uninstall.sh` などの追加ユーティリティがインストールされます。
* [kubeconfig](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)ファイルは`/etc/rancher/k3s/k3s.yaml`に保存され、K3sがインストールしたkubectlで自動的に利用されます。

ワーカーノードをインストールして、クラスタに追加するには、`K3S_URL` および `K3S_TOKEN` の環境変数を設定してインストールスクリプトを実行します。ワーカーノードを追加する方法は以下の通りです:

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -
```
`K3S_URL`パラメータが設定されていると、K3sがワーカーモードで実行されます。ワーカーモードのK3sエージェントは、指定されたURLのK3sサーバーに登録されます。`K3S_TOKEN`の値は、サーバーノードの`/var/lib/rancher/k3s/server/node-token`に格納されています。

注意: 各マシンのホスト名は一意である必要があります。マシンのホスト名が一意でない場合は、各ノードで一意のホスト名として有効な値を`K3S_NODE_NAME`環境変数に指定します。
