---
title: "インストール"
weight: 20
---

この節では、さまざまな環境にK3をインストールする手順について説明します。K3のインストールを開始する前に、[ノード要件]({{< baseurl >}}/k3s/latest/en/installation/node-requirements/)を満たしていることを確認してください。

[インストールオプション]({{< baseurl >}}/k3s/latest/en/installation/install-options/)では、K3のインストール時に使用できるオプションについて説明します。


[外部DBによる高可用性構成]({{< baseurl >}}/k3s/latest/en/installation/ha/)では、MySQL、PostgreSQL、etcdなどの外部データストアを利用したHA K3sクラスタのセットアップ方法について説明します。

[組み込みDB(試験実装)による高可用性]({{< baseurl >}}/k3s/latest/ja/installation/ha-embedded/)では、組み込みの分散データベースを利用するHA K3sクラスタのセットアップ方法について説明します。

[インターネット隔離環境でのインストール]({{< baseurl >}}/k3s/latest/ja/installation/airgap/)では、インターネットに直接アクセスできない環境でK3sをセットアップする方法について説明します。

### アンインストール

`install.sh`スクリプトを使ってK3sをインストールした場合、インストール時にアンインストールスクリプトが同時に作成されます。アンインストールスクリプトは、ノードの`/usr/local/bin/k3s-uninstall.sh`(エージェントの場合は、`k3s-agent-uninstall.sh`)に作成されます。
