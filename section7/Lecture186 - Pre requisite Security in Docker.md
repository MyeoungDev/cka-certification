# Lecture 186 - Pre requisite Security in Docker

## 컨테이너 Process 동작

- Virtual Machine 과 달리 Container 는 호스트의 Linux 커널을 공유하고 네임스페이스를 통해 격리된다. 
- 리눅스에서 네임스페이스(namespace)는 커널 기능으로, 시스템 리소스를 격리해 프로세스 그룹별로 독립적인 환경을 제공하는 기술 
- 네임스페이스 내의 프로세스는 자신에게 할당된 리소스만 보고 조작할 수 있어, 여러 프로세스가 하나의 호스트에서 실행되면서도 서로 영향을 주지 않고 독립적으로 동작
- 각 컨테이너는 전용 네임스페이스 내에서 동작하며, 다른 컨테이너에게 자신의 프로세스를 노출시키기 않는다.
- Docker는 네임스페이스를 사용하여 프로세스 격리를 유지한다.- 컨테이너 프로세스는 자체 네임스페이스 내에서 `PID 1`로 시작하지만, 호스트 시스템에서는 다른 PID를 받는다.

## 사용자 권한 및 보안

- Docker 는 기본적으로 컨테이너 프로세스를 루트 사용자로 시작한다.
- 컨테이너는 기본적으로 루트 사용자로 실행되지만, Docker는 Linux 기능을 사용하여 컨테이너의 루트 권한을 제한한다.
- 이를 통해 컨테이너 루트가 호스트 루트와 동일한 수준의 제어 권한을 갖지 않도록 할 수 있다.
- 따라서, 보안을 강화하기 위해서는 컨테이너 프로세스를 루트 사용자로 실행하지 않는 것이 좋다.
- `--user` 옵션을 사용하여 런타임에 사용자를 변경할 수 있다.
- `docker run --user=1000 ubuntu sleep 3600`

```bash
ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
1000         1  0.0  0.0  4528  828 ?        Ss   03:06   0:00 sleep 3600
```

- 추가로, Docker 이미지에 사용자를 지정하는 것을 보안을 강화시킨다.
```bash
$ vi Dockerfile
FROM ubuntu
USER 1000

$ docker build -t my-ubuntu-image .
$ docker run my-ubuntu-image sleep 3600
```

## Linux 기능 관리
- Docker 는 Linux 기능을 활용하여 컨테이너 루트 사용자의 권한을 제한한다.
- 모든 작업을 수행할 수 있는 시스템의 루트 사용자와 달리, 컨테이너 루트는 기본 권한 집합으로 제한된다.
- 이를 통해 컨테이너가 호스트 시스템이나 다른 컨테이너를 손상시킬 수 있는 작업을 숭행하는 것을 방지할 수 있다.
- 만약, 추가 권한이 필요한 경우 `--cap-add` 옵션을 통해 권한을 부여할 수 있다.
- 반대로 권한을 제거하기 위해서는 `--cap-drop` 옵션을 사용한다.
- 모든 권하으로 실행되어야 되는 컨테이너의 경우 `--privileged` 옵션을 사용할 수 있지만 주의가 필요하다.

```bash
$ docker run --cap-add=MAC_ADMIN ubuntu
$ docker run --cap-drop=MAC_ADMIN ubuntu
$ docker run --privileged ubuntu
```