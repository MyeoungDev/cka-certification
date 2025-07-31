# Kube-API server

## Kube-apiserver 의 역할

- 중앙 관리 구성 요소
  - kube-apiserver 는 kubernetes 클러스터의 모든 작업을 조정하는 중심저인 관리 구성 요소.
- API 노출
  - `kubectl` 명령이나 직접적인 API 호출을 통해 클러스터와 상호작용하는 유일한 컴포넌트.
- 요청 처리
  - 사용자의 요청을 가장 먼저 받아 **인증(Authentication) 과 유효성 검사(Validation)** 을 수행.
- ETCD 와 상호작용
  - etcd 데이터 저장소와 직접 상호작용하는 유일한 컴포넌트.
  - 다른 모든 컴포넌트는(Scheduler, Controller Manager, kubelet) 은 kube-apiserver 를 통해서만 클러스터의 데이터를 읽고 쓰기 가능.
- 상태 업데이트
  - 클러스터의 상태에 변화가 생기면(예시: 파드 생성), kube-apiserver 는 etcd 에 해당 정보를 업데이트하고, 다른 컴포넌트들에게 변경 사항 전달.

```
$ kubectl get nodes

NAME      STATUS   ROLES    AGE   VERSION
master    Ready    master   20m   v1.11.3
node01    Ready    <none>   20m   v1.11.3

```

## 파드 생성 과정에서의 Kube-apiserver 의 과정

A. 사용자 요청

- `kubectl` 이나 API 를 통해 파드 생성 요청이 `kube-apiserver` 에 전달.
- 
B. 인증 및 유효성 검사
- `kube-apiserver` 가 요청 검증.
- 파드 객체 생성.
- etcd 정보 저장.
- 현재 시점 까지는 아직 파드에 할당된 노드가 없음.

C. 스케쥴러와 통신
- `kube-scheduler` 는 `kube-apiserver` 를 지속적으로 모니터링하여, 노드가 할당되지 않는 새로운 파드가 있음을 감지.

D. 노드 할당
- 스케쥴러가 최적의 노드를 식별.
- `kube-apiserver` 에게 전달.
- `kube-apiserver` 는 `etcd`에 정보 업데이트.

E. kubelet 통신
- `kube-apiserver` 는 할당된 노드의 `kubelet`에게 파드 생성 지시.

F. 파드 생성 및 상태 보고
- `kubelet` 은 컨테이너 런타임 엔진을 통해 실제 파드 생성.
- 생성 완료후 `kube-apiserver` 에 파드 상태 보고.

G. 최종 상태 업데이트
- `kube-apiserver` 는 `etcd`의 클러스터 데이터를 최종 상태로 업데이트.

## kube-apiserver의 설치 및 구성

1. kube-apiserver의 설치 및 구성

binary 설치

```
$ wget <https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-apiserver>

# kube-apiserver.service
ExecStart=/usr/local/bin/kube-apiserver \\\\
  --advertise-address=${INTERNAL_IP} \\\\
  --allow-privileged=true \\\\
  --apiserver-count=3 \\\\
  --authorization-mode=Node,RBAC \\\\
  --bind-address=0.0.0.0 \\\\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\\\
  --enable-admission-plugins=Initializers,NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\\\
  --enable-swagger-ui=true \\\\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\\\
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\\\
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\\\
  --etcd-servers=https://127.0.0.1:2379 \\\\
  --event-ttl=1h \\\\
  --experimental-encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\\\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\\\
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\\\
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\\\
  --kubelet-https=true \\\\
  --runtime-config=api/all \\\\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\\\
  --service-cluster-ip-range=10.32.0.0/24 \\\\
  --service-node-port-range=30000-32767 \\\\
  --v=2

```

- kube-apiserver 는 인증(SSL/TLS), 권한, 보안 및 다른 컴포넌트(ETCD) 와 연결 등을 설정.

## kube-apiserver 배포 확인

```
$ kubectl get pods -n kube-system

NAMESPACE      NAME                                        READY   STATUS    RESTARTS   AGE
kube-system    coredns-78fcdf6894-hwrq9                    1/1     Running   0          16m
kube-system    coredns-78fcdf6894-rzhjr                    1/1     Running   0          16m
kube-system    etcd-master                                 1/1     Running   0          15m
kube-system    kube-apiserver-master                       1/1     Running   0          15m
kube-system    kube-controller-manager-master              1/1     Running   0          15m
kube-system    kube-proxy-lzt6f                            1/1     Running   0          16m
kube-system    kube-proxy-zm5qd                            1/1     Running   0          15m
kube-system    kube-scheduler-master                       1/1     Running   0          15m
kube-system    weave-net-29z42                             2/2     Running   1          16m
kube-system    weave-net-snm1l                             2/2     Running   1          16m

```

```
$ cat /etc/systemd/system/kube-apiserver.service

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\\\
  --advertise-address=${INTERNAL_IP} \\\\
  --allow-privileged=true \\\\
  --apiserver-count=3 \\\\
  --audit-log-maxage=30 \\\\
  --audit-log-maxbackup=3 \\\\
  --audit-log-maxsize=100 \\\\
  --audit-log-path=/var/log/audit.log \\\\
  --authorization-mode=Node,RBAC \\\\
  --bind-address=0.0.0.0 \\\\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\\\
  --enable-admission-plugins=Initializers,NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\\\
  --enable-swagger-ui=true \\\\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\\\
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\\\
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\\\
  --etcd-servers=https://10.240.0.10:2379,<https://10.240.0.11:2379>,<https://10.240.0.12:2379> \\\\
  --event-ttl=1h \\\\
  --experimental-encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\\\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\\\
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\\\
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\\\
  ...

```