---
title: "ネットワークオプション"
weight: 25
---

> **注意:** CoreDNS、Traefik、およびサービスLBについては、[ネットワーク]({{< baseurl >}}/k3s/latest/en/networking)ページを参照してください。

デフォルトでは、K3はCNIとしてflannelを使用し、デフォルトのバックエンドとしてVXLANを使用します。CNIを変更するには、[カスタムCNI](#custom-cni)の設定に関するセクションを参照してください。flannelバックエンドを変更するには、flannelオプションのセクションを参照してください。

### Flannel オプション

flannelのデフォルトのバックエンドはVXLANです。暗号化を有効にするには、次のIPSec(インターネットプロトコルセキュリティ)を通すか、または以下のWireGuardオプションを使用します。

FlannelのネルバックエンドとしてWireGuardを使いたいなら、追加のカーネルモジュールが必要かもしれません。詳しくは[WireGuardインストールガイド] (https://www.wireguard.com/install/)をご覧ください。WireGuardのインストール手順では、ご使用のオペレーティング・システムに適切なカーネル・モジュールがインストールされていることを確認します。WireGuard flannelバックエンドオプションを利用する前に、サーバとエージェントの両方のすべてのノードにWireGuardをインストールする必要があります。

  CLIフラグと値 | 詳細
  -------------------|------------
 <span style="white-space: nowrap">`--flannel-backend=vxlan`</span> | (デフォルト) VXLANをバックエンドとして使用 |
 <span style="white-space: nowrap">`--flannel-backend=ipsec`</span> | ネットワークトラフィックを暗号化するIPSECバックエンドを使用 |
 <span style="white-space: nowrap">`--flannel-backend=wireguard`</span> | ネットワークトラフィックを暗号化するWireGuardバックエンドを使います。追加のカーネルモジュールと設定が必要になる場合があります。 |

### カスタム CNI

`--flannel-backend=none` を指定してK3sを実行し、選択したCNIをインストールします。CanalとCalicoではIP Forwardingを有効にする必要があります。以下の手順を参照してください。

{{% tabs %}}
{{% tab "Canal" %}}

[プロジェクトCalicoドキュメント](https://docs.projectcalico.org/)にアクセスしてください。Canalをインストールするには以下の手順に従います。Canal YAMLを変更してcontainer_settingsセクションでIP転送が許可されるようにします。次に例を示します:

```
"container_settings": {
              "allow_ip_forwarding": true
          }
```

Canal YAMLを適用

ホストで次のコマンドを実行して、設定が適用されていることを確認します:

```
cat /etc/cni/net.d/10-calico.conflist
```

IP forwardingがtrueに設定されていることを確認します。

{{% /tab %}}
{{% tab "Calico" %}}

[Calico CNIプラグインガイド](https://docs.projectcalico.org/master/reference/cni-plugin/configuration)に従ってください。container_settingsセクションでIP転送が許可されるようにCalico YAMLを変更します。次に例を示します。

```
"container_settings": {
              "allow_ip_forwarding": true
          }
```

Canal YAMLを適用

ホストで次のコマンドを実行して、設定が適用されていることを確認します:

```
cat /etc/cni/net.d/10-canal.conflist
```

IP forwardingがtrueに設定されていることを確認します。


{{% /tab %}}
{{% /tabs %}}

