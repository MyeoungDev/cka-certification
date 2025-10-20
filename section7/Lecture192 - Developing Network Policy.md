# Lecture 192 - Developing network policy

- 쿠버네티스는 기본적으로 파드 간의 모든 트래픽을 허용한다.
- 파드간의 통신 제한을 진행하기 위해서는 명시적인 네트워크 정책이 필요하다.

## 데이터베이스 Pod에 대한 모든 수신 트래픽 차단

- `labels`, `podSelector` 를 이용하여 DB Pod 를 타겟하는 Network Policy 생성
- 기본적으로 모든 Ingress 트래픽을 거부하는 정책 정의
- Ingress 규칙이 지정되지 않으면 Ingress Traffic 에 대한 완전한 차단 처리.

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress

```

## API Pod에서 Ingress 트래픽 허용

- API 서버의 경우 DB Pod 포트로 들어오는 Ingress Traffic 을 허용해야 한다.
- 허용된 트래픽에 대한 응답은 자동으로 허용되므로, Ingress 규칙만 정의해도 된다.
    - 즉, Ingress 가 열리면 응답에 대한 Egress 의 경우 정의하지 않아도 된다.

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
      namespaceSelector:
        matchLabels:
          name: prod
    - ipBlock:
        cidr: 192.168.5.10/32
  ports:
  - protocol: TCP
    port: 3306

```

- `podSelector.matchLabels.name: api-pod`
    - `api-pod` 라는 이름을 가진 Pod 로 부터 Ingress 트래픽 허용
- `podSelector.namespaceSelector.matchLabels.name: prod`
    - `prod` 로 지정된 `namespace` 에서만 가능.
- `ipBlock.cdir: 192.168.5.10/32`
    - 192.168.5.10 IP 를 가진 요청에 대한 허용 (외부 백업 서버)

## Egress Traffic Configure

- Ingress, Egress 트래픽을 모두 지원하려면 Egress 를 포함한 `policyTypes` 규칙을 정의해야 한다.

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
      namespaceSelector:
        matchLabels:
          name: prod
  ports:
  - protocol: TCP
    port: 3306
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 80

```