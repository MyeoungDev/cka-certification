# Lecture 58 - Labels and Selectors

## Labels and Selectors

### Labels

- 라벨은 쿠버네티스 리소스(Pod, Service, ReplicaSet 등) 에 붙이는 Key-Value 태그이다.
- 리소스를 식별하고 그룹화하는데 사용한다.
- 사용자가 원하는 대로 자유롭게 정의가 가능하다.
- 하나의 리소스에 여러 개의 라벨의 정의가 가능하다.

### Selectors

- 라벨을 사용해서 특정 조건을 만족하는 리소스를 찾아내는 도구.
- `ReplicaSet` 이나 `Service` 같은 상위 컨트롤러가 자신이 관리할 파드를 셀레겉를 통해 찾는데 사용.
- 데이터베이스에서 쿼리를 사용하는 것과 유사.ㅁ
- `app: my-web-app` 처럼 라벨을 가진 모든 파드를 한 번에 선택 가능.

## ReplicaSets 에서 라벨과 셀렉터 사용.

- 라벨은 리소스에 "이름표"를 붙이는 것이고, 셀렉터는 이름표를 통해 "찾아내는" 과정이다.
- 클러스터 내 객체를 효과적으로 관리하고 그룹화 하는데 필수적이다.
- Pod 선택에 대해 더욱 세부적인 제어가 필요한 경우 `matchLabels` 섹션에 여러 개의 레이블을 나열할 수 있다.


```bash
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        function: Front-end
    spec:
      containers:
      - name: simple-webapp
        image: simple-webapp
```

## Annotation

- 애노테이션은 선택 대상이 아닌 추가 메타데이터를 저장하는 데 사용된다.
- 해당 메타데이터에는 도구 버전, 빌드 정보와 같은 세부 정보가 포함될 수 있다.

```bash

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
  annotations:
    buildversion: "1.34"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        function: Front-end
    spec:
      containers:
      - name: simple-webapp
        image: simple-webapp
        
```
