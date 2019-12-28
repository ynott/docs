---
title: FAQ
weight: 60
---

FAQは定期的に更新され、ユーザーがK3sについて最もよく尋ねられる質問に回答するようにしています。

**K3sはK8sの代替品として適していますか?**

K3sはk8sができることのほとんど全てが可能です。K8sより軽量なバージョンです。詳細については、[メイン]({{<baseurl>}}/k3s/latest/en/) のドキュメントページを参照してください。

**Traefikの代わりに自分のIngressを使うにはどうすればいいですか?**

単に `--no-deploy=traefik` でK3sサーバを起動し、ingressをデプロイして利用します。

**K3sはWindowsをサポートしていますか?**

現時点では、K3sはWindowsをネイティブにサポートしていませんが、私たちは将来的にはそのアイデアが実現されることを願っています。

**ソースからビルドする方法は?**

手順については、K3s[BUILDING.md](https://github.com/rancher/k3s/blob/master/BUILDING.md) を参照してください。
