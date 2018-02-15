
# Settings Commands

## label

リソースのラベルを更新する

```bash
# ラベルのセット
kubectl label deploy/alpine hoge=fuga

# ラベルの更新
kubectl label deploy/alpine --overwrite hoge=fugafuga

# ラベルの削除
kubectl label deploy/alpine hoge-
```

## annotate

リソースのアノテーションを更新する

```bash
# アノテーションのセット
kubectl annotate deploy/alpine description='test deployment'

# アノテーションの更新
kubectl annotate deploy/alpine --overwrite description='test description'

# アノテーションの削除
kubectl annotate deploy/alpine description-
```

## completion

Shellの補完コードの出力

```bash
# bashの補完を適用
source <(kubectl completion bash)
```