# Lecture 191 - Network Policy

네트워크 트래픽에는 두가지 주요 유형이 있다.

- Ingress Traffic
    - 사용자 요청.
    - 서버로 들어오는 트래픽
    - 클러스터 밖 -> 안
- Egress Traffic
    - 서버의 요청.
    - 서버에서 서버로 전송되는 트래픽
    - 클러스터 안 -> 밖

### 네트워크 흐름 예시

웹 서버 → API 서버 → DB 서버로 이어지는 통신 흐름을 가정

- 웹 서버 → API 서버
    - 사용자는 웹 서버의 80번(HTTP)으로 들어온다 (웹 서버 ingress 80).
    - 웹 서버는 API 서버의 5000번으로 요청을 보낸다 (웹 서버 egress→API:5000, API 서버 ingress 5000).
- API 서버 → DB 서버
    - API 서버는 DB의 3306번(MySQL 등)으로 요청을 보낸다(API 서버 egress→DB:3306, DB 서버 ingress 3306).

## 쿠버네티스의 네트워크 보안

- 쿠버네티스 클러스터에서 노드는 각각 고유한 IP 주소가 할당된 파드와 서비스를 호출한다.
- 쿠버네티스의 중요한 기능 중 하나는 사용자 지정 경로 설정과 같은 추가 구성 없이도 파드가 서로 통신할 수 있다는 것이다.
- 일반적으로 모든 파드는 전체 클러스터에 걸쳐 있는 가상 사설망(VPN) 에 상주하여 파드 IP, 파드 이름 또는 구성된 서비스를 사용하여 상호 작용할 수 있다.
- 기본적으로 Kubernetes는 `all-allow` 규칙을 사용하여 모든 Pod가 클러스터 내의 다른 모든 Pod 또는 서비스와 통신할 수 있도록 허용한다.

### 네트워크 정책에 따른 통신 제한

- 보안 요구 사항에 따라 특성 서버끼리는 직접 통신이 안되어야 하는 경우가 있다. (웹 <-> DB)
- `Network Policy` 를 통해 이를 강제할 수 있다.
- 쿠버네티스의 Network Policy 는 `labels` 와 `selectors` 를 사용하여 하나 이상의 파드에 연결하는 객체로 정의한다.
- Network Policy 를 적요하기 위해서는 Pod 에 `label` 을 지정하고 Network Policy 에 일치하는 `selector` 정의해야 한다.

```bash
$ vi db-policy.yaml

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
    ports:
    - protocol: TCP
      port: 3306

$ kubectl create -f db-policy.yaml

```

- `podSelector` 는 해당 레이블을 통해 Pod 를 대상으로 지정한다.
- `policyTypes` 는 수신 트래픽만 영향 받도록 지정한다.
    - 현재 설정의 경우 `api-pod` 이름의 Pod 트래픽만 허용한다.
- `policyTypes` 이 지정되지 않은 트래픽은 기본적으로 자동 허용된다.