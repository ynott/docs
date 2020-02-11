---
title: "ボリュームとストレージ"
weight: 30
---

データを保持するアプリケーションを導入する場合は、永続ストレージを作成する必要があります。永続ストレージを使用すると、アプリケーションを実行するPodの外部にアプリケーションのデータを格納できます。永続ストレージを使用してアプリケーションのPodに障害が発生した場合でも、アプリケーションのデータが維持されます。

パーシステントボリューム(PV)は、Kubernetesクラスターで使用するストレージの一つで、パーシステントボリュームクレームのリクエストによりストレージから切り出されるものです。PVとPVCがどのような挙動をするかについて詳しくは、公式のKubernetesの[ストレージ](https://kubernetes.io/docs/concepts/storage/volumes/)のドキュメントを参照してください。

こちらのページでは、ローカルストレージプロバイダーか、[Longhorn](#setting-up-longhorn)を使ってパーシステントストレージをセットアップする方法を解説します。

# ローカルストレージプロバイダー
K3sにはRancherのLocal Path Provisionerが付属していて、最初から各ノードのローカルストレージを使用して永続ボリュームクレームを作成することができるようになっています。以下に簡単な例を示します。詳細については、[公式ドキュメント](https://github.com/rancher/local-path-provisioner/blob/master/README.md#usage)を参照してください。

hostPathによってバックアップされた永続ボリュームクレームとそれを使用するPodを作成する:

### pvc.yaml

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-path-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 2Gi
```

### pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
  namespace: default
spec:
  containers:
  - name: volume-test
    image: nginx:stable-alpine
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: volv
      mountPath: /data
    ports:
    - containerPort: 80
  volumes:
  - name: volv
    persistentVolumeClaim:
      claimName: local-path-pvc
```

yamlファイルを適用する:

```
kubectl create -f pvc.yaml
kubectl create -f pod.yaml
```

PV と PVC が作られたのを確認します:

```
kubectl get pv
kubectl get pvc
```

それぞれステータスが、Boundになっているか確認してください。

# Longhornをセットアップする

> **注:** 現時点では、Longhornはamd64のみをサポートしています。

K3sは[Longhorn](https://github.com/longhorn/longhorn)をサポートします。Longhornは、Kubernetes向けのオープンソース分散ブロックストレージです。

以下に簡単な例を示します。詳細については、[公式ドキュメント](https://github.com/longhorn/longhorn/blob/master/README.md)を参照してください。

longhorn.yamlを適用してLonghornをインストールします。

```
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/master/deploy/longhorn.yaml
```

Longhornは `longhorn-system` という名前空間にインストールされます。

PVCを作成する前に、このyamlを使用してLonghorn用のストレージクラスを作成します。

```
kubectl create -f https://raw.githubusercontent.com/longhorn/longhorn/master/examples/storageclass.yaml
```

yamlを適用してPVCとPodを作成します。

```
kubectl create -f pvc.yaml
kubectl create -f pod.yaml
```

### pvc.yaml

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: longhorn-volv-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: longhorn
  resources:
    requests:
      storage: 2Gi
```

### pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
  namespace: default
spec:
  containers:
  - name: volume-test
    image: nginx:stable-alpine
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: volv
      mountPath: /data
    ports:
    - containerPort: 80
  volumes:
  - name: volv
    persistentVolumeClaim:
      claimName: longhorn-volv-pvc
```

PVとPVCが作成されていることを確認:

```
kubectl get pv
kubectl get pvc
```

それぞれステータスが、Boundになっているか確認してください。
