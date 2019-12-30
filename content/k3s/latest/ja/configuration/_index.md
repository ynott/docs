---
title: "構成情報"
weight: 50
---

このセクションでは、さまざまな設定でK3sを使用する方法について説明します。


マニフェストの自動配布
------------------------

`/var/lib/rancher/k3s/server/manifests`にあるファイルは、`kubectl apply`を実行するように
自動的にKubernetesに展開されます。

Helmチャートをデプロイすることもできます。K3sは、チャートをインストールするためのCRDコントローラをサポートします。YAMLファイル仕様は次のようになります(以下の例は `/var/lib/rancher/k3s/server/manifests/traefik.yaml` から取りました):

```yaml
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: traefik
  namespace: kube-system
spec:
  chart: stable/traefik
  set:
    rbac.enabled: "true"
    ssl.enabled: "true"
```

K3sのデプロイコントローラーは、`kube-system` のネームスペースで新しいHelmChartリソースを監視するように設定されているため、HelmChartリソースのメタデータセクションの `namespace` は常に `kube-system` であることに注意してください。実際のヘルムリリースでネームスペースを指定するには、specセクションで`targetNamespace`キーを使用します:

```yaml
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: grafana
  namespace: kube-system
spec:
  chart: stable/grafana
  targetNamespace: monitoring
  set:
    adminPassword: "NotVerySafePassword"
  valuesContent: |-
    image:
      tag: master
    env:
      GF_EXPLORE_ENABLED: true
    adminUser: admin
    sidecar:
      datasources:
        enabled: true
```

specセクションでは`set`の他に`valuesContent`を使用できることにも注意してください。両方使っても問題ありません。

K3sバージョン`<=v 0.5.0`ではhelmchartsのAPIグループは、`k3s.cattle.io`を使用していました。これ以降のバージョンではこれを`helm.cattle.io`に変更しました。

helm CRDを利用する
---------------------

以下のサンプルのようにするとサード・パーティのhelmチャートを配布できます:

```yaml
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: nginx
  namespace: kube-system
spec:
  chart: nginx
  repo: https://charts.bitnami.com/bitnami
  targetNamespace: default
```

以下のサンプルのようにすると特定のバージョンのヘルムチャートをインストールできます:

```yaml
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: stable/nginx-ingress
  namespace: kube-system
spec:
  chart: nginx-ingress
  version: 1.24.4
  targetNamespace: default
```

外部からのクラスタへのアクセス
-----------------------------

`/etc/rancher/k3s/k3s.yaml` をコピーしてクラスターの外側にあるマシンに `~/.kube/config` に保存します。そのファイルの
"localhost"をK3sサーバのIPまたは名前を置換します。`kubectl` でK3sクラスタを管理できるようになります。

ノードの登録
-----------------

エージェントは、`/etc/rancher/node/password`に保存されているランダムに生成されたパスワードとして利用される
ノードクラスタシークレットを使ってサーバーに登録されます。サーバーには
`/var/lib/rancher/k3s/server/cred/node-passwd`に個々のノードのパスワードが保存され、
それ以降の操作では同じパスワードを使用する必要があります。エージェントの`/etc/rancher/node`ディレクトリを
削除した場合、エージェント用のパスワード・ファイルを再作成するか、サーバーからエントリーを除去する必要があります。
`--with-node-id`フラグを使用してk3sサーバーまたはエージェントを起動すると、ホスト名に一意のノードIDを追加できます。

ContainerdとDocker
----------

K3sにはcontainerdが含まれ、デフォルトでcontainerdに設定されます。containerdの代わりにDockerを使用したい場合は、単に`--docker`フラグを付けてエージェントを実行する必要があります。
K3sは`/var/lib/rancher/k3s/agent/etc/containerd/config.toml`にcontainerdのための設定を生成します。このファイルを高度にカスタマイズするには、同じディレクトリに`config.toml.tmpl`という別のファイルを作成すると代わりに使用されます。

`config.toml.tmpl`はGolangテンプレートファイルとなっていて、`config.Node`の構成がテンプレートになっています。以下は設定ファイルの構成をカスタマイズする方法の例です。 https://github.com/rancher/k3s/blob/master/pkg/agent/templates/templates.go#L16-L32

rootless (試験的実装)
--------

_**警告**:_ 試験的機能

初期のrootlessサポートが追加されましたが、その副作用によりユーザビリティーにいくつかの重要な課題があります。
私たちはrootlessに興味を持つ人たちのために初期サポートをリリースしました。いつか何人かの人がユーザビリティを改善
してくれることを期待しています。まず、ユーザーの名前空間が適切に設定され、サポートされていることを確認します。
手順については、RootlessKitの[要件セクション](https://github.com/rootless-containers/rootlesskit#setup)を
参照してください。簡単に言うと、Rootlessを動かすのに一番よい方法は最新のUbuntuを使うことです。

**Rootlessの課題**:

* **ポート**

    rootlessを実行すると、新しいネットワーク名前空間が作成されます。これは、K3sインスタンスがホストからまったく
    切り離されたネットワークで実行されていることを意味します。そうするとホストからK3sのサービスに接続する唯一の
    方法は、K3s ネットワークネームスペースにポートフォワードするしかありません。ホストに6443を自動的にバインドし、
    1024未満のポートを10000オフセットしてホストにバインドするコントローラがあります。

    つまり、ホストのサービスポート80は10080になりますが、8080はオフセットなしで8080になります。
 
    現在、自動的にバインドされるのは `LoadBalancer`サービスのみです。

* **Daemon ライフサイクル**

    K3sをkillしてからK3sの新しいインスタンスを起動すると、新しいネットワーク名前空間が作成されますが、古いポッドはkillされません。
    そうすると壊れた状態が残ります。これは現在のところ、ネットワーク名前空間をどのように扱うかという大きな問題です。

    この課題は https://github.com/rootless-containers/rootlesskit/issues/65 で追跡されています。

* **Cgroups**

    Cgroupsはサポートされていません。

**Rootlessで起動する**:

サーバかエージェントのどちらかに`--rootless`フラグを追加するだけです。`k3s server--rootless`を実行し、クラスターにアクセス
するためのkubeconfigがどこにあるかを調べるために `Wrote kubeconfig [パス]`というメッセージを探します。注意して欲しいのですが、
`-o｀を使ってkubeconfigを別のディレクトリに書き込んでいる場合、たぶんうまくいかないでしょう。
これは、内のK3sインスタンスが別のマウント名前空間で実行されているためです。

ノードラベルとtaint
----------------------

K3sエージェントは、kubeletにラベルとtaintを追加するオプション`--node-label`と`--node-taint`で設定できます。
この2つのオプションは、登録時にラベルまたはtaint(あるいはその両方) を追加するだけなので、K3sを実行してラベルを追加した後は変更
できません。ノードの登録後にノードのラベルや属性を変更したい場合は`kubectl`を使用してください。次の例は、ラベルとtaint
を追加する方法を示しています:
```
     --node-label foo=bar \
     --node-label hello=world \
     --node-taint key1=value1:NoExecute
```
