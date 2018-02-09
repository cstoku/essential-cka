
# Basic Commands (Intermediate)

## get

リソース一覧の取得

```bash
# Nodeの一覧の取得
kubectl get nodes

# deploymentの一覧をyamlで表示
kubectl get deploy -o yaml

# podの詳細一覧表示
kubectl get po -o wide
```

## explain

リソースについてのドキュメントを表示

```bash
# deploymentのフィールドについてのドキュメントを表示
kubectl explain deploy

# podについての有効なフィールドを再帰的に表示
kubectl explain po --recursive
```

## edit

リソースの編集

```bash
# deploy/alpineを編集
kubectl edit deploy/alpine
```

## delete

リソースの削除

```bash
# deploy/alpineの削除
kubectl delete deploy/alpine

# podの全削除
kubectl delete pods --all
```