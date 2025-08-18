# Lecture 79 - Priority Class

## Priority Class

- 쿠버네티스에서 워크로드 스케줄링을 관리하려면 우선순위 클래스를 이해하는 것이 필수적이다.
- 쿠버네티스는 다양한 애플리케이션을 중요도가 서로 다른 파드로 실행한다.
- Control Plane 구성 요소는 클러스터 내에서 파드로 실행되며 클러스터 운영에 필수적이다.
- 쿠버네티스는 더 중요한 워크로드가 덜 중요한 워크로드보다 먼저 스케줄링될 수 있도록 우선순위 클래스를 사용한다.
- 우선순의 클래스를 사용하면 파드에 숫자 값을 할당할 수 있으며, 숫자가 높을수록 우선수위가 높다.
- 사용자 배포 애플리케이션의 경우, 이 값은 약 -20 ~ +10억 까지 이다.
- 쿠버네티스 제어 영여과 같이 시스템의 중요한 구성 파드는 최대 20억까지 값을 지정할 수 있는 예약된 범위가 있다.

```bash

$ vi kubectl get priorityclass

NAME                      VALUE        GLOBAL-DEFAULT   AGE
system-cluster-critical   2000000000   false            157d
system-node-critical      2000001000   false            157d

```

## Priority Class 생성

```bash

$ vi priority-class.yaml

apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000000
description: "Priority class for mission critical pods"
globalDefault: true

$ vi pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
      - containerPort: 8080
priorityClassName: high-priority

```

- PriorityClass 를 생성한 후 Pod 에서 `priorityClassName` 속성을 사용해 우선순위 클래스를 할당할 수 있다.
- 우선순위 클래스를 지정하지 않으면 Pod 는 기본 우선순위 값인 0이 할당된다.
- 만약, 파드의 기본 우선순위를 변경하려면, `globalDefault` 속성을 통해 사용해야 한다.
- 단, 하나의 우선수위 클래스만 전역 기본값으로 지정할 수 있다.

## Pod 의 우선 순위 선점 방식

- 새로운 파드가 기존의 우선순위가 낮은 파드를 선점 방식은 우선순위 클래스에 정의된 선점 정책에 따라 달라진다.
- PreemptLowerPriority
    - 우선순위가 낮은 파드를 축출하여 우선순위가 높은 파드에 리소스 할당.
    - 쿠버네티스 기본 우선순위 정책

```bash
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
 name: high-priority
value: 1000000000
description: "Priority class for mission critical pods"
preemptionPolicy: PreemptLowerPriority

```

- Never
    - 우선순위 높은 파드가 기존 파드를 제거하지 않고 스케줄링 대기 상태.

```bash

apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000000
description: "Priority class for mission critical pods"
preemptionPolicy: Never

```