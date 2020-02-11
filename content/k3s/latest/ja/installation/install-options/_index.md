---
title: "インストールオプション"
weight: 20
---

本項目では、K3sを最初にセットアップするときに使うオプションについてご紹介します:

- [インストールスクリプトのオプション](#インストールスクリプトのオプション)
- [K3sをバイナリーでインストールする方法](#K3sをバイナリーでインストールする方法)
- [K3sサーバー起動オプション](#K3sサーバー起動オプション)
- [K3sエージェント起動オプション](#K3sエージェント起動オプション)

より詳細なオプションについての内容は、[こちら]({{<baseurl>}}/k3s/latest/en/advanced)を参照してください。

# インストールスクリプトのオプション

[クイックスタートガイド]({{< baseurl >}}/k3s/latest/en/quick-start/)に書かれているように、https://get.k3s.ioからダウンロードしたインストールスクリプトを利用すれば、systemdやopenrcベースのシステムでK3sをサービスとしてインストールできます。

以下のようにシンプルに実行するだけです:
```sh
curl -sfL https://get.k3s.io | sh -
```

この方法でK3sをインストールする場合は、インストールオプションとして次の環境変数を指定できます:

- `INSTALL_K3S_SKIP_DOWNLOAD`

    trueに設定すると、K3sのハッシュまたはバイナリがダウンロードされません。

- `INSTALL_K3S_SYMLINK`

    この環境変数を`skip`に設定するとシンボリックリンクは作成されず、`force`では上書きされます。コマンドがパスに存在しない場合、通常はシンボリックリンクを作ります。

- `INSTALL_K3S_SKIP_START`

    trueに設定すると、K3sサービスは開始されません。

- `INSTALL_K3S_VERSION`

    githubからダウンロードするK3sのバージョン。指定されていない場合は、最新バージョンをダウンロードします。

- `INSTALL_K3S_BIN_DIR`

    K3sバイナリ、リンク、およびアンインストールスクリプトをインストールするディレクトリを指定します。指定しない場合、`/usr/local/bin`がデフォルトになります。

- `INSTALL_K3S_BIN_DIR_READ_ONLY`

    trueに設定すると、ファイルを`INSTALL_K3S_BIN_DIR`に書き込みません。強制的に`INSTALL_K3S_SKIP_DOWNLOAD=true`に設定されます。

- `INSTALL_K3S_SYSTEMD_DIR`

    systemdサービスおよび環境ファイルをインストールするディレクトリ。指定しない場合、通常は`/etc/systemd/system`です。

- `INSTALL_K3S_EXEC`

    K3sをサービスで起動する時に指定する起動コマンドとフラグです。コマンドが指定されていない場合、`K3S_URL`が設定されていれば"agent"として起動し、設定されていなければ"server"で動くようにデフォルトでセットされます。
    
    最終的にsystemdでのコマンドは、この環境変数とスクリプト引数の組み合わせが一緒になります。つまり、以下のコマンドは全てflannelを取り除いてサーバーに登録するという同じ動作になります:
     ```sh
     curl ... | INSTALL_K3S_EXEC="--no-flannel" sh -s -
     curl ... | INSTALL_K3S_EXEC="server --no-flannel" sh -s -
     curl ... | INSTALL_K3S_EXEC="server" sh -s - --no-flannel
     curl ... | sh -s - server --no-flannel
     curl ... | sh -s - --no-flannel
     ```

   - `INSTALL_K3S_NAME`

    systemdに登録される時のサービス名を指定します。指定しない場合は、K3s execコマンドがデフォルトで設定されます。指定すると、その名前の先頭に'k3s-'が付きます。

   - `INSTALL_K3S_TYPE`

    systemdに登録されるサービスのタイプ。指定しない場合は、K3s実行コマンドがデフォルトで設定されます。


`K3S_`で始まる環境変数は、systemdおよびopenrcサービスにより設定されます。`K3S_URL`をexecコマンドで明示的に設定していないとデフォルトで"agent"として設定されます。エージェントとして実行するときには、`K3S_TOKEN`も設定する必要があります。


### K3sをバイナリーでインストールする方法

前述のように、インストールスクリプトは基本的にK3sをサービスとして起動するように設定します。このスクリプトを使用しない場合は、[リリースページ](https://github.com/rancher/k3s/releases/latest)からバイナリのみダウンロードし、パスに配置して実行するだけでK3sを起動できます。K3sバイナリは、次のコマンドをサポートしています:

コマンド | 詳細
--------|------------------
<span class='nowrap'>`k3s server`</span> | K3s管理サーバーを起動します。このサーバーは、APIサーバー、コントローラマネージャー、スケジューラなどのKubernetesコントロールプレーンコンポーネントが起動されます。
<span class='nowrap'>`k3s agent`</span> |  K3sノードエージェントを起動します。これにより、K3sはワーカーノードとして動作し、`kubelet`と`kube-proxy`のKubernetesノードのサービスが起動されます。
<span class='nowrap'>`k3s kubectl`</span> | 組み込みの[kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) CLIを実行します。K3sサーバーノードとして起動するときに`KUBECONFIG` 環境変数が設定されていない場合は、`/etc/rancher/k3s/k3s.yaml`に作成された設定ファイルが自動的に参照されます。
<span class='nowrap'>`k3s crictl`</span> | 組み込みの[crictl](https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md)を実行します。これはKubernetesのコンテナランタイムインターフェース(CRI)とやりとりするためのCLIです。デバッグに便利です。
<span class='nowrap'>`k3s ctr`</span> | 組み込みの[ctr](https://github.com/projectatomic/containerd/blob/master/docs/cli.md)を実行します。これは、K3sが使用するコンテナデーモンcontainerdのCLIです。デバッグに便利です。
<span class='nowrap'>`k3s help`</span> | コマンドの一覧または1つのコマンドのヘルプを表示します

`k3s server`および`k3s agent`コマンドのその他の設定オプションは、<span class='nowrap'>`k3s server --help`</span>または<span class='nowrap'>`k3s agent --help`</span>で見られます。ヘルプテキストを以下に掲載します:

# K3sサーバー起動オプション

```
NAME:
   k3s server - Run management server

USAGE:
   k3s server [OPTIONS]

OPTIONS:
   -v value                                   (logging) Number for the log level verbosity (default: 0)
   --vmodule value                            (logging) Comma-separated list of pattern=N settings for file-filtered logging
   --log value, -l value                      (logging) Log to file
   --alsologtostderr                          (logging) Log to standard error as well as file (if set)
   --bind-address value                       (listener) k3s bind address (default: 0.0.0.0)
   --https-listen-port value                  (listener) HTTPS listen port (default: 6443)
   --advertise-address value                  (listener) IP address that apiserver uses to advertise to members of the cluster (default: node-external-ip/node-ip)
   --advertise-port value                     (listener) Port that apiserver uses to advertise to members of the cluster (default: listen-port) (default: 0)
   --tls-san value                            (listener) Add additional hostname or IP as a Subject Alternative Name in the TLS cert
   --data-dir value, -d value                 (data) Folder to hold state default /var/lib/rancher/k3s or ${HOME}/.rancher/k3s if not root
   --cluster-cidr value                       (networking) Network CIDR to use for pod IPs (default: "10.42.0.0/16")
   --service-cidr value                       (networking) Network CIDR to use for services IPs (default: "10.43.0.0/16")
   --cluster-dns value                        (networking) Cluster IP for coredns service. Should be in your service-cidr range (default: 10.43.0.10)
   --cluster-domain value                     (networking) Cluster Domain (default: "cluster.local")
   --flannel-backend value                    (networking) One of 'none', 'vxlan', 'ipsec', or 'flannel' (default: "vxlan")
   --token value, -t value                    (cluster) Shared secret used to join a server or agent to a cluster [$K3S_TOKEN]
   --token-file value                         (cluster) File containing the cluster-secret/token [$K3S_TOKEN_FILE]
   --write-kubeconfig value, -o value         (client) Write kubeconfig for admin client to this file [$K3S_KUBECONFIG_OUTPUT]
   --write-kubeconfig-mode value              (client) Write kubeconfig with this mode [$K3S_KUBECONFIG_MODE]
   --kube-apiserver-arg value                 (flags) Customized flag for kube-apiserver process
   --kube-scheduler-arg value                 (flags) Customized flag for kube-scheduler process
   --kube-controller-manager-arg value        (flags) Customized flag for kube-controller-manager process
   --kube-cloud-controller-manager-arg value  (flags) Customized flag for kube-cloud-controller-manager process
   --datastore-endpoint value                 (db) Specify etcd, Mysql, Postgres, or Sqlite (default) data source name [$K3S_DATASTORE_ENDPOINT]
   --datastore-cafile value                   (db) TLS Certificate Authority file used to secure datastore backend communication [$K3S_DATASTORE_CAFILE]
   --datastore-certfile value                 (db) TLS certification file used to secure datastore backend communication [$K3S_DATASTORE_CERTFILE]
   --datastore-keyfile value                  (db) TLS key file used to secure datastore backend communication [$K3S_DATASTORE_KEYFILE]
   --default-local-storage-path value         (storage) Default local storage path for local provisioner storage class
   --no-deploy value                          (components) Do not deploy packaged components (valid items: coredns, servicelb, traefik, local-storage, metrics-server)
   --disable-scheduler                        (components) Disable Kubernetes default scheduler
   --disable-cloud-controller                 (components) Disable k3s default cloud controller manager
   --disable-network-policy                   (components) Disable k3s default network policy controller
   --node-name value                          (agent/node) Node name [$K3S_NODE_NAME]
   --with-node-id                             (agent/node) Append id to node name
   --node-label value                         (agent/node) Registering kubelet with set of labels
   --node-taint value                         (agent/node) Registering kubelet with set of taints
   --docker                                   (agent/runtime) Use docker instead of containerd
   --container-runtime-endpoint value         (agent/runtime) Disable embedded containerd and use alternative CRI implementation
   --pause-image value                        (agent/runtime) Customized pause image for containerd sandbox
   --private-registry value                   (agent/runtime) Private registry configuration file (default: "/etc/rancher/k3s/registries.yaml")
   --node-ip value, -i value                  (agent/networking) IP address to advertise for node
   --node-external-ip value                   (agent/networking) External IP address to advertise for node
   --resolv-conf value                        (agent/networking) Kubelet resolv.conf file [$K3S_RESOLV_CONF]
   --flannel-iface value                      (agent/networking) Override default flannel interface
   --flannel-conf value                       (agent/networking) Override default flannel config file
   --kubelet-arg value                        (agent/flags) Customized flag for kubelet process
   --kube-proxy-arg value                     (agent/flags) Customized flag for kube-proxy process
   --rootless                                 (experimental) Run rootless
   --agent-token value                        (experimental/cluster) Shared secret used to join agents to the cluster, but not servers [$K3S_AGENT_TOKEN]
   --agent-token-file value                   (experimental/cluster) File containing the agent secret [$K3S_AGENT_TOKEN_FILE]
   --server value, -s value                   (experimental/cluster) Server to connect to, used to join a cluster [$K3S_URL]
   --cluster-init                             (experimental/cluster) Initialize new cluster master [$K3S_CLUSTER_INIT]
   --cluster-reset                            (experimental/cluster) Forget all peers and become a single cluster new cluster master [$K3S_CLUSTER_RESET]
   --no-flannel                               (deprecated) use --flannel-backend=none
   --cluster-secret value                     (deprecated) use --token [$K3S_CLUSTER_SECRET]
```

# K3sエージェント起動オプション

```
NAME:
   k3s agent - Run node agent

USAGE:
   k3s agent [OPTIONS]

OPTIONS:
   -v value                            (logging) Number for the log level verbosity (default: 0)
   --vmodule value                     (logging) Comma-separated list of pattern=N settings for file-filtered logging
   --log value, -l value               (logging) Log to file
   --alsologtostderr                   (logging) Log to standard error as well as file (if set)
   --token value, -t value             (cluster) Token to use for authentication [$K3S_TOKEN]
   --token-file value                  (cluster) Token file to use for authentication [$K3S_TOKEN_FILE]
   --server value, -s value            (cluster) Server to connect to [$K3S_URL]
   --data-dir value, -d value          (agent/data) Folder to hold state (default: "/var/lib/rancher/k3s")
   --node-name value                   (agent/node) Node name [$K3S_NODE_NAME]
   --with-node-id                      (agent/node) Append id to node name
   --node-label value                  (agent/node) Registering kubelet with set of labels
   --node-taint value                  (agent/node) Registering kubelet with set of taints
   --docker                            (agent/runtime) Use docker instead of containerd
   --container-runtime-endpoint value  (agent/runtime) Disable embedded containerd and use alternative CRI implementation
   --pause-image value                 (agent/runtime) Customized pause image for containerd sandbox
   --private-registry value            (agent/runtime) Private registry configuration file (default: "/etc/rancher/k3s/registries.yaml")
   --node-ip value, -i value           (agent/networking) IP address to advertise for node
   --node-external-ip value            (agent/networking) External IP address to advertise for node
   --resolv-conf value                 (agent/networking) Kubelet resolv.conf file [$K3S_RESOLV_CONF]
   --flannel-iface value               (agent/networking) Override default flannel interface
   --flannel-conf value                (agent/networking) Override default flannel config file
   --kubelet-arg value                 (agent/flags) Customized flag for kubelet process
   --kube-proxy-arg value              (agent/flags) Customized flag for kube-proxy process
   --rootless                          (experimental) Run rootless
   --no-flannel                        (deprecated) use --flannel-backend=none
   --cluster-secret value              (deprecated) use --token [$K3S_CLUSTER_SECRET]
```

### エージェント付与するノードラベルとtaint

K3sエージェントは、kubeletにラベルとtaintを追加するオプション`--node-label`と`--node-taint`があります。
この2つのオプションは、登録時にラベルまたはtaint(あるいはその両方) を追加するだけなので、K3sを実行してラベルを追加した後は変更
できません。

次の例は、ラベルとtaintを追加する方法です:
```
     --node-label foo=bar \
     --node-label hello=world \
     --node-taint key1=value1:NoExecute
```

ノードの登録後にノードのラベルやtaintを変更したい場合は`kubectl`を使う必要があります。[taints](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)を追加する方法や[ノードラベル](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/#add-a-label-to-a-node)の変更はKubernetesの公式ドキュメントを参照してください。
