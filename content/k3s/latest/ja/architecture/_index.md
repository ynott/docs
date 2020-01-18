---
title: アーキテクチャ
weight: 1
---

このページでは、K3sサーバーの高可用性アーキテクチャとシングルノードサーバークラスターとの違いがどうなっているかを説明します。

また、エージェントノードがK3sサーバーにどのように登録されるかについても説明します。

サーバーノードは`k3s server`コマンドを動かしているマシン(ベアメタルか仮想)と定義します。ワーカーノードは、`k3s agent`コマンドを動かしているマシンと定義します。

このページでは、以下のトピックについて取り上げます:

- [Single-server setup with an embedded database](#single-server-setup-with-an-embedded-db)
- [High-availability K3s server with an external database](#high-availability-k3s-server-with-an-external-db)
  - [Fixed registration address for agent nodes](#fixed-registration-address-for-agent-nodes)
- [How agent node registration works](#how-agent-node-registration-works)

# 組み込みDBでのシングルサーバーのセットアップ

以下のダイアグラムは、組み込みSQLiteデータベースを使ったシングルノードのK3sサーバーのクラスターの例です。

この構成では、それぞれのエージェントノードが同じサーバーに登録されています。K3sの利用者は、K3sのサーバー上のAPIを呼び出すことによりKubernetesリソースを操作することができます。

![アーキテクチャ]({{<baseurl>}}/img/rancher/k3s-single-node-server-architecture.svg)

# 外部データベースを使った高可用構成のK3sサーバー

シングルサーバーのクラスターで様々なユースケースにマッチしますが、Kubernetesのコントロールプレーンの稼働時間がシビアな環境である場合、HAの高可用性環境でK3sを動かしたくなるでしょう。HAのK3sクラスターは次のもので構成されます:

* 2つ以上の**サーバーノード**で動いているKubernetes APIサーバーサービスと、その他のコントロールプレーンのサービス
* **外部のデータベース** (シングルサーバーでセットアップされている組み込みSQLiteデータベースではないもの)

![アーキテクチャ]({{< baseurl >}}/img/rancher/k3s-ha-architecture.svg)

### エージェントノードの登録固定アドレス

高可用性サーバー設定では、それぞれのノードが固定アドレスでKubernetes APIから登録される必要があります。以下のような構成図になります。

登録後、エージェントノードは、サーバーノードの一つに直接接続します。

![k3s HA]({{< baseurl >}}/img/k3s/k3s-production-setup.svg)

# エージェントノードが登録されるときの動作

エージェントノードは、`k3s agent`プロセスが開始したWebsocket接続によって登録され、エージェントプロセスの一部として動くクライアント側のロードバランサーにより接続が維持されます。

ノード用にランダムに生成されたパスワードは、`/etc/rancher/node/password`に保存され、それをノードクラスターシークレットとして使用してエージェントはサーバーに登録されます。サーバーにはノード毎の個別のパスワードが`/var/lib/rancher/k3s/server/cred/node-passwd`に保存されています。それ以降の試行では、同じパスワードを使用する必要があります。

エージェントの`/etc/rancher/node`ディレクトリが削除された場合、エージェントのパスワードファイルを再作成するか、サーバーからエントリを削除する必要があります。

`--with-node-id`フラグを使用してK3sサーバーまたはエージェントを起動することにより、ノード毎の一意のノードIDをホスト名に追加できます。
