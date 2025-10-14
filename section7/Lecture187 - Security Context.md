# Lecture 187 Security Context

- 쿠버네티스에서 컨테이너는 항상 Pod 에 캡슐화된다.
- Docker 컨테이너 실행 시 특정 사용자 지정, 컨테이너 권한 부여 등에 대한 작업을 쿠버네티스에서 Pod 수준에 유사하게 적용할 수 있다.
- 이는 파드 내의 모든 컨테이너에 영향을 미치거나, 컨테이너 수준에서 정의될 수 있다.
- 파드 수준과 컨테이너 수준이 동일한 보안 구성이 설정된 경우, 컨테이너별 설정이 파드 수준 구성보다 우선된다.

```bash
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
        capabilities:
          add: ["MAC_ADMIN"]
```