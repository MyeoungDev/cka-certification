# Lecture 65 - Node Affinity


## Node Affinity

- Node Affinity 는 `In`, `NotIn`, `Exists` 와 같은 고급 표현식을 사용해서 기본 Node Selector 의 기능을 확장할 수 있다.
- 해당 기능을 통해 Node Labels 기반으로 파드 배치에 대한 세부 규치을 지정할 수 있다.
- Node Affinity 를 사용해서 파드를 스케쥴링하면, 해당 규칙은 스케줄링 중에만 평가된다.
- 스케줄링 이후 노드 레이블을 변경하더라도 `ignored during execution` 동작으로 실행 중인 파드에는 영향을 미치지 않는다.

### Node Affinity 의 두 가지 주요 스케줄링 동작

**Required During Scheduling, Ignored During Execution**

- 포드는 Affinity 규칙을 완전히 충족하는 노드에만 예약된다.
- 실행이 완료되면 노드 레이블이 변경되어도 Pod에 영향을 미치지 않는다.

**Preferred During Scheduling, Ignored During Execution**

- 스케줄러는 친화성 규칙을 충족하는 노드를 선호하지만, 일치하는 노드가 없는 경우 다른 노드에 포드를 배치한다.

### 기본 Node Selector 구성

```bash

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
```

## Node Affinity 사용 구성

- `affinity` 는 `spec` 아래에 정의된다.
- `requiredDuringSchedulingIgnoredDuringExecution` 는 스케줄러가 기준을 충족하는 노드에 파드를 배치해야 된다는 옵션.
    - 파드가 실행되면 노드의 레이블 변경 사항은 무시된다.
- `nodeSelectorTerms` 배열에는 하나 이상의 `matchExpression` 이 포함된다.
    - 각 표현식은 `key`, `operator`, `values` 을 지정한다.
-

### `In` Operation 구성

```bash

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: data-processor
      image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: In
                values:
                  - Large

```

### `NotIn` Operator 구성


```bash

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: data-processor
      image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: NotIn
                values:
                  - Small
                  
```

### `Exists` Operator 구성

```bash

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: Exists
            
```