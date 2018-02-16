
# Job

https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#job-v1-batch

## 概要

- Podを一つ以上作成し、指定された数だけ正常終了するのを保証する

## 重要そうなパラメータ

### spec.backoffLimit

Job失敗の際のリトライ回数

### spec.completions

正常終了するべき回数？ようは実行回数

### spec.parallelism

同時実行並列数

`(.spec.completions - .status.successful) < .spec.parallelism`

という条件らしい

### spec.template

Podに同じ

## テンプレ

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  # Unique key of the Job instance
  name: example-job
spec:
  template:
    metadata:
      name: example-job
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl"]
        args: ["-Mbignum=bpi", "-wle", "print bpi(2000)"]
      # Do not restart containers after they exit
      restartPolicy: Never
```