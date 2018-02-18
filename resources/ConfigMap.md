
# ConfigMap

https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#configmap-v1-core

## 概要

- Key-Value?のデータを持たせてPod間でデータを共有できる
- Volumeか環境変数としてデータを渡す

## 重要そうなパラメータ

APIとサンプルみて

## テンプレ

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-example
data:
  example.property.1: hello
  example.property.2: area
  example.property.3: niki’s
  example.property.4: world
  example.property.5: create
```