# Lecture 174 - Authorization

- 관리자는 Pod 나 노드와 같은 객체를 생성하거나 삭제할 수 있는 완전한 제어권을 갖는다.
- 클러스터가 확장되고, 다양한 사용자 및 외부 애플리케이션 등을 포함한 더 많은 사용자가 시스템에 액세스하멩 따라, 각 사용자 역할에 필요한 액세스 수준만 제공하는 것이 중요하다.
- 쿠버네티는 다음과 같은 여러 권한 부여 매커니즘을 지원한다.
  - 노드 권한 부여
  - 속성 기반 권한 부여
  - 역할 기반 액세스 제어
  - 웹훅 인증

## 속성 기반 권한 부여

- 속성 기반 권한 부여는 특정 사용자 또는 그룹을 정의된 권한 집합과 연결한다.
- JSON 형식의 정책 파일을 생성하여 API 서버에 전달하는 방식이다.
- 보안 요구 사항이 변경될 때마다 정책 파일을 수동으로 업데이트하고 Kube API 서버를 다시 시작해야 한다는 번거로운 점이 존재한다.

```bash
{"kind": "Policy", "spec": {"user": "dev-user", "namespace": "*", "resource": "pods", "apiGroup": "*"}}
{"kind": "Policy", "spec": {"user": "dev-user-2", "namespace": "*", "resource": "pods", "apiGroup": "*"}}
{"kind": "Policy", "spec": {"group": "dev-users", "namespace": "*", "resource": "pods", "apiGroup": "*"}}
{"kind": "Policy", "spec": {"user": "security-1", "namespace": "*", "resource": "csr", "apiGroup": "*"}}
```

## 역할 기반 액세스 제어(RBAC)

- RBAC 는 개발 사용자에게 권한을 직접 연결하는 대신 역할을 정의하여 사용자 권한 관리를 간소화 한다.
- 권한에 연결된 모든 사용자에게 적용되며, 사용자 액세스 권한 변경은 역할을 업데이트하면 처리된다.

## 외부 승인 메커니즘

- 쿠버네티스 기본 메커니즘 대신 외부에서 권한 부여를 관리하려는 경우 OPA(Open Policy Agent) 와 같은 도구를 사용할 수 있다.
  - https://www.openpolicyagent.org/
- OPA 는 쿠버네티스에서 API 호출을 통해 사용자 정보 와 접근 요구사항을 처리하여 접근 제어와 권한 부여를 모두 처리할 수 있다.

## AlwaysAllow 및 AlwaysDeny 모드

- AlwaysAllow : 어떠한 권한 검사도 수행하지 않고 모든 요청을 허용
- AlwaysDeny : 모든 요청을 거부
- Kube API 서버의 authorization-mode 옵션을 사용하여 구성되며, 어떤 권한 부여 메커니즘이 활성화되어 있는지 확인하는 데 중요
- 모드를 지정하지 않으면 기본적으로 AlwaysAllow가 사용

```bash
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --authorization-mode=AlwaysAllow \\
  --bind-address=0.0.0.0 \\
  --enable-swagger-ui=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/apiserver-etcd-client.crt \\
  --etcd-keyfile=/var/lib/kubernetes/apiserver-etcd-client.key \\
  --etcd-servers=https://127.0.0.1:2379 \\
  --event-ttl=1h \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/apiserver-etcd-client.crt \\
  --kubelet-client-key=/var/lib/kubernetes/apiserver-etcd-client.key \\
  --service-node-port-range=30000-32767 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --tls-cert-file=/var/lib/kubernetes/apiserver.crt \\
  --tls-private-key-file=/var/lib/kubernetes/apiserver.key \\
  -v=2
```

- 여러 권한 부여 모드를 쉼표로 구분하여 여러개를 사용할 수 있다.
- 여러 모드가 구성된 경우, 각 요청은 지정된 순서대로 순차적으로 처리된다.

```bash
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --authorization-mode=Node,RBAC,Webhook \\
  --bind-address=0.0.0.0 \\
  --enable-swagger-ui=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/apiserver-etcd-client.crt \\
  --etcd-keyfile=/var/lib/kubernetes/apiserver-etcd-client.key \\
  --etcd-servers=https://127.0.0.1:2379 \\
  --event-ttl=1h \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.crt \\
  --tls-cert-file=/var/lib/kubernetes/apiserver.crt \\
  --tls-private-key-file=/var/lib/kubernetes/apiserver.key \\
  --v=2
```