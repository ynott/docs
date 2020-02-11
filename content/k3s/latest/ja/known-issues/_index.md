---
title: "既知の問題"
weight: 70
---
「既知の問題」は定期的に更新され、次のリリースではすぐに対処されないと考えられる課題について情報共有しています。

**Snap Docker**

SnapでインストーラされたdockerでK3sを使用する予定がある場合、K3sの実行時に問題が発生すること分かっているため、Snapパッケージを使用してDockerをインストールすることはお勧めしません。

**Iptables**

古いモードではなく、nftablesモードでiptablesを実行している場合、問題が発生する可能性があります。問題を回避するために、新しいiptables(1.6.1以降など)を使うことをお勧めします。

**RootlessKit**

K3sをRootlessKit上で動かす機能は、試験的に実装されているものですので、いくつか[既知の問題]({{<baseurl>}}/k3s/latest/en/advanced/#known-issues-with-rootlesskit)があります。
