---
title: "ネットワーク"
weight: 35
---

>**注:** CNIオプションの詳細については、[ネットワークインストールオプション]({{< baseurl >}}/k3s/latest/ja/installation/network-options/)の章を参照してください。Flannelと様々なflannelバックエンドオプションの詳細や、独自のCNIの設定方法についても、その章を参照してください。

オープンポート
----------
ポート情報については、[ノード要件]({{< baseurl >}}/k3s/latest/ja/installation/node-requirements/#networking)の章を参照してください。

CoreDNS
-------

CoreDNSはエージェントの起動時にデプロイされます。無効にする場合は、`--no-deploy coredns` オプションをつけて実行します。

CoreDNSをインストールしない場合は、クラスターにDNSプロバイダーを自分でインストールする必要があります。

Traefik イングレスコントローラー
--------------------------

[Traefik](https://traefik.io/)は簡単にマイクロサービスをデプロイすることができるロードバランサーでモダンなHTTPリバースプロキシーです。これによりアプリケーションの設計、導入、および実行時のネットワークの複雑さが軽減されます。

Traefikは、サーバーの起動時にデフォルトでデプロイされます。詳細については、[マニフェストの自動配布]({{< baseurl >}}/k3s/latest/ja/configuration/#auto-deploying-manifists)を参照してください。デフォルトの設定ファイルは `/var/lib/rancher/k3s/server/manifests/traefik.yaml` にあります。このファイルに加えられた変更は `kubectl apply` と同じように自動的にKubernetesに展開されます。

Traefikイングレスコントローラーは、ホストのポート80,443および8080を使用します(つまり、このポートはHostPortまたはNodePortでは使用できません)。

用途に応じて、traefik.yamlファイルにオプションを設定することでtraefikを調整できます。詳細については、公式の[TraefikのHelm設定パラメータ](https://github.com/helm/charts/tree/master/stable/traefik#configuration)のreadmeを参照してください。

これを無効にするには、`--no-deploy traefik`オプションを指定して各サーバーを起動します。

サービスロードバランサー
---------------------

K3sには、利用可能なhostPortを使用する基本的なサービスロードバランサーが組み込まれています。例えば、80ポートでリッスンするロードバランサーを作成しようとすると、クラスター内で80ポートが空いているホストを見つけようとします。使用可能なポートがない場合、ロードバランサーは保留中になります。

組み込みロードバランサーを無効にするには、`--no-deploy servicelb`オプションを指定してサーバーを起動します。MetalLBなどの別のロードバランサーを実行する場合に使用してください。
