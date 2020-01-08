---
title: "アップグレード"
weight: 25
---

インストールスクリプトでK3sをアップグレードすることができます。もしくは、指定のバージョンのバイナリーを手動でインストールします。

>**注:** アップグレード時には、まずサーバノードを1つずつアップグレードしてから、ワーカノードをアップグレードします。

### インストールスクリプトでアップグレードする

K3sを古いバージョンからアップグレードするには、同じフラグを使用してインストールスクリプトを再実行します。次に例を示します:

```sh
curl -sfL https://get.k3s.io | sh -
```

特定のバージョンにアップグレードする場合は、次の様にコマンドを実行します:

```sh
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=vX.Y.Z-rc1 sh -
```

### バイナリーを使って手動でアップグレードする

または、K3sを手動でアップグレードします:

1. 必要なバージョンのK3sを[リリース](https://github.com/rancher/k3s/releases/latest)からダウンロードします。
2. 適切な場所にインストールします(通常は `/usr/local/bin/k3s`)
3. 古いバージョンを停止
4. 新しいバージョンを起動

### K3sの再起動

K3sの再起動は、systemdおよびopenrcに対するインストールスクリプトでサポートされています。
systemdの場合、手動で再起動するには、次のコマンドを使用します:
```sh
sudo systemctl restart k3s
```

openrcの場合、手動で再起動するには、次のコマンドを使用します:
To restart manually for openrc use:
```sh
sudo service k3s restart
```