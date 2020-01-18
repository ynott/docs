---
title: "構成設定と詳細オプション"
weight: 40
aliases:
  - /k3s/latest/en/running/
  - /k3s/latest/en/configuration/
---

このセクションでは、K3sを実行および管理するさまざまな方法について詳しく説明します。

- [マニフェストの自動配布](#マニフェストの自動配布)
- [Helm CRDを利用する](#Helm-CRDを利用する)
- [kubectlを使って外部からクラスタへアクセスする](#kubectlを使って外部からクラスタへアクセスする)
- [コンテナランタイムにDockerを使用する](#コンテナランタイムにDockerを使用する)
- [RootlessKitでK3sを動かす(試験的実装)](#RootlessKitでK3sを動かす(試験的実装))
- [ノードのラベルとtaint](#ノードのラベルとtaint)
- [インストールスクリプトでのサーバーの起動](#インストールスクリプトでのサーバーの起動)
- [Alpine Linuxで設定するときの追加セットアップ](#Alpine-Linuxで設定するときの追加セットアップ)
- [K3d(Dockerで動くK3s)をdocker-composeで動かす](#K3d(Dockerで動くK3s)をdocker-composeで動かす)

# マニフェストの自動配布

`/var/lib/rancher/k3s/server/manifests`に置かれたファイルは、全て`kubectl apply`を適用する習慣のように
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

注意していただきたいことは、HelmChartのリソースメタデータセクションの`namespace`は常に`kube-system`にする必要があることです。というのもK3sデプロイコントローラーは新しくデプロイされたHelmChartのリソースでこの名前空間名を常に監視するように設定されているためです。実際のヘルムリリースでネームスペースを指定するには、以下の設定サンプルのように`spec`セクションディレクティブで、`targetNamespace`キーを使用します。

もう一つの注意点としては、`set`の他に、`spec`ディレクティブの配下で`values Content`を使用することもできます。また、両方を使っても問題ありません:

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

K3sバージョン`<= v0.5.0`ではHelmChartsのAPIグループは、`k3s.cattle.io`を使用していました。以降のバージョンは`helm.cattle.io`に変更されました。

# Helm CRDを利用する

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

# kubectlを使って外部からクラスタへアクセスする

`/etc/rancher/k3s/k3s.yaml` をコピーしてクラスターの外側にあるマシンに `~/.kube/config` に保存します。そのファイルの"localhost"をK3sサーバのIPまたは名前を置換します。`kubectl` でK3sクラスタを管理できるようになります。

# コンテナランタイムにDockerを使用する

K3sには業界標準の[containerd,](https://containerd.io/)が含まれ、デフォルトに設定されています。containerdの代わりにDockerを使用したい場合は、単に`--docker`フラグを付けてエージェントを実行するだけです。

K3sは`/var/lib/rancher/k3s/agent/etc/containerd/config.toml`にcontainerdのためのconfig.toml設定を生成します。このファイルを高度にカスタマイズするには、同じディレクトリに`config.toml.tmpl`という別のファイルを作成すると代わりこちらが使用されます。

`config.toml.tmpl`はGolangテンプレートファイルとなっていて、`config.Node`の構成がテンプレートになっています。以下は設定ファイルの構成をカスタマイズする方法の例です。 https://github.com/rancher/k3s/blob/master/pkg/agent/templates/templates.go#L16-L32

# RootlessKitでK3sを動かす(試験的実装)

_**警告**:_ 試験的機能

RootlessKitはLinuxネイティブの "root偽装" ユーティリティの一種で、主に[特権を持たないユーザとしてDockerとKubernetesを実行](https://github.com/rootless-containers/usernetes)ために作られていて、潜在的なコンテナ侵入攻撃からホスト上の実際のrootを保護します。

初期のrootlessサポートが追加されましたが、その副作用によりユーザビリティーにいくつかの重要な課題があります。

私たちはrootlessに興味を持つ人たちのために初期サポートをリリースしました。いつか何人かの人がユーザビリティを改善してくれることを期待しています。まず、ユーザーの名前空間が適切に設定され、サポートされていることを確認します。手順については、RootlessKitの[要件セクション](https://github.com/rootless-containers/rootlesskit#setup)を参照してください。
簡単に言うと、Rootlessを動かすのに一番よい方法は最新のUbuntuを使うことです。

### RootlessKitの課題

* **ポート**

    rootlessを実行すると、新しいネットワーク名前空間が作成されます。これは、K3sインスタンスがホストからまったく切り離されたネットワークで実行されていることを意味します。そうするとホストからK3sのサービスに接続する唯一の方法は、K3s ネットワークネームスペースにポートフォワードするしかありません。ホストに6443を自動的にバインドし、1024未満のポートを10000オフセットしてホストにバインドするコントローラがあります。

    つまり、ホストのサービスポート80は10080になりますが、8080はオフセットなしで8080になります。
 
    現在、自動的にバインドされるのは `LoadBalancer`サービスのみです。

* **Daemon ライフサイクル**

    K3sをkillしてからK3sの新しいインスタンスを起動すると、新しいネットワーク名前空間が作成されますが、古いポッドはkillされません。
    そうすると壊れた状態が残ります。これは現在のところ、ネットワーク名前空間をどのように扱うかという大きな問題です。

    この課題は https://github.com/rootless-containers/rootlesskit/issues/65 で検討されています。

* **Cgroups**

    Cgroupsはサポートされていません。

### Rootlessでサーバーとエージェントを起動する

サーバかエージェントのどちらかに`--rootless`フラグを追加するだけです。`k3s server --rootless`を実行し、クラスターにアクセス
するためのkubeconfigがどこにあるかを調べるために `Wrote kubeconfig [パス]`というメッセージを探します。

注意して欲しいのですが、`-o` を使って
kubeconfigを別のディレクトリに書き込んでいる場合、うまくいかない可能性があります。これは、内のK3sインスタンスが別のマウント名前空間で
実行されているためです。

# ノードのラベルとtaint

K3sエージェントは、kubeletにラベルとtaintを追加するオプション`--node-label`と`--node-taint`で設定できます。この2つのオプションは、[登録時]({{<baseurl>}}/k3s/latest/en/installation/install-options/#node-labels-and-taints-for-agents)にラベルまたはtaint(あるいはその両方) を追加するだけなので、K3sをラベルを追加して一度実行して後は変更できません。

ノードの登録後にノードのラベルや属性を変更したい場合は`kubectl`を使用してください。ラベルと[taint](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)と[ノードラベル](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/#add-a-label-to-a-node)を追加する方法は、それぞれ公式ドキュメントを参照してください。

# インストールスクリプトでのサーバーの起動

インストールスクリプトは、OSがsystemdまたはopenrcを使用しているかどうかを自動的に検出し、サービスを起動します。
ログはopenrcの場合は `/var/log/k3s.log` に出力されます。

systemdの場合は、`/var/log/syslog` に出力されます。`journalctl-u k3s`を使用して表示することもできます。

インストールスクリプトを使用したインストールと自動起動の例:

```bash
curl -sfL https://get.k3s.io | sh -
```

サーバーを手動で実行すると、次のような出力が表示されます:

```
$ k3s server
INFO[2019-01-22T15:16:19.908493986-07:00] Starting k3s dev                             
INFO[2019-01-22T15:16:19.908934479-07:00] Running kube-apiserver --allow-privileged=true --authorization-mode Node,RBAC --service-account-signing-key-file /var/lib/rancher/k3s/server/tls/service.key --service-cluster-ip-range 10.43.0.0/16 --advertise-port 6445 --advertise-address 127.0.0.1 --insecure-port 0 --secure-port 6444 --bind-address 127.0.0.1 --tls-cert-file /var/lib/rancher/k3s/server/tls/localhost.crt --tls-private-key-file /var/lib/rancher/k3s/server/tls/localhost.key --service-account-key-file /var/lib/rancher/k3s/server/tls/service.key --service-account-issuer k3s --api-audiences unknown --basic-auth-file /var/lib/rancher/k3s/server/cred/passwd --kubelet-client-certificate /var/lib/rancher/k3s/server/tls/token-node.crt --kubelet-client-key /var/lib/rancher/k3s/server/tls/token-node.key 
Flag --insecure-port has been deprecated, This flag will be removed in a future version.
INFO[2019-01-22T15:16:20.196766005-07:00] Running kube-scheduler --kubeconfig /var/lib/rancher/k3s/server/cred/kubeconfig-system.yaml --port 0 --secure-port 0 --leader-elect=false 
INFO[2019-01-22T15:16:20.196880841-07:00] Running kube-controller-manager --kubeconfig /var/lib/rancher/k3s/server/cred/kubeconfig-system.yaml --service-account-private-key-file /var/lib/rancher/k3s/server/tls/service.key --allocate-node-cidrs --cluster-cidr 10.42.0.0/16 --root-ca-file /var/lib/rancher/k3s/server/tls/token-ca.crt --port 0 --secure-port 0 --leader-elect=false 
Flag --port has been deprecated, see --secure-port instead.
INFO[2019-01-22T15:16:20.273441984-07:00] Listening on :6443                           
INFO[2019-01-22T15:16:20.278383446-07:00] Writing manifest: /var/lib/rancher/k3s/server/manifests/coredns.yaml 
INFO[2019-01-22T15:16:20.474454524-07:00] Node token is available at /var/lib/rancher/k3s/server/node-token 
INFO[2019-01-22T15:16:20.474471391-07:00] To join node to cluster: k3s agent -s https://10.20.0.3:6443 -t ${NODE_TOKEN} 
INFO[2019-01-22T15:16:20.541027133-07:00] Wrote kubeconfig /etc/rancher/k3s/k3s.yaml
INFO[2019-01-22T15:16:20.541049100-07:00] Run: k3s kubectl                             
```

エージェントが大量のログを作成するため、出力はかなり長くなります。デフォルトでは、
サーバー自身をノードとして登録します(エージェントを実行します)。

# Alpine Linuxで設定するときの追加セットアップ

Alpine Linuxでセットアップするには事前に以下のステップを実行する必要があります:

```bash
echo "cgroup /sys/fs/cgroup cgroup defaults 0 0" >> /etc/fstab

cat >> /etc/cgconfig.conf <<EOF
mount {
cpuacct = /cgroup/cpuacct;
memory = /cgroup/memory;
devices = /cgroup/devices;
freezer = /cgroup/freezer;
net_cls = /cgroup/net_cls;
blkio = /cgroup/blkio;
cpuset = /cgroup/cpuset;
cpu = /cgroup/cpu;
}
EOF
```

それから **/etc/update-extlinux** を修正して以下を追加します:

```
default_kernel_opts="...  cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory"
```

次に設定を更新したらリブートします:

```bash
update-extlinux
reboot
```

再起動後:

- **k3s** を **/usr/local/bin/k3s** へダウンロード
- **/etc/init.d** にopenrcファイルを作成

# K3d(Dockerで動くK3s)をdocker-composeで動かす

[k3d](https://github.com/rancher/k3d)はDockerでK3sを簡単に実行できるように設計されたユーティリティです。

このユーティリティは、MacOSの場合は[brew](https://brew.sh/)ユーティリティを使用してインストールできます。

```
brew install k3d
```

`rancher/k3s`イメージはDockerでK3sサーバとエージェントを実行するためにも利用できます。

K3sリポジトリのルートにある`docker-compose.yml`を使ってDockerでK3sを実行する例です。このレポジトリの`docker-compose`から起動するには以下のようにします:

    docker-compose up --scale node=3
    # kubeconfig はカレントディレクトリに書かれます

    kubectl --kubeconfig kubeconfig.yaml get node

    NAME           STATUS   ROLES    AGE   VERSION
    497278a2d6a2   Ready    <none>   11s   v1.13.2-k3s2
    d54c8b17c055   Ready    <none>   11s   v1.13.2-k3s2
    db7a5a5a5bdd   Ready    <none>   12s   v1.13.2-k3s2

Dockerでエージェントのみ実行するには、`docker-compose up node`とします。

もしくは、`docker run`コマンドで起動することもできます:

    sudo docker run \
            -d --tmpfs /run \
            --tmpfs /var/run \
            -e K3S_URL=${SERVER_URL} \
            -e K3S_TOKEN=${NODE_TOKEN} \
            --privileged rancher/k3s:vX.Y.Z

