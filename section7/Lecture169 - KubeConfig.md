# Lecture 169 - KubeConfig

## curl과 kubectl을 이용한 인증서 인증

### curl 을 이용한 인증서 인증

```bash
$ curl <https://my-kube-playground:6443/api/v1/pods> \\
--key admin.key \\
--cert admin.crt \\
--cacert ca.crt

{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
  "selfLink": "/api/v1/pods"
  },
  "items": []
}

```

### API 를 이용한 인증서 인증

```bash
$ kubectl get pods \\
--server <https://my-kube-playground:6443> \\
--client-key admin.key \\
--client-certificate admin.crt \\
--certificate-authority ca.crt

```

이러한 옵션을 매번 입력하는 대신, 이를 kubeconfig 파일로 옮겨 작업 흐름을 간소화하세요.

## Kubeconfig 파일 이해

- 위와 같이 옵션을 매번 입력하는 대신, `kubeconfig` 파일로 옮겨 작업 흐름을 간소화 할 수 있다.
- `kubectl` 은 `~/.kube` 디렉토리에서 `config` 라는 이름의 `kubeconfig` 파일을 검색한다.
- 그리고 `kubectl` 은 파일 내에 정의된 구성을 자동으로 사용한다.
- `kubeconfig` 파일은 세 가지 주요 섹션으로 구성된다.
    - Cluster
        - 액세스해야 하는 Kubernetes 클러스터를 정의.
        - 개발기, 프로덕션 환경, 다양한 클라우드 공급자가 호스팅하는 클러스터 등등.
    - Users
        - 클러스터에 대한 권한이 있는 사용자 계정과 관련 자격 증명 지정.
        - 관리자, 게발자, 프로덕션 사용자 등등.
    - Contexts
        - 어떤 사용자가 어떤 클러스터에 접근해야 하는지 지정하여 클러스터와 사용자를 연결
        - 기본 네임스페이스 등등.

```bash
apiVersion: v1
kind: Config
clusters:
- name: my-kube-playground  # values hidden…
- name: development
- name: production
- name: google
contexts:
- name: my-kube-admin@my-kube-playground
- name: dev-user@google
- name: prod-user@production
users:
- name: my-kube-admin
- name: admin
- name: dev-user
- name: prod-user

```

- 위 구성은 `my kube playground` 클러스터에 대한 서버 사영한 `clusters` 섹션에 정의되어 있다.
- 관리자 사용자의 자격 증명은 `users` 세션에 정의되어 있다.
- `my-kube-admin@my-kube-playground` 명명된 컨텍스트가 이를 연결한다.
- 여러 클러스터와 사용자에 대해 여러 컨텍스트를 생성할 수 있으며, `current-context` 필드를 사용하여 기본 컨텍스트를 설정할 수 있다.

## Kubeconfig 보기 및 사용자 지정

- `kubectl config` 명령을 사용하여 필요에 따라 항목을 업데이트하거나 삭제할 수 있다.

```bash
$ kubectl config view

apiVersion: v1
kind: Config
current-context: kubernetes-admin@kubernetes
clusters:
- cluster:
    certificate-authority-data: REDACTED
    server: <https://172.17.0.5:6443>
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED

$ kubectl config view --kubeconfig=my-custom-config

apiVersion: v1
kind: Config
current-context: my-kube-admin@my-kube-playground
clusters:
- name: my-kube-playground
- name: development
- name: production
contexts:
- name: my-kube-admin@my-kube-playground
- name: prod-user@production
users:
- name: my-kube-admin
- name: prod-user

```

### 활성 컨텍스트를 변경

```bash
$ kubectl config use-context prod-user@production

apiVersion: v1
kind: Config
current-context: prod-user@production
clusters:
- name: my-kube-playground
- name: development
- name: production
contexts:
- name: my-kube-admin@my-kube-playground
- name: prod-user@production
users:
- name: my-kube-admin
- name: prod-user

```

## 기본 네임스페이스 구성

- 쿠버네티스의 네임스페이스는 클러스터를 여러 가상 클러스터로 분할하는 데 도움이 된다.
- 특정 네임스페이스를 자동으로 사용하도록 컨텍스트를 구성할 수 있다.
- 이 컨텍스트로 전환하면 kubectl은 지정된 네임스페이스 내에서 자동으로 작동한다.

```bash
apiVersion: v1
kind: Config
clusters:
- name: production
  cluster:
    certificate-authority: ca.crt
    server: <https://172.17.0.51:6443>
contexts:
- name: admin@production
  context:
    cluster: production
    user: admin
    namespace: finance
users:
- name: admin
  user:
    client-certificate: admin.crt
    client-key: admin.key

```

## Kubeconfig 파일에서 인증서 관리

- `kubeconfig` 파일에 인증서응 등록하여 사용할 수 있다.
- 모범사례의 경우 인증서 파일의 전체 경로를 명시해서 사용하는 것이다.
- `certificate-authority-data` 필드를 사용해서 인증서 데이터를 직접 포함하여 사용할 수도 있다.

```bash
apiVersion: v1
kind: Config
clusters:
- name: production
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt

```

```bash
apiVersion: v1
kind: Config
clusters:
- name: production
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    certificate-authority-data: LS0tLS1CRUdJTiBD...

```