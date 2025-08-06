# Lecture 49 - Kubectl Apply Command

## Kubectl Apply

- 쿠버네티스 객체를 선언적으로 관리하기 위해 kubectl apply를 사용하는 것은 일반적인 관행.

```bash

$ vi nginx.yaml

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end-service
spec:
  containers:
    - name: nginx-container
      image: nginx:1.18

# 단일 파일 apply
$ kubectl apply -f nginx.yaml

# 디렉토리 내의 모든 구성 파일 apply
$ kubectl apply -f /path/to/config-files

```

## Kubectl Apply 동자 방식

- kubectl apply 는 선언적(declarative) 접근법의 핵심 명령어 이다.
- 원하는 상태(Desired State) 를 이전의 상태와 비교하여 불필요한 변경 사항만 적용한다.
- 이 과정은 3-Way-merge (3 방향 병합) 을 통해 이뤄진다.

### 3가지 소스 비교

- 로컬 구성 파일
    - 사용자가 변경하여 `kubectl apply -f ` 로 지정한 파일
- 라이브(Live) 객체 구성
    - 쿠버네티스 클러스터에 현재 실행 중인 객체의 실제 상태.
-

- 객체가 존재하지 않으면 쿠버네티스는 로컬 구성을 기반으로 객체를 생성
- 객체 생성 과정에서 쿠버네티스는 내부적으로 객체의 상태를 모니터링하기 위한 추가 필드를 추가
- YAML 구성은 JSON으로 변환되어 주석에 "마지막으로 적용된 구성"으로 저장
- 이 정보는 후속 업데이트에서 차이점을 식별하는 데 사용
- 로컬 구성이 변경되면(예: 이미지 버전 업데이트) kubectl apply는 로컬 파일, 라이브 구성 및 마지막으로 적용된 구성을 사용하여 3방향 병합을 수행

### 변경 사항 적용 과정.

- 초기 생성
    - 설정이 존재하지 않으면, 로컬 구성 파일을 기반으로 생성.
    - 생성된 구성에 로컬 구성 파일의 내용이 JSON 형태로 주석에 저장.
- 업데이트
    - 로컬 구성 파일의 내용이 변경되면 (ex: nginx:1.18 -> nginx:1.19), kubectl apply 는 로컬 파일, 라이브 구성, 그리고 주석에 저장된 이전 구성을 비교.
    - 3가지 비교를 통해 변경된 필드를 정확히 식별하고, 라이브 구성의 해당 필드만 업데이트.
- 필드 제거
    - 로컬 구성 파일에서 특정 필드를 제거하면, 주석에 있던 필드가 로컬 파일에 없다는 것을 감지.
    - 이전 구성에는 있었지만 현재 구성에 없는 필드이므로, 라이브 구성에서 해당 필드를 삭제.

### 주의점.

- `kubectl create` 나 `kubectl replace` 와 같은 명령어는 Last Applied Configuare 가 남지 않는다.
- 따라서, `kubectl apply` 와 혼용해서 사용 시 주석이 누락되어 예상치 못한 결과나 오류가 발생할 수 있다.
- 선언적 접근법을 사용하기로 했다면, 모든 변경 사항은 `kubectl apply` 만을 사용해서 관리하는 것이 가장 중요하다.



