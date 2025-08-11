# Lecture 69 - Resource Limits

- 쿠버네티스 스케줄러는 요청된 리소스를 각 노드가 제공할 수 있는 리소스와 비교하여 어떤 노드가 포드를 호스팅할지 결정.
- 기본적으로 쿠버네티스는 CPU 또는 메모리 요청 및 제한을 적용하지 않는다.
- 즉, 제한이 지정되지 않은 파드는 노드에서 사용 가능한 모든 리소스를 소비하여 다른 파드와 시스템 프로세스에 영향을 미칠 수 있다.
- 만약, 리소스 요구사항에 충족하는 Node 가 없을 경우 Pending 상태로 대기하며, 아래와 같은 에러 메세지가 난다.

```bash
NAME              READY   STATUS    RESTARTS   AGE
Nginx             0/1     Pending   0          7m
Events:
  Reason           Message
  ------           -------
  FailedScheduling  No nodes are available that match all of the following predicates:: Insufficient cpu (3).

```

## Resource Request

- Resource Request 는 컨테이너가 스케줄링될 때 보장되는 최소 CPU 및 메모리를 정의.
- 예를 들어, 파드 정의에 1 CPU, 1 MEM 를 지정할 경우 최소 리소스를 제공할 수 있는 노드에만 스케쥴링 된다.
- 쿠버네티스 스케줄러는 이러한 요구 사항을 충족하는 노드를 검색하여 파드가 선언된 리소스에 액세스할 수 있도록 보장한다.

```bash
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
    - containerPort: 8080
    resources:
      requests:
        memory: "4Gi"
        cpu: 2

```

## Resource Limits

- 기본적으로 포드 내 컨테이너는 제한이 설정되지 않은 경우 노드에서 사용 가능한 모든 리소스를 사용할 수 있다.
- 하지만, Resource Limit 을 정의하여 사용량을 제한할 수 있다.
- 컨테이너가 CPU 제한을 초과하면, 쿠버네티스는 CPU 사용량을 제한한다.
- 컨테이너가 허용된 메모리보다 많은 메모리를 사용하면 파드를 정료하고 OOM 으로 기록한다.

## LimitRange

- 네임스페이스 내의 모든 파드에 대해 기본 리소스 요청 및 제한 값을 설정하여, 리소스 관리를 강제.

```bash

# limit-range-cpu.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
    - default:
        cpu: 500m
        memory: 1Gi
      defaultRequest:
        cpu: 500m
        memory: 1Gi
      max:
        cpu: "1"
      min:
        cpu: 100m
        memory: 500Mi
      type: Container

```