# Lecture 199 - Operator Framework 2025 Updates

- Operator Framework 는 쿠버네티스에서 애플리케이션 배포 및 관리를 간소화 하는데 도움을 준다.
- 기존에 CRD 와 Custom Controller 를 별도로 생성해서 Pod 또는 Deployment 로 배포하던 것을 Operator Framework 를 사용하여 모든 구성 요소를 함께 패키징하여 단일 엔티티로 배포한다.
- Operator Framework 는 단순히 CRD 와 Controller 를 배포하는 것 이상으 기능을 제공한다.
- 대표적으로 `etcd Operator` 가 있다.
    - etcd 클러스터를 정의하는 CRD
    - etcd 배포를 모니터링하고 관리하는 맞춤형 컨트롤러
- Operator Hub 에서 `etcd`, `MySQL`, `Prometheus`, `Grafana`, `Argo CD`, `Istio` 등 인기 애플리케이션이 있다.
- 아래와 같이 Operator Framework 를 사용하기 위해 `Operator Lifecycle Manager` 를 설치하여 사용한다.

```bash

$ curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.19.1/install.sh | bash -s v0.19.1

$ kubectl create -f https://operatorhub.io/install/etcd.yaml

$ kubectl get csv -n my-etcd
```
