# Lecture 224 - Prerequisite Docker Networking

## Docker 네트워킹 모드

- 컨테이너를 실행하면 Docker 는 애플리케이션의 요구 사항에 맞게 연결을 맞춤화할 수 있는 여러 가지 네트워킹 모드를 제공한다.

### None 네트워크

- "none" 네트워크 모드에서는 컨테이너가 어떤 네트워크에도 연결되지 않는다.
- 따라서 외부 시스템과 통신할 수 없으며 외부 시스템도 컨테이너에 접근할 수 없다.
- none 네트워크 모드에서 시작된 여러 컨테이너는 서로 및 외부 네트워크로부터 완전히 격리된 상태로 유지된다.

```bash
$ docker run --network none nginx

```

### host 네트워크

- 호스트 네트워크 모드에서 컨테이너는 호스트의 네트워크 스택을 공유한다.
- 즉, 호스트와 컨테이너 간에 네트워크 격리가 없다.
- 예를 들어, 컨테이너 내부의 웹 애플리케이션이 포트 80에서 수신 대기하는 경우 해당 애플리케이션은 호스트의 포트 80에서 즉시 접속할 수 있다.
- 그러나 동일한 포트에 다른 컨테이너를 바인딩 하는 경우 두 프로세스가 동일한 포트를 공유할 수 없으므로 실패한다.

```bash
docker run --network host nginx
```

### bridge 네트워크

- Docker 의 기본 네트워크 모드는 Bridge 이다.
- Docker 를 설치하게 되면 호스트에 `docker0` 로 표시되는 bridge 라는 내부 사설 네트워크가 자동으로 생성되고, 기본 서브넷(일반적으로 172.17.0.0/16)이 설정됩니다.
- Docker는 내부적으로 호스트에 "docker0" 인터페이스를 생성하여 호스트와 컨테이너 간의 브리지 역할을 한다.
- 이 네트워크에 연결된 각 컨테이너는 이 서브넷에서 고유한 IP 주소를 할당받는다.

```bash

$ docker network ls

NETWORK ID NAME DRIVER SCOPE
2b6008726112 bridge bridge local
0beb4870b093 host host local
99035e02694f none null local

$ ip link show docker0

3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default
    link/ether 02:42:88:d3:93:c5 brd ff:ff:ff:ff:ff:ff
```

## 브리지 네트워크에 컨테이너 부착

- Docker 는 가상 인터페이스 쌍을 생성하여 각 컨테이너를 bridge 네트워크에 연결한다.
- 가상 인터페이스 쌍은 기본적으로 양 쪽 끝에 인터페이스가 있는 가상 케이블과 같다.
- 한 인터페이스는 호스트의 `docker0` 브리지에 연결되고, 다른 인터페이스는 컨테이너의 네트워크 네임스페이스 내부에 배치된다.

```bash
# Docker 호스트의 인터페이스 검사
$ ip link show

4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
link/ether 02:42:9b:5f:d6:21 brd ff:ff:ff:ff:ff:ff
8: vethbb1c343@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group
default
link/ether 9e:71:37:83:9f:50 brd ff:ff:ff:ff:ff:ff link-netnsid 1

# 컨테이너의 네트워크 네임스페이스 검사
$ ip -n b3165c10a92b link show

7: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0

# 컨테이너의 네트워크 인터페이스에 할당된 IP 주소 확인
$ ip -n b3165c10a92b addr show eth0

7: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
valid_lft forever preferred_lft forever
```

### 도커의 컨테이너 생성 시 절차

새로운 컨테이너 생성될 때 Docker 의 실행 절차

- 새로운 네임스페이스 생성
- 가상 인터페이스 쌍을 설정
- 한쪽 끝을 컨테이너의 네임스페이스에 연결하고 다른 쪽 끝을 `docker0` 브리지에 연결.
- 컨테이너 인터페이스에 IP 주소 할당.
- 가상 인터페이스 쌍에 일관된 넘버링

## 포트 매핑

- 기본적으로 컨테이너는 프라이빗 네트워크 세그먼트에서 실행된다.
- 동일한 네트워크 또는 호스트에 있는 다른 컨테이너만 액세스할 수 있다.
- Docker 의 Port Mapping 은 호스트의 포트를 컨테이너의 포트에 매핑하여 외부 액세스를 가능하게 만든다.
- 호스트의 매핑된 포트로 요청 시 매핑된 컨테이너로 요청이 전달된다.

```bash

# 호스트 8080 포트 -> 컨테이너 80 포트 매핑
$ docker run -p 8080:80 nginx
```

## 포트 포워딩 작동 방식

- Docker는 iptables 을 사용해 포트 포워딩을 구현한다.
- 이 메커니즘은 특정 호스트 포트에 도착하는 트래픽을 해당 컨테이너 포트로 변환하는 네트워크 주소 변환(NAT) 규칙을 추가한다.

```bash
iptables \\\\
-t nat \\\\
-A PREROUTING \\\\
-j DNAT \\\\
--dport 8080 \\\\
--to-destination 80
```

- Docker 는 `p 8080:80` 명령어 사용시 위와 같은 규칙을 추가한다.

```bash
$ iptables -nvL -t nat

Chain DOCKER (2 references)
target prot opt source destination
RETURN all -- anywhere anywhere
DNAT tcp -- anywhere anywhere tcp dpt:8080 to:172.17.0.2:80
```

- NAT 설정을 확인하면 위와 같이 NAT 설정이 된 것을 확인 할 수 있다.