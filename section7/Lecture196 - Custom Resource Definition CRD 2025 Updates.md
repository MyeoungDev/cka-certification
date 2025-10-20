# Lecture 196 - Custom Resource Definition CRD 2025 Updates

## 쿠버네티스 리소스 및 컨트롤러

- 쿠버네티스에서 Deployment 를 생성하면 API Server 는 해당 구성을 etcd 에 저장한다.

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      type: front-end
  template:
    metadata:
      name: myapp-pod
      labels:
        type: front-end
    spec:
      containers:
      - image: nginx
```

```bash
$ kubectl create -f deployment.yml
$ kubectl get deployments

# Output:
# NAME                DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE

$ kubectl delete -f deployment.yml

# Output: deployment "myapp-deployment" deleted
```

- Deployment 가 생성되면 Replica 수에 따라 Pod 가 생성된다.
- 이는 Deployment Controller 에 의해 동작한다.
- 클러스터 리소스를 모니터링 하여 실제 상태가 메니페스트의 상태와 일치하는지 관리된다.
- 내부적으로 Deployment Controller 는 ReplicaSet 을 생성하고, ReplicaSet 은 Pod 생성을 관리한다.

## Custom Resources: The Flight Ticket Example

- 사용자 지정 리소스 `kind: FlightTicket` 유형의 객체를 사용한다고 가정.
- 사용자 정의 리소스 (CRD) 를 통해 쿠버네티스에 명시적으로 등록하지 않을 경우 오류 발생.

```bash
apiVersion: flights.com/v1
kind: FlightTicket
metadata:
  name: my-flight-ticket
spec:
  from: Mumbai
  to: London
  number: 2
```

```bash
$ kubectl create -f flightticket.yml

# Output: no matches for kind "FlightTicket" in version "flights.com/v1"
```

## Defining a Custom Resource with CRD

- CRD 는 쿠버네티스에 새로운 사용자 지정 리소스에 대한 정보를 제공하여 리소스 생성 및 관리를 지원한다.
- `apiVersion` 은  `apiextensions.k8s.io/v1` 을 사용한다
- `kind` 는 `CustomResourceDefinition` 을 사용한다.
- `spec` 를 통해 스펙 정의 scope 및 group 을 지정할 수 있다.
- 

```bash
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: flighttickets.flights.com
spec:
  scope: Namespaced
  group: flights.com
  names:
    kind: FlightTicket
    singular: flightticket
    plural: flighttickets
    shortNames:
      - ft
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                from:
                  type: string
                to:
                  type: string
                number:
                  type: integer
                  minimum: 1
                  
```

```bash
$ kubectl create -f flightticket-custom-definition.yml

# Output: customresourcedefinition "flighttickets.flights.com" created
```

```bash
$ kubectl create -f flightticket.yml
$ kubectl get flightticket

# Output:
# NAME             STATUS

$ kubectl delete -f flightticket.yml

# Output: flightticket "my-flight-ticket" deleted
```

```bash
$ kubectl api-resources

# Relevant output:
# NAME             SHORTNAMES   APIGROUP     NAMESPACED   KIND
# flighttickets    ft           flights.com  true         FlightTicket
```

## Beyond Data Storage: Custom Controlles

- CRD 는 주로 etcd 에 데이터를 저장하지만, 외부 서비를 통한 작업의 경우 실제 자동화 되어야 되는 경우가 많다.
- 해당 부분에서 사용자 지정 컨트롤러가 중요한 역할을 한다.
- 일반적으로 `Go` 로 작성된 커스텀 컨트롤러는 리소스 변경 사항을 감시하고, 리소스 변경사항에 대한 적절한 트리거가 진행된다.
- 이를 활용하여 비즈니스 요구사항에 맞게 리소스를 관리할 수 있다.