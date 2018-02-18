
# ConfigMap

https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#secret-v1-core

## 概要

- Key-Value?のデータを持たせてPod間でデータを共有できる
- Kubernetes側で暗号化されて保存されてる？
- Volumeか環境変数としてデータを渡す(ほぼConfigMapと同じ)
- Manifestに書く場合はBase64でエンコードする必要あり

## 重要そうなパラメータ

APIとサンプルみて

## テンプレ

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  rootPassword: MWYyZDFlMmU2N2Rm
```