
# DaemonSet

https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#daemonset-v1-apps

## 概要

- 全てのノードにPodが配置され、実行される
- NodeSelectorで指定することも可能

## 重要そうなパラメータ

labelSelectorでPodのLabelを指定すること

あとは基本Podと同じ

## テンプレ

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  # Unique key of the DaemonSet instance
  name: daemonset-example
spec:
  selector:
    matchLabels:
      app: daemonset-example
  template:
    metadata:
      labels:
        app: daemonset-example
    spec:
      containers:
      # This container is run once on each Node in the cluster
      - name: daemonset-example
        image: ubuntu:trusty
        command:
        - /bin/sh
        args:
        - -c
        # This script is run through `sh -c <script>`
        - >-
          while [ true ]; do
          echo "DaemonSet running on $(hostname)" ;
          sleep 10 ;
          done
```