---
title: "詳細オプション"
weight: 40
aliases:
  - /k3s/latest/en/running/
---

このセクションでは、K3を実行および管理するさまざまな方法について詳しく説明します。

サーバーの起動
------------------

インストールスクリプトは、OSがsystemdまたはopenrcを使用しているかどうかを自動検出し、サービスを開始します。
ログはopenrcの場合は `/var/log/k3s.log` に出力され、systemdの場合は、`/var/log/syslog` に出力されます。`journalctl-u k3s`を使用して表示することもできます。インストールスクリプトを使用したインストールと自動起動の例:

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

エージェントが大量のログを作成するため、出力はかなり長くなります。デフォルトでは、サーバー自身をノードとして登録します(エージェントを実行します)。

Alpine Linux
------------

Alpine Linuxを利用するには事前に以下のステップを実行する必要があります:

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

それから以下を追加 **/etc/update-extlinux** を修正します:

```
default_kernel_opts="...  cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory"
```

次に設定を更新してリブートします:

```bash
update-extlinux
reboot
```

再起動後:

- **k3s** を **/usr/local/bin/k3s** へダウンロード
- **/etc/init.d** にopenrcファイルを作成

Dockerで動かす(docker-composeを利用)
-----------------

[k3d](https://github.com/rancher/k3d)はDockerで簡単にK3を実行できるように設計されたユーティリティです。このユーティリティは、MacOSの場合は[brew](https://brew.sh/)ユーティリティを使用してインストールできます。

`rancher/k3s` イメージはDockerからK3sサーバとエージェントを実行するためにも利用できます。`docker-compose.yml` はK3sリポジトリのルートにあるのでDockerからK3sを実行する方法の一例です。このレポジトリから `docker-compose` を使って起動するにはいかのようにします:

    docker-compose up --scale node=3
    # kubeconfig はカレントディレクトリに書かれます
    kubectl --kubeconfig kubeconfig.yaml get node

    NAME           STATUS   ROLES    AGE   VERSION
    497278a2d6a2   Ready    <none>   11s   v1.13.2-k3s2
    d54c8b17c055   Ready    <none>   11s   v1.13.2-k3s2
    db7a5a5a5bdd   Ready    <none>   12s   v1.13.2-k3s2

エージェントをDockerだけで実行するには、`docker-compose up node` を使用します。または、Docker runコマンドを使用することもできます:

    sudo docker run \
            -d --tmpfs /run \
            --tmpfs /var/run \
            -e K3S_URL=${SERVER_URL} \
            -e K3S_TOKEN=${NODE_TOKEN} \
            --privileged rancher/k3s:vX.Y.Z

