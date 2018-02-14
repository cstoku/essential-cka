
# Troubleshooting and Debugging Commands

## describe

指定されたリソースの詳細を表示

```bash
kubectl describe deploy/alpine
```

## logs

Pod内のコンテナやリソースのログを表示
Podが1つのコンテナしか持たない場合はコンテナ名は指定しなくても良い

```bash
# labelがapp=nginxがついているもののlogを表示
kubectl logs -lapp=nginx

# webappというPod内にあるnginxコンテナのログをリアルタイムで表示
kubectl logs -f -c nginx webapp
```

## attach

指定リソースのコンテナにアタッチ

```bash
# コンテナにアタッチ
kubectl attach deploy/alpine

# 標準入力とTTY割り当ててアタッチ
kubectl attach -it -c alpine deploy/alpine 
```

## exec

コンテナ上でプロセスの起動

```bash
# pod内のコンテナを指定してbashを起動
kubectl exec -it alpine -- ash
```

## port-forward

Podへのポートフォワード

```bash
# mysql Podに対し3306ポートをフォワード
kubectl port-forward mysql 3306

# nginx Podに対し8080番から80番へポートフォワード
kubectl port-forward nginx 8080:80
```

## proxy

apiserverへのproxyサーバーを提供

```bash
kubectl proxy
```

## cp

ファイル・フォルダのコピー

コンテナ内にtarのライブラリがナイト失敗する

```bash
kubectl cp hoge.zip alpine:/tmp
```

## auth

権限の調査系

### can-i

操作が許可されているかの確認

```bash
# Podの作成が可能か確認
kubectl auth can-i get create pods
```

### reconcile

調査？をした上でapplyをしてくれる
RBAC系のリソースはapplyよりこちらを使用するべきらしい

よく分かってないので再度調査したい・・。

```bash
kubectl auth reconcile -f rsrc.yaml
```