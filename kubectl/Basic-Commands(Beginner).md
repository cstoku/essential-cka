
# Basic Commands (Beginner)

## create

リソースの作成

```bash
# ファイルから作成
kubectl create -f deployment.yaml

# stdinから作成
cat deployment.yaml | kubectl create -f -

# cli上で作成
kubectl create deploy --image nginx
```

cli上で作成できるリソース

- clusterrole
- clusterrolebinding
- configmap
- deployment
- namespace
- poddisruptionbudget
- priorityclass
- quota
- role
- rolebinding
- secret
- service
- serviceaccount

## expose

serviceとしてリソースを公開

可能なリソースは

- pod
- service
- replicationcontroller
- deployment
- replicaset

```bash
# ポートを公開
kubectl expose pod nginx --port 80

# ポートを転送して公開&serviceの名前指定
kubectl expose deploy nginx --port 8080 --target-port 80 --name nginx-service

# NodePortとして公開
kubectl expose deploy nginx --port 80 --type NodePort
```

## run

deploymentかjobを作成し実行

```bash
# 単体実行
kubectl run nginx --image nginx

# envを設定&ポート公開
kubectl run nginx --image nginx --env "HOGE=fuga" --port 80

# stdinにアタッチとtty割り当て、commandの上書き
kubectl run alpine --image alpine -it --command -- ash

# job実行
kubectl run pi --schedule="0/5 * * * ?" --image=perl --restart=OnFailure -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```

## set

 値のセット
 
 - env
 - image
 - resources
 - selector
 - serviceaccount
 - subject
 
```bash
# sample-buildの環境変数STORAGE_DIRに/dataをセット
kubectl set env deployment/sample-build STORAGE_DIR=/data

# nginxをnginx:1.9.1に、busyboxをbusyboxにセット
kubectl set image deployment/nginx busybox=busybox nginx=nginx:1.9.1
```