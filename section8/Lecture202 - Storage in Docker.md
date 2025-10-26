# Lecture 202 - Storage in Docker

- Docker 를 설치하게 되면 `/var/lib/docker` 와 같은 디렉토리 구조가 생긴다.
    - 해당 디렉토리 하위 에는 `overlay2`, `containers`, `images`, `volumes` 디렉토리가 존재한다.
- 해당 디렉토리에는 Docker Image, Container Runtime Data, Volume 등이 저장된다.
- 컨테이너에 대한 정보들은 위에 정의된 디렉토리에 저장된다.

## Docker 이미지 레이어

- Docker 이미지는 계층화된 아키텍처를 사용하여 빌드된다.
- Dockerfile 의 각 명령은 이전 layer의 수정 사항만 포함하는 새로운 layer 를 생성한다.

```bash
# Dockerfile for Application 1
FROM ubuntu

RUN apt-get update && apt-get -y install python
RUN pip install flask flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run

```

```bash
$ docker build Dockerfile -t mmumshad/my-custom-app

```

### Dockerfile 실행 과정

레이어는 다음 순서로 생성된다.

- 기본 Ubuntu 이미지(약 120MB).
- APT 패키지를 설치하는 레이어(약 300MB).
- Python 패키지 종속성을 위한 레이어.
- 애플리케이션 소스 코드를 추가하는 레이어.
- 진입점을 설정하는 레이어.
- 각 레이어는 이전 레이어의 변경 사항만 저장하므로, Docker는 유사한 이미지에서 재사용할 수 있도록 변경 사항을 캐시한다.
- 예를 들어, 약간 수정된 두 번째 애플리케이션은 다음과 같은 Dockerfile을 사용할 수 있다.

```bash
# Dockerfile2 for Application 2
FROM ubuntu

RUN apt-get update && apt-get -y install python
RUN pip install flask flask-mysql

COPY app2.py /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app2.py flask run

```

```bash
$ docker build Dockerfile2 -t mmumshad/my-custom-app-2

```

- 처음 세 레이어 (기본 base 이밎, APT 패키지, Python 종속성) 은 동일하므로 Docker 는 캐시된 레이어를 재사용하고 새 소스 코드 및 진입점과 관련된 레이어만 빌드한다.
- 이러한 재사용은 빌드 시간을 단축하고 디스크 공간을 절약한다.
- 애플리케이션 코드가 변경되면, Docker 는 변경되지 않은 모든 계층의 캐시를 활용하고 새 코드가 포함된 계층만 다시 빌드한다.

## 컨테이너 쓰기 가능 레이어 및 Copy-On-Write

- 이미지가 빌드되면 해당 레이어는 변경 불가능(Read-Only) 상태로 유지된다.
- 해당 이미지에서 `docker run`명령을 사용하여 컨테이너를 실행하면 읽기 전용 이미지 레아어 위인 최상단에 쓰기 가능한 레이어가 추가된다.
- 이 레이어는 로그 파일, 임시 파일 또는 애플리케이션 변경 사항과 같은 모든 런타임 수정 사항을 캡처한다.
- 컨테이너 내부의 파일을 수정하게 되면, Docker 는 Copy-on-Write 매커니즘을 사용한다.
- 읽기 전용 이미지 레이어에서 생성된 파일을 수정하기 전에 Docker 는 먼저 해당 파일을 쓰기 가능한 레이어에 복사하고, 변경 사항을 복사된 파일에 적용하므로써 원본 이미지는 그대로 유지한다.
- 컨테이너가 제거되면 쓰기 가능한 레이어와 그 안의 모든 변경 사항은 제거된다.

## 볼륨 및 바인드 마운트를 사용한 영구 데이터

- 컨테이너의 쓰기 가능 계층은 임시적이므로 컨테이너가 제거되면 해당 계층에 저장된 모든 데이터가 손실된다.
- Docker는 데이터베이스 등의 데이터를 보관하기 위해 Volume 과 Bind Mount 를 모두 제공한다.

### Volume Mount

- Volume 은 Docker 에서 관리되며 `/var/lib/docker/volumes` 에 저장된다.
- 존재하지 않는 볼륨 이름으로 컨테이너를 실행하면 Docker가 자동으로 해당 볼륨을 생성한다.

```bash
$ docker volume create data_volume
$ docekr run -v data_volume :/var/lib/mysql mysql

$ docker run -v data_volume2:/var/lib/mysql mysql

```

### Bind Mount

- Bind Mount 를 사용하면 Docker 호스트의 특정 디렉토리를 사용할 수 있다.
- `-mount` 플래그는 모든 매개변수를 지정하도록 요구함으로써 더욱 명확한 구문을 제공

```bash
$ docker run -v /data/mysql:/var/lib/mysql mysql

$ docker run \\
  --mount type=bind,source=/data/mysql,target=/var/lib/mysql \\
  mysql

```

## Docker Storage Driver

- Docker의 Storage Driver 는 이미지 레이어 유지 관리부터 COW(Copy-On-Write) 기능을 갖춘 쓰기 가능 컨테이너 레이어 처리까지 모든 것을 관리한다.
- 일반적인 Storage Driver 로는 AUFS, ZFS, BTRFS, Device Mapper, Overlay, Overlay2 등이 있다.
- Storage Driver 선택은 호스트 OS에 따라 달라진다.
    - 예를 들어, Ubuntu는 기본적으로 AUFS를 사용하는 반면, Fedora나 CentOS는 Device Mapper를 선호된다.
- Docker는 성능 및 안정성 요소를 기반으로 시스템에 가장 적합한 드라이버를 자동으로 선택한다.