
# Pod

https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#pod-v1-core

## 概要

- 一つ以上のコンテナの集合体
- 各コンテナはlocalhostで通信できる


## 重要そうなパラメータ

### spec.containers.lifecycle

Podの開始後と停止前にHTTP GETリクエストを送るかコマンドを実行することができる

### spec.containers.livenessProbe

コンテナの生存確認用

失敗時にはRestartを行う

### spec.containers.readinessProbe

コンテナが準備できているかの確認用

失敗時にはServiceのエンドポイントから外される

### spec.containers.volumeDevices

コンテナで使用するブロックデバイスを指定

persistentVolumeClaimを指定する

(1.9ではAlphaらしい)

### spec.containers.volumeMounts

コンテナ内部でのVolumeのマウント先の指定

### spec.restartPolicy

Pod内のコンテナが停止した際のリスタートのポリシーを指定

- Always(default)
- OnFaiure
- Never

がある

### spec.serviceAccountName

Podで使用するServiceAccountを指定

### spec.tolerations

nodeのtaintに対して許容するものを指定
taintの条件が真の場合はそのEffectを許容する。

### spec.volumes

Pod内のコンテナと共有するボリュームを指定

https://kubernetes.io/docs/concepts/storage/volumes/

## テンプレ

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-example
spec:
  containers:
  - name: ubuntu
    image: ubuntu:trusty
    command: ["echo"]
    args: ["Hello World"]
```