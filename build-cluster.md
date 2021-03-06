
# クラスタ構築

## マスター構築

### 取得と配置

- [kubenetes](https://github.com/kubernetes/kubernetes/releases): 1.9.2
- [etcd](https://github.com/coreos/etcd/releases): 3.3.0
- [cfssl](https://cfssl.org/): 1.2

#### kubernetes

githubのリリースから特定のバージョンを落として展開

`kubernetes/cluster/get-kube-binaries.sh` を実行、色々とってきてくれる。
`kubernetes/server/kubernetes-server-linux-amd64.tar.gz` を展開
`kubernetes/server/bin/` の中にある

- kube-apiserver
- kube-controller-manager
- kube-scheduler

と `kubernetes/client/bin/` の中にある

- kubectl

を `/usr/local/bin` 下に持ってくる

#### etcd

展開して出てきたフォルダの中にある

- etcd
- etcdctl

を `/usr/local/bin` 下に持ってくる

#### cfssl

- cfssl
- cfssljson

をDLして `/usr/local/bin` に配置

### cfsslでCAを作る

```bash
mkdir -p /etc/cfssl/ca; cd /etc/cfssl/ca
```

/etc/cfssl/ca/ca-config.json

```json
{
    "signing": {
        "default": {
            "expiry": "43800h"
        },
        "profiles": {
            "server": {
                "expiry": "43800h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            },
            "client": {
                "expiry": "43800h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            },
            "peer": {
                "expiry": "43800h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
```

/etc/cfssl/ca/ca-csr.json

```json
{
    "CN": "kube-test-cluster CA",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "US",
            "L": "CA",
            "ST": "San Francisco",
            "OU": "kube-test-cluster"
        }
    ]
}
```

```bash
cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
```

### etcd

#### 証明書作る

```bash
mkdir -p /etc/cfssl/server/etcd; cd /etc/cfssl/server/etcd
```

/etc/cfssl/server/etcd/server.json

```json
{
    "CN": "k8s-test-cluster",
    "hosts": [
        "127.0.0.1",
        "localhost",
        "10.146.0.3",
        "kube-master"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    }
}
```

```bash
cfssl gencert -ca=/etc/cfssl/ca/ca.pem -ca-key=/etc/cfssl/ca/ca-key.pem -config=/etc/cfssl/ca/ca-config.json -profile=server server.json | \
cfssljson -bare server
```

#### 設定ファイル書くなど

/etc/etcd.yml

```yaml
name: k8s-test-cluster
data-dir: /var/lib/etcd/k8s-test-cluster
advertise-client-urls: https://10.146.0.3:2379,https://127.0.0.1:2379
listen-client-urls: https://0.0.0.0:2379

client-transport-security:
  key-file: /etc/cfssl/server/etcd/server-key.pem
  cert-file: /etc/cfssl/server/etcd/server.pem
  trusted-ca-file: /etc/cfssl/ca/ca.pem
```

```bash
useradd -M etcd
mkdir -p /var/lib/etcd/k8s-test-cluster
chown -R etcd /var/lib/etcd /etc/cfssl/server/etcd
```

/etc/systemd/system/etcd.service

```
[Unit]
Description=etcd key-value store
Documentation=https://github.com/coreos/etcd
After=network.target

[Service]
User=etcd
Type=notify
ExecStart=/usr/local/bin/etcd --config-file /etc/etcd.yml
Restart=always
RestartSec=10s
LimitNOFILE=40000

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload && systemctl start etcd
etcdctl --endpoints https://127.0.0.1:2379 --ca-file /etc/cfssl/ca/ca.pem set /coreos.com/network/config << EOF
{
    "Network": "172.16.0.0/16",
    "SubnetLen": 20,
    "SubnetMin": "172.16.0.0",
    "SubnetMax": "172.16.224.0",
    "Backend": {
        "Type": "udp",
        "Port": 7890
    }
}
EOF
```

### kube-apiserver

### cfsslでk8s用CAを作る

```bash
mkdir -p /etc/kubernetes/ca; cd /etc/kubernetes/ca
```

/etc/kubernetes/ca/ca-config.json

```json
{
    "signing": {
        "default": {
            "expiry": "43800h"
        },
        "profiles": {
            "server": {
                "expiry": "43800h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            },
            "client": {
                "expiry": "43800h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            },
            "peer": {
                "expiry": "43800h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
```

/etc/kubernetes/ca/ca-csr.json

```json
{
    "CN": "kube-test-cluster k8s CA",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "US",
            "L": "CA",
            "ST": "San Francisco",
            "OU": "kube-test-cluster k8s"
        }
    ]
}
```

```bash
cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
```

#### 証明書作る

```bash
mkdir -p /etc/kubernetes/server/kube-apiserver; cd /etc/kubernetes/server/kube-apiserver
```

/etc/cfssl/server/kube-apiserver/server.json

```json
{
    "CN": "k8s-test-cluster k8s",
    "hosts": [
        "127.0.0.1",
        "localhost",
        "10.146.0.3",
        "kube-master",
        "172.16.240.1",
        "kubernetes",
        "kubernetes.default",
        "kubernetes.default.svc"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    }
}
```

```bash
cfssl gencert -ca=/etc/kubernetes/ca/ca.pem -ca-key=/etc/kubernetes/ca/ca-key.pem -config=/etc/kubernetes/ca/ca-config.json -profile=server server.json | \
cfssljson -bare server
chown -R kube: /etc/kubernetes
```

#### その他設定

```bash
useradd -M kube
mkdir -p /run/kubernetes /var/lib/kube-apiserver
touch /var/lib/kube-apiserver/known_tokens.csv
chown -R kube /var/lib/kube-apiserver
```

/etc/systemd/system/kube-apiserver.service

```
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target
After=etcd.service

[Service]
EnvironmentFile=-/run/kubernetes/config.env
EnvironmentFile=-/run/kubernetes/apiserver.env
User=kube
ExecStart=/usr/local/bin/kube-apiserver \
        $KUBE_LOGTOSTDERR \
        $KUBE_LOG_LEVEL \
        $KUBE_ETCD_SERVERS \
        $KUBE_API_ADDRESS \
        $KUBE_API_PORT \
        $KUBELET_PORT \
        $KUBE_ALLOW_PRIV \
        $KUBE_SERVICE_ADDRESSES \
        $KUBE_ADMISSION_CONTROL \
        $KUBE_API_ARGS
Restart=on-failure
Type=notify
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

/run/kubernetes/config.env

```bash
###
# kubernetes system config
#
# The following values are used to configure various aspects of all
# kubernetes services, including
#
#   kube-apiserver.service
#   kube-controller-manager.service
#   kube-scheduler.service
#   kubelet.service
#   kube-proxy.service
# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"

# journal message level, 0 is debug
KUBE_LOG_LEVEL=""

# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow-privileged=false"

# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=https://127.0.0.1:6443"
```

/run/kubernetes/apiserver.env

```bash
###
# kubernetes system config
#
# The following values are used to configure the kube-apiserver
#

# The address on the local server to listen to.
KUBE_API_ADDRESS=""

# The port on the local server to listen on.
KUBE_API_PORT="--insecure-port=0"

# Port minions listen on
# KUBELET_PORT="--kubelet-port=10250"

# Comma separated list of nodes in the etcd cluster
KUBE_ETCD_SERVERS="--etcd-servers=https://127.0.0.1:2379 --etcd-cafile /etc/cfssl/ca/ca.pem"

# Address range to use for services
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=172.16.240.0/20"

# default admission control policies
KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,DefaultTolerationSeconds,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota"

# Add your own!
KUBE_API_ARGS= \
    --token-auth-file /var/lib/kube-apiserver/known_tokens.csv \
    --client-ca-file /etc/kubernetes/ca/ca.pem \
    --tls-ca-file /etc/kubernetes/ca/ca.pem \
    --tls-cert-file /etc/kubernetes/server/kube-apiserver/server.pem \
    --tls-private-key-file /etc/kubernetes/server/kube-apiserver/server-key.pem
```

### kube-controller-manager

```bash
mkdir -p /var/lib/kube-controller-manager
TOKEN=$(dd if=/dev/urandom bs=128 count=1 2>/dev/null | base64 | tr -d "=+/" | dd bs=32 count=1 2>/dev/null)
echo "$TOKEN,system:kube-controller-manager,1,\"system:masters\"" >> /var/lib/kube-apiserver/known_tokens.csv
kubectl config --kubeconfig /var/lib/kube-controller-manager/kubeconfig \
    set-cluster k8s-test-cluster \
    --certificate-authority=/etc/kubernetes/ca/ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443
kubectl config --kubeconfig /var/lib/kube-controller-manager/kubeconfig \
    set-credentials controller-manager \
    --token=$TOKEN
kubectl config --kubeconfig /var/lib/kube-controller-manager/kubeconfig \
    set-context controller-manager-context \
    --cluster=k8s-test-cluster \
    --user=controller-manager
kubectl config --kubeconfig /var/lib/kube-controller-manager/kubeconfig \
    use-context controller-manager-context
chown -R kube /var/lib/kube-controller-manager
```

- [ ] `system:kube-controller-manager` に `system:masters` グループがついてるのはなぜか権限エラーになるため。(適当処置)

/etc/systemd/system/kube-controller-manager

```
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
EnvironmentFile=-/run/kubernetes/config.env
EnvironmentFile=-/run/kubernetes/controller-manager.env
User=kube
ExecStart=/usr/local/bin/kube-controller-manager \
        $KUBE_LOGTOSTDERR \
        $KUBE_LOG_LEVEL \
        $KUBE_MASTER \
        $KUBE_CONTROLLER_MANAGER_ARGS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

/run/kubernetes/controller-manager.env

```bash
###
# The following values are used to configure the kubernetes controller-manager

# defaults from config and apiserver should be adequate

# Add your own!
KUBE_CONTROLLER_MANAGER_ARGS= \
    --kubeconfig /var/lib/kube-controller-manager/kubeconfig \
    --cluster-signing-cert-file /etc/kubernetes/ca/ca.pem \
    --cluster-signing-key-file /etc/kubernetes/ca/ca-key.pem \
    --service-account-private-key-file /etc/kubernetes/server/kube-apiserver/server-key.pem
```

### kube-scheduler

```bash
mkdir -p /var/lib/kube-scheduler
TOKEN=$(dd if=/dev/urandom bs=128 count=1 2>/dev/null | base64 | tr -d "=+/" | dd bs=32 count=1 2>/dev/null)
echo "$TOKEN,system:kube-scheduler,2" >> /var/lib/kube-apiserver/known_tokens.csv
kubectl config --kubeconfig /var/lib/kube-scheduler/kubeconfig \
    set-cluster k8s-test-cluster \
    --certificate-authority=/etc/kubernetes/ca/ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443
kubectl config --kubeconfig /var/lib/kube-scheduler/kubeconfig \
    set-credentials scheduler \
    --token=$TOKEN
kubectl config --kubeconfig /var/lib/kube-scheduler/kubeconfig \
    set-context scheduler-context \
    --cluster=k8s-test-cluster \
    --user=scheduler
kubectl config  --kubeconfig /var/lib/kube-scheduler/kubeconfig \
    use-context scheduler-context
chown -R kube /var/lib/kube-controller-manager
```

/etc/systemd/system/kube-scheduler.service

```
[Unit]
Description=Kubernetes Scheduler Plugin
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
EnvironmentFile=-/run/kubernetes/config.env
EnvironmentFile=-/run/kubernetes/scheduler.env
User=kube
ExecStart=/usr/local/bin/kube-scheduler \
	$KUBE_LOGTOSTDERR \
	$KUBE_LOG_LEVEL \
	$KUBE_MASTER \
	$KUBE_SCHEDULER_ARGS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

/run/kubernetes/scheduler.env

```bash
###
# kubernetes scheduler config

# default config should be adequate

# Add your own!
KUBE_SCHEDULER_ARGS="--kubeconfig /var/lib/kube-scheduler/kubeconfig"
```

### kubelet,kube-proxy用Tokenの作成

```bash
TOKEN=$(dd if=/dev/urandom bs=128 count=1 2>/dev/null | base64 | tr -d "=+/" | dd bs=32 count=1 2>/dev/null)
echo "$TOKEN,kubelet-bootstrap,3,\"system:bootstrappers\"" >> /var/lib/kube-apiserver/known_tokens.csv
TOKEN=$(dd if=/dev/urandom bs=128 count=1 2>/dev/null | base64 | tr -d "=+/" | dd bs=32 count=1 2>/dev/null)
echo "$TOKEN,kube-proxy,4" >> /var/lib/kube-apiserver/known_tokens.csv
TOKEN=$(dd if=/dev/urandom bs=128 count=1 2>/dev/null | base64 | tr -d "=+/" | dd bs=32 count=1 2>/dev/null)
echo "$TOKEN,kube-admin,5,\"system:masters\"" >> /var/lib/kube-apiserver/known_tokens.csv
```

### 色々起動

```bash
systemctl daemon-reload
systemctl start kube-apiserver kube-controller-manager kube-scheduler
```

### ロールバインドをいくつか作成

#### kubectl設定

TOKENにkube-admin用のトークンをセット

```bash
kubectl config set-cluster k8s-test-cluster \
    --certificate-authority=/etc/kubernetes/ca/ca.pem \
    --embed-certs=true \
    --server=https://10.146.0.3:6443
kubectl config set-credentials admin \
    --token=$TOKEN
kubectl config set-context admin-context \
    --cluster=k8s-test-cluster \
    --user=admin
kubectl config use-context admin-context
```

#### ClusterRoleBindingの作成

crb.yaml

```yaml
# Approve all CSRs for the group "system:bootstrappers"
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: auto-approve-csrs-for-group
subjects:
- kind: Group
  name: system:bootstrappers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: system:node-bootstrapper
  apiGroup: rbac.authorization.k8s.io
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: node-client-cert-renewal
subjects:
- kind: Group
  name: system:nodes
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: system:node
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f crb.yaml
```

## Node構築

### 取得と配置

- docker(pkgから)
- socat(pkgから)
- [kubenetes](https://github.com/kubernetes/kubernetes/releases): 1.9.2
- [flannel](https://github.com/coreos/flannel/releases): 0.10.0
- [cni](https://github.com/containernetworking/cni/releases): 0.6.0

#### kubernetes

`kubernetes/cluster/get-kube-binaries.sh` を実行。
`kubernetes/server/kubernetes-server-linux-amd64.tar.gz` を展開
`kubernetes/server/bin/` の中にある

- kube-proxy
- kubelet

と `kubernetes/client/bin/` の中にある

- kubectl

を `/usr/local/bin` 下に持ってくる

#### flannel

展開して出てきた

- flanneld
- mk-docker-opts.sh

を `/usr/local/bin` 下に持ってくる

#### cni

展開して出てきた `cnitool` を `/usr/local/bin` 下に持ってくる

#### CA公開鍵

マスターで作成したk8s用のCAの公開鍵を各ノードの同じ場所に配置する

`/etc/kubernetes/ca/ca.pem` のこと。

#### socat

aptでインストール

port-fowardにこいつが必要らしい

```bash
apt install socat
```

### flannel

```bash
mkdir -p /run/flannel
```

/run/flannel/options.env

```bash
FLANNELD_ETCD_ENDPOINTS=https://10.146.0.3:2379
FLANNELD_ETCD_CAFILE=/etc/cfssl/ca/ca.pem
```

/etc/systemd/system/flanneld.service

```
[Unit]
Description=Network fabric for containers
Documentation=https://github.com/coreos/flannel
After=etcd.service
Before=docker.service

[Service]
Type=notify
Restart=always
RestartSec=5
EnvironmentFile=-/run/flannel/options.env
LimitNOFILE=40000
LimitNPROC=1048576
ExecStartPre=/sbin/modprobe ip_tables
ExecStartPre=/bin/mkdir -p /run/flannel

ExecStart=/usr/local/bin/flanneld

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload && systemctl start flanneld
```

### docker

```bash
apt update
apt install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
apt update
apt install -y docker-ce

iptables -t nat -F
ip link set docker0 down
ip link delete docker0

mkdir -p /etc/systemd/system/docker.service.d
```

/etc/systemd/system/docker.service.d/override.conf

```
[Service]
EnvironmentFile=-/run/docker_opts.env
ExecStartPre=/usr/local/bin/mk-docker-opts.sh
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// $DOCKER_OPTS
```

```bash
systemctl daemon-reload && systemctl restart docker
```

### kubelet

TOKENにkubelet用に生成したトークンをセット。

```bash
useradd -M kube
mkdir -p /var/lib/kubelet /run/kubernetes
kubectl config --kubeconfig /var/lib/kubelet/bootstrap.kubeconfig \
    set-cluster k8s-test-cluster \
    --certificate-authority=/etc/kubernetes/ca/ca.pem \
    --embed-certs=true \
    --server=https://10.146.0.3:6443
kubectl config --kubeconfig /var/lib/kubelet/bootstrap.kubeconfig \
    set-credentials kubelet-bootstrap \
    --token=$TOKEN
kubectl config --kubeconfig /var/lib/kubelet/bootstrap.kubeconfig \
    set-context kubelet-bootstrap-context \
    --cluster=k8s-test-cluster \
    --user=kubelet-bootstrap
kubectl config  --kubeconfig /var/lib/kubelet/bootstrap.kubeconfig \
    use-context kubelet-bootstrap-context
chown -R kube /var/lib/kubelet
```

/run/kubernetes/kubelet.env

`--address` と `--hostname-override` は適宜変更

```bash
###
# kubernetes kubelet (minion) config

# The address for the info server to serve on (set to 0.0.0.0 or "" for all interfaces)
KUBELET_ADDRESS="--address=10.146.0.4"

# The port for the info server to serve on
# KUBELET_PORT="--port=10250"

# You may leave this blank to use the actual hostname
KUBELET_HOSTNAME="--hostname-override=kube-node-1"

# Add your own!
KUBELET_ARGS= \
    --cni-bin-dir /usr/local/bin \
    --bootstrap-kubeconfig /var/lib/kubelet/bootstrap.kubeconfig \
    --kubeconfig /var/lib/kubelet/kubeconfig \
    --cluster-dns 172.16.255.254 \
    --cluster-domain cluster.local \
    --rotate-certificates
```

/run/kubernetes/config.env

```bash
###
# kubernetes system config
#
# The following values are used to configure various aspects of all
# kubernetes services, including
#
#   kube-apiserver.service
#   kube-controller-manager.service
#   kube-scheduler.service
#   kubelet.service
#   kube-proxy.service
# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"

# journal message level, 0 is debug
KUBE_LOG_LEVEL=""

# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow-privileged=false"

# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=https://10.146.0.3:6443"
```

/etc/systemd/system/kubelet.service

```
[Unit]
Description=Kubernetes Kubelet Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Requires=docker.service

[Service]
WorkingDirectory=/var/lib/kubelet
EnvironmentFile=-/run/kubernetes/config.env
EnvironmentFile=-/run/kubernetes/kubelet.env
ExecStart=/usr/local/bin/kubelet \
        $KUBE_LOGTOSTDERR \
        $KUBE_LOG_LEVEL \
        $KUBELET_API_SERVER \
        $KUBELET_ADDRESS \
        $KUBELET_PORT \
        $KUBELET_HOSTNAME \
        $KUBE_ALLOW_PRIV \
        $KUBELET_ARGS
Restart=on-failure
KillMode=process

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload && systemctl restart kubelet
```

### kube-proxy

#### 必要そうなpkgを入れる

```bash
apt install conntrack
```

#### 設定

TOKENにkube-proxy用に生成したトークンをセット。

```bash
mkdir -p /var/lib/kube-proxy
kubectl config --kubeconfig /var/lib/kube-proxy/kubeconfig \
    set-cluster k8s-test-cluster \
    --certificate-authority=/etc/kubernetes/ca/ca.pem \
    --embed-certs=true \
    --server=https://10.146.0.3:6443
kubectl config --kubeconfig /var/lib/kube-proxy/kubeconfig \
    set-credentials proxy \
    --token=$TOKEN
kubectl config --kubeconfig /var/lib/kube-proxy/kubeconfig \
    set-context proxy-context \
    --cluster=k8s-test-cluster \
    --user=proxy
kubectl config  --kubeconfig /var/lib/kube-proxy/kubeconfig \
    use-context proxy-context
chown -R kube /var/lib/kube-proxy
```

/etc/systemd/system/kube-proxy.service

```
[Unit]
Description=Kubernetes Kube-Proxy Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
EnvironmentFile=-/run/kubernetes/config.env
EnvironmentFile=-/run/kubernetes/proxy.env
ExecStart=/usr/local/bin/kube-proxy \
        $KUBE_LOGTOSTDERR \
        $KUBE_LOG_LEVEL \
        $KUBE_MASTER \
        $KUBE_PROXY_ARGS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

/run/kubernetes/proxy.env

```bash
###
# kubernetes proxy config

# default config should be adequate

# Add your own!
KUBE_PROXY_ARGS="--kubeconfig /var/lib/kube-proxy/kubeconfig"
```

```bash
systemctl daemon-reload && systemctl restart kube-proxy
```

## クライアント設定

### 取得と配置

- [kubenetes](https://github.com/kubernetes/kubernetes/releases): 1.9.2

#### kubernetes

githubのリリースから特定のバージョンを落として展開

`kubernetes/cluster/get-kube-binaries.sh` を実行。
 
 `kubernetes/client/bin/` の中にある

- kubectl

を `/usr/local/bin` 下に持ってくる

### CA公開鍵

マスターで作成したk8s用のCAの公開鍵を各ノードの同じ場所に配置する

TOKENにkubectl用に生成したトークンをセット。

```bash
kubectl config set-cluster k8s-test-cluster \
    --certificate-authority=/etc/kubernetes/ca/ca.pem \
    --embed-certs=true \
    --server=https://10.146.0.3:6443
kubectl config set-credentials admin \
    --token=$TOKEN
kubectl config set-context admin-context \
    --cluster=k8s-test-cluster \
    --user=admin
kubectl config use-context admin-context
```

#### 接続テスト

```bash
kubectl auth can-i get nodes
```

## 署名リクエストの承認

```bash
kubectl get csr -o yaml | kubectl certificate approve -f -
```

### NODEの確認

```bash
kubectl get nodes
```

## kube-dnsの構築

```bash
sed -e 's/__PILLAR__DNS__SERVER__/172.16.255.254/g' \
    -e 's/__PILLAR__DNS__DOMAIN__/cluster.local/g' \
    kubernetes/cluster/addons/dns/kube-dns.yaml.base | \
kubectl apply -f -
```

(これが動作すればservice経由でAPIを蹴ってるのでserviceが正しく動作している)

### テスト

```bash
kubectl delete -f - << EOF
apiVersion: v1
kind: Pod
metadata:
  name: alpine
spec:
  containers:
  - image: alpine
    command:
      - tail
      - -f
      - /dev/null
    imagePullPolicy: IfNotPresent
    name: alpine
  restartPolicy: Always
  terminationGracePeriodSeconds: 0
EOF
kubectl exec alpine nslookup alpine 172.16.255.254
kubectl exec alpine nslookup kubernetes 172.16.255.254
kubectl exec alpine cat /etc/resolv.conf
kubectl delete po/alpine
```