# Lecture 106 - Commands and Arguments in Docker

Docker의 명령과 인수

이 세션에서는 컨테이너가 프로세스를 실행하는 방식과 이러한 개념이 나중에 Kubernetes에서 Pod 정의로 어떻게 변환되는지 자세히 알아보겠습니다.
이러한 주제는 자격증 과정에서 간과되는 경우가 많지만, 컨테이너화를 완벽하게 이해하려면 반드시 이해해야 합니다.

## 컨테이너 이해하기

- 컨테이너의 프로세스 실행 방식과 개념은 Pod 컨테이너화 과정을 완벽하게 이해하려면 숙지하는 것이 좋다.

```bash

$ docker run ubuntu

$ docker ps -a

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS
45aacca36850        ubuntu              "/bin/bash"         43 seconds ago      Exited (0) 41 seconds ago

```

- `ubuntu` 컨테이너를 실행했지만, 동시에 종료되었다.
- 컨테이너는 단일 작업이나 프로세스를 실행하도록 최적화 되어 있다.
- 컨테이너는 작업이 완료되면 종료된다.
- 컨테이너의 생명 주기가 컨테이너 내부에서 실행 중인 프로세스와 직접 연결되어 있기 때문이다.

## Dockerfile 정의 및 Runtime ENTRYPOINT

```bash

$ vi Dockerfile

FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]

```

- `ENTRYPOINT` 는 컨테이너가 시작될 때 항상 실행되는 명령어를 정의하는 부분이다.
- `CMD` 는 사용하면 런타임 인수가 기본 명령을 대체합니다.
- `ENTRYPOINT`를 사용하면 런타임 인수가 지정된 실행 파일에 추가되어 매개변수만 재정의할 수 있다.
    - `docker run ubuntu-sleeper 10`
    - `docker run --entrypoint sleep2.0 ubuntu-sleeper 10`
- 이러한 컨테이너의 기본 개념을 숙지해야 쿠버네티스에서 이러한 개념들이 어떻게 Pod 정의로 변환되는지 이해할 수 있다.