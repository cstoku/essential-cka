
# Deploy Commands

## rollout

ロールアウト関連のコマンド

- history
- pause
- resume
- status
- undo

以下のリソースを指定可能

- deployments
- daemonsets
- statefulsets

```bash
# ロールアウトの履歴確認
kubectl rollout history deploy/alpine

# ロールバック
kubectl rollout undo deploy/alpine --to-revision=1
```

## rolling-update

ReplicationController用なので放置ｗ

## scale

スケール処理

以下のリソースで実行可能

- Deployment
- ReplicaSet
- Replication Controller
- Job

```bash
# podを3にスケール
kubectl scale --replicas=3 deploy/alpine

# 現在のpod数が2だった場合に5にスケール
kubectl scale --current-replicas=2 --replicas=5 deploy/alpine
```

## autoscale

オートスケール設定

以下のリソースで実行可能

- Deployment
- ReplicaSet
- Replication Controller

```bash
# デフォルトのオートスケールポリシーで2〜10までオートスケール
kubectl autoscale deploy alpine --min=2 --max=10

# cpu使用率が80%をターゲットに1〜5でスケール
kubectl autoscale deploy alpine --min=1 --max=5 --cpu-percent=80
```