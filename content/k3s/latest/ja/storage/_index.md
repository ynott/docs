---
title: "ボリュームとストレージ"
weight: 30
---

データを保持する必要があるアプリケーションを導入する場合は、永続ストレージを作成する必要があります。永続ストレージを使用すると、アプリケーションを実行するポッドの外部にアプリケーションデータを格納できます。このストレージの利用により、アプリケーションのポッドに障害が発生した場合でも、アプリケーション・データを維持します。

# ローカルストレージプロバイダ
K3sにはRancherのLocal Path Provisionerが付属していて、最初から各ノードのローカルストレージを使用して永続ボリュームクレームを作成することができるようになっています。以下に簡単な例を示します。詳細については、公式ドキュメントの [ここ](https://github.com/rancher/local-path-provisioner/blob/master/README.md#usage) を参照してください。

hostPathによってバックアップされた永続ボリュームクレームとそれを使用するポッドを作成する:

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

`kubectl create -f pvc.yaml` と `kubectl create -f pod.yaml` で yamlファイルを適用する

PV と PVC が作られたのを確認します。`kubectl get pv` と `kubectl get pvc` のコマンドでそれぞれ bound になっている必要があります。

# Longhorn

[comment]: <> (pending change - longhorn may support arm64 and armhf in the future.)

> **注:** 現時点では、Longhornはamd64のみをサポートしています。

K3sは[ロングホーン] (https://github.com/longhorn/longhorn)をサポートします。以下に簡単な例を示します。詳細については、公式ドキュメント[ここ](https://github.com/longhorn/longhorn/blob/master/README.md) を参照してください。

longhorn.yamlを適用してLonghornをインストールします。

```
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/master/deploy/longhorn.yaml
```

Longhornは `longhorn-system` という名前空間にインストールされます。

PVCを作成する前に、このyamlを使用してLonghorn用のストレージクラスを作成します。

```
kubectl create -f https://raw.githubusercontent.com/longhorn/longhorn/master/examples/storageclass.yaml
```

ここで、次のyamlを適用してPVCとポッドを `kubectl create-f pvc.yaml` と `kubectl create-f pod.yaml` で作成します。

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

PVとPVCが作成されていることを確認します。`kubectlget pv` と `kubectlget pvc` のステータスはそれぞれBoundになっている必要があります。
