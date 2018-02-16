
# StatefulSet

https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#statefulset-v1-apps

## 概要

- 状態を持つ(ステートフル)ようなアプリを管理
- Deploymentに似ているが各々のPodに固有のIDを持つ
- 静的な永続ストレージを持つ

## 重要そうなパラメータ

### spec.serviceName

StatefulSetsのサービス名を指定
ネットワークIDをとして使用されるっぽい

https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#stable-network-id

### spec.volumeClaimTemplates

PersistentVolumeClaimのオブジェクトを指定。
これが上で指定した永続ストレージのことかと。

## テンプレ

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: my-storage-class
      resources:
        requests:
          storage: 1Gi
```