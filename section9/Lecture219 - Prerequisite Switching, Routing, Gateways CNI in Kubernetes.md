# Lecture 219 - Prerequisite Switching, Routing, Gateways CNI in Kubernetes

## 기본 네트워킹 개념

## 네트워크 인터페이스 및 스위칭

- A 와 B 두 시스템 간 통신을 가능하게 하려면 각 시스템이 해당 네트워크 인터페이스를 가진 스위치에 연결되어야 한다.
- Linux 시스템에서 사용 가능한 인터페이스를 확인 하기위해 `ip link` 명령어를 사용한다.

```bash
$ ip link

eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000

```

- 이러한 시스템이 네트워크 `192.168.1.0` 에 속한다고 가정하면 다음 명령을 사용하여 IP 주소를 할당할 수 있다.

```bash
$ ip addr add 192.168.1.10/24 dev eth0

```

- 위와 같이 구성 후 동일한 네트워크 내의 다른 호스트에 `ping` 을 보내 연결을 확인한다.

```bash
$ ping 192.168.1.11

Reply from 192.168.1.11: bytes=32 time=4ms TTL=117

```

## 서브넷 간 라우팅

- 서로 다른 네트워크 간의 통신을 위해서는 라우터가 필요하다.
- 라우터는 두 개 이상의 네트워크를 연결하고 각 네트워크의 IP 주소를 관리한다.
- 예를 들어, 첫 번째 네트워크는 `192.168.1.1` 이고 두 번째 네트워크는 `192.168.2.1` 이다.
- 네트워크 192.168.1.0(예: IP 192.168.1.11)에 있는 시스템이 네트워크 192.168.2.0에 있는 시스템과 통신해야 할 때, 라우터는 패킷을 라우터로 전달한다.
- 각 시스템은 패킷이 의도한 목적지에 도달하도록 게이트웨이 또는 특정 경로 항목을 구성해야 한다.

```bash
$ route

Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.219.0   0.0.0.0         255.255.255.0   U     100    0        0 enp0s8

```

- 현재 라우팅 테이블을 확인하기 위해서는 `route` 명령어를 사용한다.
- 처음에는 통신이 동일한 서브넷으로만 제한된다.

```bash
$ ip route add 192.168.2.0/24 via 192.168.1.1

$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.2.0     192.168.1.1     255.255.255.0   UG    0      0        0 eth0

```

- 라우터(IP 192.168.1.1)을 통해 192.168.2.0 으로 향하는 트래픽을 라우팅하려면 라우팅을 추가해야 한다.

## 인터넷 액세스를 위한 기본 경로 구성

- 인터넷 접속(예: 172.217.194.0 과 같은 외부 호스트 접속)을 활성화하기 위해서는 라우터를 Default Gateway로 설정해야 한다.
    - Default Gateway 는 컴퓨터가 이니터넷이나 외부 네트워크에 접속할 때, 어디(어떤 라우터)로 데이터를 전달해야 하는지 알려주는 기본 경로이다.
    - 내 컴퓨터는 네트워크 바깥(예: 인터넷)으로 보내는 모든 데이터를 Default Gateway 로 전달한다.
    - 특정 목적지에 대한 Routing Table 이 존재하지 않으면 `default` 경로를 따라간다.

```bash
$ ip route add default via 192.168.2.1

$ route

Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.1.0     192.168.2.1     255.255.255.0   UG    0      0        0 eth0
172.217.194.0   192.168.2.1     255.255.255.0   UG    0      0        0 eth0
default         192.168.2.1     0.0.0.0         UG    0      0        0 eth0

```

### 기본 경로 이해

- `default` 또는 `0.0.0.0` 항목은 라우팅 테이블에 명시적으로 나열되지 않은 모든 대상이 지정된 게이트웨이를 통해 전달됨을 나타낸다.
- 여러 라우터가 사용되는 시나리오(예: 하나는 인터넷 트래픽 처리 다른 하나는 내부 네트워크 관리) 의 경우, 각 네트워크에 특정 라우팅 항목과 다른 모든 트래픽에 대한 `default` 경로가 있는지 확인해야 한다.
- 예를 들어, 대체 게이트웨이(192.168.2.2) 를 통해 네트워크 192.168.1.0 으로 트래픽을 라우팅하려면 아래와 같이 추가해줘야 한다.

```bash
$ ip route add 192.168.1.0/24 via 192.168.2.2

$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         192.168.2.1     0.0.0.0         UG    0      0        0 eth0
192.168.1.0     192.168.2.2     255.255.255.0   UG    0      0        0 eth0

```

## Linux 호스트를 라우터로 구성

- 세 대의 호스트(A, B, C) 가 있고, 호스트 B 가 두 개의 인터페이스를 사용하여 두 개의 호스트(A, C)가 연결되는 시나리오.

호스트 A: 192.168.1.5
호스트 B: 192.168.1.6 및 192.168.2.6
호스트 C: 192.168.2.5

- 호스트 A가 호스트 C와 통신하려면 호스트 A는 네트워크 192.168.2.0 으로 향하는 트래픽을 호스트 B로 전달해야 한다. (게이트웨이로 192.168.1.6 사용)
- 호스트 C는 호스트 B를 통해 192.168.1.0 네트워크에 대한 경로를 추가해야 한다. (게이트웨이로 192.168.2.6 사용)

```bash
# Host A
$ ip route add 192.168.2.0/24 via 192.168.1.6

# Host C
$ ip route add 192.168.1.0/24 via 192.168.2.6

```

- 위와 같이 설정이 완료되면, 호스트 A 와 호스트 C 사이의 통신이 가능하다.

### Linux에서 IP 전달 활성화

- 올바른 라우팅 테이블이 있더라도 Linux 는 보안을 위해 기본적으로 인터페이스 간에 패킷을 전달하지 않는다.
- 이 설정은 `.NET Framework` 의 `/proc/sys/net/ipv4/ip_forward` 안의 IP forwarding parameter 의해 제어된다.

```bash
$ cat /proc/sys/net/ipv4/ip_forward

0

```

- 위 명령어를 통해 IP Forwarding 상태를 확인할 수 있다.
- 반환 값이 0인 경우 패킷 전달이 비활성화 되어있음을 나타낸다.

```bash

$ echo 1 > /proc/sys/net/ipv4/ip_forward

$ cat /proc/sys/net/ipv4/ip_forward

1

```

- `echo 1 > /proc/sys/net/ipv4/ip_forward` 명령어를 사용할 경우 IP Forward 를 일시적으로 활성화 할 수 있다.

```bash

$ vi /etc/sysctl.conf

...

net.ipv4.ip_forward = 1

...

```

- 시스템 재시작 후에도 IP Forwarding 을 활성화 하려면 `net.ipv4.ip_forward = 1` 설정을 수정해야 한다.