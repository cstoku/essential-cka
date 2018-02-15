
# Advanced Commands

## apply

設定の適用、リソースが無い場合には作成を行う

applyを使うなら最初に `apply` か `create --save-config` すること
(上記2つの操作をしないと適用時の設定ファイルが保存されないため)

```bash
kubectl apply -f alpine.yaml
```

`create` してから `apply` するとWarningでるけど、実行する前に

```bash
kubectl apply set-last-applied -f alpine.yaml --create-annotation
```

するとアノテーションセットしてくれてWarning出てこなくなる。

### edit-last-applied

last-applied-configurationアノテーションを編集

### set-last-applied

last-applied-configurationアノテーションをセット

この時、リソースのアノテーション書き換わるだけでそれ以外の変更はされない

### view-last-applied

last-applied-configurationアノテーションを表示

## patch

リソースにパッチを当てる

```bash
# パッチ適用
kubectl patch deploy/alpine -p '{"spec": {"replicas":2}}'

# nginxコンテナのイメージを変更(nameがキーになっている)
kubectl patch pod valid-pod -p '{"spec":{"containers":[{"name":"nginx","image":"nginx:1.13"}]}}'

# jsonパッチでパッチの適用をする
kubectl patch pod valid-pod --type='json' -p='[{"op": "replace", "path":"/spec/containers/0/image", "value":"httpd"}]'
```

## replace

リソースの置き換え。 `create` で作ったリソースの変更で使用するのだと思ってる。
`apply` 使うなら必要ないかと。

```bash
kubectl replace -f alpine.yaml
```

## convert

異なるAPI間の設定ファイルの変換

```bash
# 最終バージョンのAPIに変換
kubectl convert -f alpine

# バージョンを指定しての変換
kubectl convert -f alpine --output-version 'apps/v1beta'
```