
# CronJob

https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

## 概要

- 時間ベースで管理されるJob
- 繰り返しなど

## 重要そうなパラメータ

基本Jobのまま。

### spec.schedule

Cronの書式でのスケジュールを指定

## テンプレ

```yaml
apiVersion: batch/v2alpha1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      activeDeadlineSeconds: 10
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```