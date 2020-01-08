---
title: 既知の問題
weight: 70
---
「既知の問題」は定期的に更新され、次のリリースにあるすぐには対処できないかもしれない問題について情報共有するようようにしています。

**Snap Docker**

K3sをdockerとともに使用することを計画している場合、K3sの実行時に問題が発生することが知られているため、スナップパッケージを使用してDockerをインストールすることはお勧めしません。

**Iptables**

古いiptablesではなくnftablesモードでiptablesを実行している場合、問題が発生する可能性があります。問題を避けるために、新しいiptables(1.6.1+など)を使うことをお勧めします。

**RootlessKit**

K3sをRootlessKit上で動かす機能は試験的に実装されているものですので、いくつか[既知の問題]({{<baseurl>}}/k3s/latest/en/advanced/#known-issues-with-rootlesskit)があります。