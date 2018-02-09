
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



## exec



## port-forward



## proxy



## cp



## auth



