
# Lecture 86 - Admission Controllers

## 인증 및 권한부여 흐름

### 인증

- API 서버는 요청에 대한 인증을 처리한다.
- 예를 들어 `kubectl` 을 통해 명령을 실행하면, `KubeConfig` 파일이 필요한 인증서를 제공한다.

```bash

$ cat ~/.kube/config

apiVersion: v1
clusters:
- cluster:
  certificate-authority-data: LS0tCIlRjdG......BQyMjAwFURs9tCg==
  server: https://example.com
  name: example-cluster

```

### 권한 부여(RBAC)
- 인증이 완료되면 API 서버는 사용자가 요청된 작업을 수행할 권한이 있는지 확이한다.
- 역할 기반 접근 제어(RBAC, Roll Based Access Control) 이 일반적으로 사용된다.
- 사용자는 Pod를 나열, 가져오기, 생성, 업데이트 또는 삭제할 수 있다.
- RBAC는 특정 리소스 이름이나 네임스페이스에 대한 액세스를 제한할 수도 있다.
- 이러한 RBAC 규칙은 API 수준에서 적용되며 사용자가 액세스할 수 있는 API 작업을 결정

```bash

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
```

## Admission Controller 역할

![img_1.png](img_1.png)

- RBAC는 기본저인 권한 부여를 처리하지만, Advanced Validations 이나 Mutations 기능은 제공하지 않는다.
- Admission Controller 는 리소스가 ETCD 에 저장되기 전에 요청을 검증하고, 구성을 수정하고, 추가 작업을 진행할 수 있다.
    - 파드 사양 검증
        - 이미지가 Docker Hub 레지스트레이서 나온 것인지 확인
        - latest 태그 금지
    - 루트 사용자로 컨테이너 실행하는 파드를 금지
    - 레발과 같은 필수 메타데이터 포함되어 있는지 확인.

### Built-In Admission Controller
- 쿠버네티스에는 내장되어 있는 Admission Controller 가 몇개 존재한다.
- AlwaysPullImages
    - 각 파드가 생성될 때 이미지를 강제로 새로 받아오는 역할
- DefaultStorageClass
    - 특정 스토리지가 할당되지 않을 경우 PVC 에 기본 스토리지를 자동으로 할당.
- EventRateLimit
    - 과부화 방지를 위한 동시 API 서버에 대한 요청을 제한.
- NamespaceExists
    - 존재하지 않는 네임스페이스에 대한 요청을 거부
- NamespaceAutoProvision
    - 누락된 네임스페이스 자동 생성.
    - 더 이상 사용되지 않음.
    - 해당 Admission Controller 는 NamespaceLifecycle Admission Controller 로 대체되었음.
- NamespaceLifecycle
    - 존재하지 않는 네임스페이스에 대한 요청을 거부.
    - 기본 네임스페이스 (default, kube-system, kube-public) 가 삭제되지 않도록 보호.

```bash

# 활성화 되어 있는 Admission Controller 확인
$ kube-apiserver -h | grep enable-admission-plugins

```

## Admission Controller 구성

- 새로운 Admission Controller 를 추가하기 위해서는 `--enable-admission-plugins` 해당 옵션을 사용해야 한다.
- 특정 Admission Controller 를 비활성화 하기 위해서는 `disable-admission-plugins` 옵션을 사용한다.
- 주의사항: `--enable-admission-plugins=NodeRestriction,NamespaceAutoProvision` 에 대해서 쉼표 구분자 이후 뛰어쓰가 있을 경우 제대로 실행되지 않음.

### 기존 Kube API-Server

```bash
$ vi /etc/kubernetes/manifest/kube-apiserver.yaml

ExecStart=/usr/local/bin/kube-apiserver \
    --advertise-address=${INTERNAL_IP} \
    --allow-privileged=true \
    --apiserver-count=3 \
    --authorization-mode=Node,RBAC \
    --bind-address=0.0.0.0 \
    --enable-swagger-ui=true \
    --etcd-servers=https://127.0.0.1:2379 \
    --event-ttl=1h \
    --runtime-config=api/all \
    --service-cluster-ip-range=10.32.0.0/24 \
    --service-node-port-range=30000-32767 \
    --v=2 \
    --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision

```

### Kubeadm
```bash

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --authorization-mode=Node,RBAC
    - --advertise-address=172.17.0.107
    - --allow-privileged=true
    - --enable-bootstrap-token-auth=true
    - --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
    image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
    name: kube-apiserver

```

