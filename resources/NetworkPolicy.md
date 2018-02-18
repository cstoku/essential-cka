
# NetworkPolicy

https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#networkpolicy-v1-networking

## 概要

- LabelSelectorでマッチしたPodのネットワーク制御
- ホワイトリスト製
- Ingress/Egressを定義する

## 重要そうなパラメータ

APIとサンプルみて

## テンプレ

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978```