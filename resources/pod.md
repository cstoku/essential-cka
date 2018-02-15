
# Pod

## 概要

- 一つ以上のコンテナの集合体
- 各コンテナはlocalhostで通信できる


## 重要そうなパラメータ

- spec.containers.lifecycle
- spec.containers.livenessProbe
- spec.containers.readinessProbe
- spec.containers.volumeDevices
- spec.containers.volumeMounts
- spec.restartPolicy
- spec.serviceAccountName
- spec.tolerations
- spec.volumes

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