
# Cluster Management Commands

## certificate

証明書署名申請の承認と拒否をする

```bash
# 承認
kubectl certificate approve csr/kube-node-1

# 拒否
kubectl certificate deny csr/kube-node-1
```

## cluster-info

クラスタの情報を出力

```bash
kubectl cluster-info

# 詳細を出力
kubectl cluster-info dump
```

## top

リソース使用率の表示


以下2つのリソースを指定可能

- node
- pod

```bash
# podの使用率
kubectl top po alpine

# nodeの使用率
kubectl top node
```

heapster要求してくるっぽい

## cordon

指定Nodeをスケジュール対象から外す

```bash
kubectl cordon kube-node-1
```

## uncordon

指定Nodeをスケジュール対象に入れる

```bash
kubectl uncordon kube-node-1
```

## drain

指定NodeのPodを退避

```bash
# DaemonSetを無視し、ReplicaSetなどで管理されてないPodがあっても退避させる
kubectl drain --ignore-daemonsets --force kube-node-1
```

## taint

nodeにテイントする

key=value:effectの形式で指定

```bash
# taintをうつ
kubectl taint kube-node-1 dedicated=experimental:NoSchedule

# keyとeffectが一致するtaintを削除
kubectl taint kube-node-1 dedicated:NoSchedule-

# keyが一致するtaintを削除
kubectl taint kube-node-1 dedicated-
```