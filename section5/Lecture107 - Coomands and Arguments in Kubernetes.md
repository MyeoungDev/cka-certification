# Lecture 107 - Commands and Arguments in Kubernetes

## Overriding Arguments

```bash

$ vi Dockerfile

FROM Ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]

```

- `ENTRYPOINT` 명령어는 컨테이너 가 시작될 때 실행할 명령을 지정한다..
- `CMD` 명령어 는 해당 명령에 대한 기본 매개변수를 제공한다.
- 위와 같은 이미지를 쿠버네티스 Pod 로 정의할 경우 인수 재정의를 하기 위해서는 `spec.conateinrs.args` 속성을 사용하면 된다.
- 이는 `docker run --name ubuntu-sleeper ubuntu-sleeper 10` 과 동일한 동작을 나타낸다.

```bash

$ vi ubuntu-sleeper-pod.yaml

apiVersion: v1
kind: Pod
metadata:
name: ubuntu-sleeper-pod
spec:
    containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      args: ["10"]

```

## Overriding ENTRYPOINT

- Container 의 `ENTRYPOINT` 를 Pod 정의시 재정의 할 경우에는 `spec.containers.command` 속성을 사용하면 된다.
- Docker 명령어 에서 `-entrypoint` 명령어와 동일한 역할을 한다.

```bash

apiVersion: v1
kind: Pod
metadata:
name: ubuntu-sleeper-pod
spec:
    containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep2.0"]
      args: ["10"]

```