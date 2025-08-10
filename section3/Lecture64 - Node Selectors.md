# Lecture 64 - Node Selectors

## Node Selector

- 특정 파드가 클러스터 내 지정된 노드에서만 실행되도록 하는 방법.
- Node Selector 는 노드의 기본 하드웨어 특성에 맞춰 파드 배포를 조정하여 성능과 리소스 관리를 향상시킨다.
- Node Selector 는 Pod spec 에 정의된 Key-Value 쌍을 노드의 레이블과 일치시켜서 Pod 배치를 제한한다.
- 서로 다른 리소스를 가진 클러스터 구조에서, 더 많은 성능을 요구하는 파드를 리소스 가용량이 더 많은 노드에 배치할 때 사용한다.
- 간단한 시나리오에서는 사용하기 적합하지만, 요구사항이 복잡해질 경우 Node Selector 를 사용하기 보다는 Node Affinity 또는 Node Anti-Affinity 를 사용하는 것이 좋다.

```bash

# size:Large 대상 노드 레이블 지정.

$ vi node-selector-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  nodeSelector:
    size: Large


$ kubectl label nodes node-1 size=Large

$ kubectl create -f pod-definition.yaml

```