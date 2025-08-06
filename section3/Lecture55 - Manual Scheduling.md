# Lecture 55 - Manual Scheduling

## 기본 스케쥴러 동작

- 일반적인 Pod anifest 에는 `nodeName` 이라는 필드가 비어있다.
- 쿠버네티스의 기본 스케줄러는 이 `nodeName` 필드가 없는 파드를 감지하고, 적절한 노드를 선택한 후 `nodeName` 필드를 업데이트하며 바인딩 객체를 생성한다.
- 이 과정을 통해 파드가 특정 노드에 할당된다.

## 노드 이름 수동 설정

- Pod 생성 시저에 특정 노드에 할당하려면, 파드 메니페스트의 `spec` 섹션에 `nodeName` 필드를 직접 추가해야 한다.
- `nodeName`은 파드 생성 시에만 설정할 수 있으며, 파드가 실행된 후에는 수정할 수 없다.

```bash
$ vi node-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  nodeName: node02  # <- 여기에 노드 이름 지정.
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 8080
    
    
$ kubectl get pods

NAME      READY   STATUS    RESTARTS   AGE   IP         NODE
nginx     1/1     Running   0          9s    10.40.0.4  node02
```

## 바인딩 객체를 이용한 실행 중인 Pod 재할당

- Pod 가 이미 실행 중이고 노드 할당을 변경해야 하는 경우, `nodeName` 을 직접 수정할 수 없다.
- 이 경우, 스케줄러의 동작을 모방하는 바인딩 객체를 생성할 수 있다.

```bash
$ vi binding.yaml

apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02
  
  
$ yq -o json . binding.yaml > binding.json

$ curl --header "Content-Type: application/json" --request POST --data @binding.json http://$SERVER/api/v1/namespaces/default/pods/nginx/binding

```

### 개인 궁금증

> `nodeName` 을 수동으로 설정하면 어떤게 좋은가?
>
> - `nodeName` 을 설정해서 파드를 특정 노드에 직접 스케쥴링하는 것은 일반적인 상황에서 권장되지 않는다.
> - 쿠버네티스의 스케줄러는 노드의 자원 상태, 테인드(taint), 톨러레이션(toleration), 노드 어피니티(node affinity) 등 복잡한 요소를 고려하여 가장 적합한 노드를 찾아주기 때문이다.


> 그렇다면 `nodeName` 설정의 장점은?
> - 스케줄러 무시하고 즉각적인 배치
> - `nodeName` 을 설정하면 쿠버네티스 스케줄러를 건너뛰고 바로 지정된 노드에 파드를 배치함으로써, 특정 노드에 즉시 올리고 싶을때 유용.
>- 특정 노드에 강력한 제어가 필요한 경우
>  - 특정 하드웨어에 종속성이 필요하여, 특정 노드에 배포해야할 경우.
>  - 문제가 발생한 특정 노드의 디버깅 용도.
>

> `nodeName` 에 지정한 노드가 장애가 발생하거나 없을 경우의 동작은?
>
> - 지정한 노드 (`nodeName`) 이 존재하지 않거나, 리소스가 부족할 경우 파드는 영원히 `Pending` 상태로 대기.
> - 기본 스케쥴러의 경우 다른 노드를 찾아서 실행하겠지만, `nodeName` 이 지정된 경우 이러한 동작이 진행되질 않음.
> - 지정된 노드가 다운되면 해당 파드는 복구되지 않는다.


> 그렇다면 `nodeName` 대신 더 권장되는 노드 지정 방법은?
> - `Node Affinity`
>    - 특정 노드에 선호도(Affinity) 를 부여하는 방식.
>    - 파드를 특정 노드에 까갑게 배치하되, 만약 해당 노드를 사용할 수 없으면 다른 노드에 배치되도록 한다.
> - `Taints & Tolerations`
>    - 특정 노드에 테인트(오염)을 부여하고, 파드에 톨러레이션(관용) 을 설정하여, 특정 파드만 해당 노드에서 실행될 수 있도록 허용.
>    - 예) GPU 노드에 `gpu=true:NoSchedule` 테인트 설정, GPU 사용하는 파드에 해당 톨러레이션 추가.
