
# Other Commands

## api-versions

api versionの一覧を表示

## config

### current-context

現在のコンテキストを表示

### delete-cluster

クラスタ情報の削除

### delete-context

コンテキストの削除

### get-clusters

クラスタの一覧を表示

### get-contexts

コンテキストの一覧を表示

### rename-context

コンテキストのリネーム

### set

プロパティの値を設定

### set-cluster

クラスタ情報の設定

```bash
kubectl config set-cluster k8s-cluster \
    --server=https://10.1.10.1
    --insecure-skip-tls-verify
```

### set-context

コンテキスト情報の設定

```bash
kubectl config set-context k8s-context \
    --cluster=k8s-cluster
    --user=k8s-user
```

### set-credentials

認証情報の設定

```bash
kubectl config set-credentials k8s-cred \
    --username=k8s-user
    --password=k8s-pass
```

### unset

プロパティの値を削除

### use-context

使用するコンテキストを選択

### view

kubeconfigファイルの表示

## help

コマンドの表示

`kubectl options` でグローバルなオプションのヘルプが見える

## plugin

CLIプラグインの実行

## version

バージョンの表示