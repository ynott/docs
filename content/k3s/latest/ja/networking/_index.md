---
title: "ネットワーク"
weight: 35
---

>**注:** CNIオプションの詳細については、[インストールネットワークオプション]({{< baseurl >}}/k3s/latest/ja/installation/network-options/)ページを参照してください。Flannelと様々なflannelバックエンドオプションの詳細や、独自のCNIの設定方法についても、そのページを参照してください。

オープンポート
----------
ポート情報については、[ノード要件]({{< baseurl >}}/k3s/latest/ja/installation/node-requirements/#networking)ページを参照してください。

CoreDNS
-------

CoreDNSはエージェントの起動時にデプロイされます。無効にする場合は、`--no-deploy coredns` オプションを付けて実行します。

CoreDNSをインストールしない場合は、クラスタにDNSプロバイダを自分でインストールする必要があります。

Traefik イングレスコントローラー
--------------------------

Traefikは、サーバーの起動時にデフォルトでデプロイされます。詳細については、[マニフェストの自動配布]({{< baseurl >}}/k3s/latest/ja/configuration/#auto-deploying-manifists)を参照してください。デフォルトの設定ファイルは `/var/lib/rancher/k3s/server/manifests/traefik.yaml` にあります。このファイルに加えられた変更は `kubectl apply` と同じように自動的にKubernetesに展開されます。

Traefikイングレスコントローラは、ホストのポート80,443および8080を使用します(つまり、これらはHostPortまたはNodePortでは使用できません)。

必要があれば、traefik.yamlファイルのオプションを設定することでtraefikを調整できます。
詳細については、公式の[TraefikのHelm設定パラメータ] (https://github.com/helm/charts/tree/master/stable/traefik#configuration)のreadmeを参照してください。

これを無効にするには、`--no-deploy traefik'オプションを指定して各サーバーを起動します。

サービスロードバランサー
---------------------

K3sには、空いているホストポートを使用する基本的なサービスロードバランサが含まれています。
たとえば、ポート80でリスニングするロード・バランサを作成しようとすると、クラスタ内でポート80が空いている
ホストを検索しようとします。使用可能なポートがない場合、ロード・バランサーは 保留中のままになります。

組み込みロードバランサーを無効にするには、`--no-deploy servicelb` オプションを指定してサーバーを起動します。MetalLBなどの別のロードバランサーを実行する場合に使用してください。
