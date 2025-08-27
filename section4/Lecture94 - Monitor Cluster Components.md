# Lecture 94 - Monitor Cluster Components

## 모니터링 요소

- 쿠버네티스트 클러스틀 효과적으로 모니터링 하기 위해서는 Node 와 Pod 수준 모두에서 메트릭을 추적해야 한다.
- 쿠버네티스에는 포괄적인 기본 모니터링 솔루션이 포함되어 있지 않으므로 외부 도구를 구현해야 한다.
- 인기 있는 오픈소스 모니터링 솔루션으로는 Metrics Server, Prometheus, Elastic Stack 등이 있다.

### Node

- 클러스터의 총 노드 수
- 각 노드의 상태
- CPU, 메모리, 네트워크, 디스크 활용도와 같은 성능 측정 항목

### Pod

- 실행 중인 포드의 수
- 모든 포드의 CPU 및 메모리 소비량

## Heapster 와 Metics Server

- Heapster는 전통적으로 쿠버네티스에 대한 모니터링 및 분석 지원을 제공 도구.
- 그러나, 더 이상 지원되지 않고 후속 버전인 Metrics Server 가 쿠버네티스 모니터링 표준.
- Metrics Server는 쿠버네티스 클러스터당 한 번씩 배포되도록 설계
- 노드와 파드에서 메트릭을 수집하고, 데이터를 집계하여 메모리에 보관
- 메모리에만 데이터를 저장하므로 과거 성능 데이터는 지원하지 않는다.
- Metrics Server는 단기 모니터링 및 빠른 인사이트 확보에 적합하지만, 장기적인 과거 데이터 분석에는 적합하지 않는다.
- 장기적인 메트릭을 위해서는 고급 모니터링 솔루션( Prometheus 또는 Elastic Stack) 을 통합하는 것을 고려해야 한다.

## 메트릭 수집 방법

- 모든 쿠버네티스 노드는 쿠버네티스 API 서버와 통신하고 파드 작업을 관리하는 Kubelet이라는 서비스를 실행
- Kubelet 내부에는 cAdvisor(Container Advisor)라는 통합 구성 요소가 실행 중인 파드에서 성능 지표를 수집
- 러한 지표는 Kubelet API를 통해 노출되고 메트릭 서버(Metrics Server)에서 검색

## Metrics Server 사용

```bash

$ git clone <https://github.com/kubernetes-incubator/metrics-server.git>

$ kubectl create -f deploy/1.8+/

...

$ kubectl top node

NAME         CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
kubemaster   166m         8%     1337Mi          70%
kubeno 1     36m          1%     1046Mi          55%
kubeno 2     39m          1%     1048Mi          55%

$ kubectl top pod

NAME   CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
nginx  166m         8%     1337Mi          70%
redis  36m          1%     1046Mi          55%

```