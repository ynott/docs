---
title: "インストール"
weight: 20
---

この節では、さまざまな環境にK3をインストールする手順について説明します。K3のインストールを開始する前に、[ノード要件]({{< baseurl >}}/k3s/latest/en/installation/node-requirements/) を満たしていることを確認してください。

[インストールおよび構成オプション]({{< baseurl >}}/k3s/latest/en/installation/install-options/)では、K3のインストール時に使用できるオプションについて説明します。


[外部DBによる高可用性]({{< baseurl >}}/k3s/latest/en/installation/ha/)では、MySQL、PostgreSQL、etcdなどの外部データストアでバックアップされたHA K3sクラスタのセットアップ方法について説明します。

[組み込みDB(試験実装)による高可用性]({{< baseurl >}}/k3s/latest/ja/installation/ha-embedded/)では、組み込みの分散データベースを利用するHA K3クラスタのセットアップ方法について説明します。

[エアギャップインストール]({{< baseurl >}}/k3s/latest/ja/installation/airgap/)では、インターネットに直接アクセスできない環境でK3をセットアップする方法について説明します。

### アンインストール

`install.sh`スクリプトを使ってK3をインストールした場合、アンインストールスクリプトが作成されます。アンインストールスクリーンセーバーは、ノードの`/usr/local/bin/k3s-uninstall.sh`(エージェントの場合は、`k3s-agent-uninstall.sh`)に作成されます。
