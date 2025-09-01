# Lecture 121 - Multi Container Pods

## Multi Container Pod

- Multi Container Pod 는 동일한 생명 주기를 공유하는 컨테이너들을 그룹화 되도록 설계되었다.
- 컨테이너들이 함께 생성 및 종료, 공통 네트워크 네임스페이스 공유하여 localhost 를 통한 통신, 공유 스토리지 볼륨 접근 등의 이점이 존재.
- 개별 파드 간의 볼륨 공유 및 네트워킹의 복잡성을 제거하여 구성을 간소화.
- `containers` 배열 필드를 통해 여러개의 컨테이너를 정의해서 사용.

```bash
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
    - name: log-agent
      image: log-agent

```