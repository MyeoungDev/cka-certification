# Lecture 61 Taints and Tolerations

## Taints 와 Tolerations

### Taints

- 노드에 적용되어 특정 파드가 해당 노드에 스케줄링되지 않도록 거부.
- 노드를 배타적인 공간으로 만드는 것.
- 이 노드는 이런 속성이 있어. 이런 속성을 용인하지 않는 파드는 여기 오지 마!' 라고 선언하는 것과 같다.
- 톨러레이션이 없는 일반 파드들은 해당 노드에 스케줄링되지 않는다.

### Taints 종류

- `NoSchedule`: Taints 를 허용하지 않는 Pod 는 노드에 Scheduling 불가.
- `PreferNoSchedule`: Taints 를 허용하지 않는 Pod 를 노드에 Scheduling 할 수 없으나, 노드의 자원 부족인나 특정 상황에 따라서 배포될 수 있다.
- `NoExecute`: Pod 에 대해서 Toleration 설정이 없으면, Scheduling 되지 않고, 기존에 실행되던 Pod도 삭제된다.

### Tolerations

- 파드에 적용되어 테인트된 노드에서도 실행될 수 있도록 허용하는 것.
- 특정 파드에게 그 공간에 들어갈 수 있는 통행증을 주는 것.
- 나는 노드에 적용된 그 속성을 용인할 수 있어. 나는 그 노드로 갈 수 있어!' 라고 선언하는 것과 같다.
- 파드에 톨러레이션을 적용하면, 해당 파드는 테인트가 적용된 노드에 스케줄링될 수 있는 자격을 얻게 된다.


## Taints 와 Tolerations 동작 방식

- 파드가 노드에서 실행될 수 있는지 여부는 두 가지 요소에 따라 결정된다.
    - 노드의 적용된 Taints
    - 파드의 Tolerations 할 수 있는 Taints 정도
- 쿠버네티스에서 테인트와 톨러레이션은 보안이 아닌 스케줄링 제어에만 사용된다.

### 상황

세 개의 워커 노드(노드 1, 2, 3)와 네 개의 파드(A, B, C, D)로 구성된 간단한 클러스터

```bash
# node1 에 app=blue 테인트 부여
$ kubectl taint nodes node1 app=blue:NoSchedule

# Pod D 에 tolerations 부여. app=blue:NoSchedule
$ vi pod-d.yaml

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
  tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
      
      
```
- Pod A: 톨러레이션이 없으므로, app=blue 테인트가 있는 노드 1에는 스케줄링되지 않는다. 노드 2나 노드 3으로 배정된다.
- Pod B: 이미 노드 1에서 실행 중인 상태에서 NoExecute와 같은 효과를 가진 테인트가 적용되면, 노드 1에서 쫓겨나 다른 노드로 재배정 된다.
- Pod C: 톨러레이션이 없으므로, 노드 1에 스케줄링되지 않는다.
- Pod D: app=blue에 대한 톨러레이션이 있으므로, 노드 1의 테인트를 받아들일 수 있다. 스케줄러는 이 파드를 전용 노드인 노드 1에 배정할 수 있게 된다.

> **NOTE**
> 테인트와 톨러레이션은 특정 워크로드만 실행해야 하는 노드가 포드가 실수로 예약되는 것을 방지하는데 사용되지만,
> 포드가 특정 노드에서 실행된다는 것을 보장하지는 않는다.
> 노드별 스케줄리의 경우 노드 어피니티(Affinity)를 사용하는 것이 좋다.

## Master Node Taint

- 쿠버네티스는 기본적으로 Master Node 에 Taint 를 적용하여 워크로드 파드가 마스터 노드에 예약되는 것을 방지한다.
- 이 설정은 마스터 노드가 중요한 시스템 구성 요소만 예약되도록 보장한다.
- 이 구성은 마스터 노드에서 필수 구성 요소만 실행되도록 보장하는 모범 사례다.

```bash

$ kubectl describe node kubemaster | grep Taint

Taints:         node-role.kubernetes.io/master:NoSchedule

```