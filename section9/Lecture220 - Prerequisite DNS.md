# Lecture 220 - Prerequisite DNS

## 로컬 이름 확인 이해

- 동일한 네트워크에 두 대의 컴퓨터
- 컴퓨터 A의 IP 주소는 192.168.1.10
- 컴퓨터 B의 IP 주소는 192.168.1.11

```bash
$ ping 192.168.1.11

Reply from 192.168.1.11: bytes=32 time=4ms TTL=117
Reply from 192.168.1.11: bytes=32 time=4ms TTL=117

```

- 컴퓨터 B 는 "db" 라는 이름을 부여
- "db"를 인식할 수 있도록 컴퓨터 A의 `/etc/hosts` 파일에 항목을 추가

```bash
$ cat >> /etc/hosts

192.168.1.11    db

```

```bash
$ ping db

PING db (192.168.1.11) 56(84) bytes of data.
64 bytes from db (192.168.1.11): icmp_seq=1 ttl=64 time=0.052 ms
64 bytes from db (192.168.1.11): icmp_seq=2 ttl=64 time=0.079 ms

```

- `/etc/hosts` 시스템에 등록되면 실제 호스트 이름과 별칭이 일치하는지는 확인하지 않는다.
- 하나의 IP 주소에 여러 개의 별칭을 지정할 수 있다.
- `ping`, `curl`, `SSH` 등을 통해 호스트 이름을 참조할 때마다 시스템은 `/etc/hosts` 파일을 참고하여 등록와 IP 주소 매핑을 확인한다.
- 이 과정을 `Name Resolution` 이라고 한다.

```bash
$ cat >> /etc/hosts

192.168.1.11    db
192.168.1.11    www.google.com

$ ping db

PING db (192.168.1.11) 56(84) bytes of data.
64 bytes from db (192.168.1.11): icmp_seq=1 ttl=64 time=0.052 ms
64 bytes from db (192.168.1.11): icmp_seq=2 ttl=64 time=0.079 ms

$ ping www.google.com
PING www.google.com (192.168.1.11) 56(84) bytes of data.
64 bytes from www.google.com (192.168.1.11): icmp_seq=1 ttl=64 time=0.052 ms
64 bytes from www.google.com (192.168.1.11): icmp_seq=2 ttl=64 time=0.079 ms

```

## 중앙 집중식 DNS 서버를 사용한 확장

- 소규모 네트워크에서는 로컬 `/etc/hosts` 파일을 관리하는 것이 효과적이다.
- 하지만, 시스템 수가 늘어나고 IP 주소가 변경되면 유지 관리가 어려워진다.
- 수많은 로컬 호스트 매핑을 관리하는 어려움을 극복하기 위해 조직에서는 모든 매핑을 중앙 DNS 서버에 통합한다.
- 중앙 DNS 서버가 IP 주소 192.168.1.100에 있다고 가정
- 각 호스트의 `/etc/resolv.conf` 파일을 수정하여 DNS 서버로 지정되도로고 변경.

```bash
$ cat /etc/resolv.conf

nameserver 192.168.1.100

```

- 위와 같이 구성이 완료되면, 찾을 수 없는 모든 호스트 이름은 DNS 서버의 `/etc/hosts` 서버를 통해 확인된다.
- IP 주소가 변경되면 각 시스템을 개별적으로 수정하는 대신 DNS 서버의 레코드를 업데이트한다.
- `/etc/hosts` 파일의 경우 DNS 보다 우선순위가 높아, `/etc/hosts` 에 등록된 정보를 먼저 조회하고 없으면 DNS 를 조회한다.

```bash
$ cat /etc/nsswitch.conf
...
hosts:          files dns
...

```

- `/etc/nsswitch.conf` 파일은 우선순위 검색 순서를 변경할 수 있다.
- 이 구성에서 시스템은 먼저 `/etc/hosts` 파일에서 호스트 이름을 검색한다.
- 일치하는 호스트 이름이 없으면 DNS 서버에 쿼리를 보낸다.
- 외부 도메인 (Facebook) 을 확인하려면 공용 DNS 서버 (예: Google의 8.8.8.8) 을 추가하거나 내부 DNS 서버를 구성하여 확인되지 않은 쿼리를 공용 DNS 로 바라보게 한다.

## 도메인 이름과 구조

- 도메인 이름 (예: [www.google.com](http://www.google.com/)) 은 점으로 구분된 여러 부분으로 구성된다.
- 최상위 도메인(TLD) 는 끝에 나타난다. (예: .com, .net, .edu, .org)
- 도메인 이름은 TLD 보다 앞에 나온다. (예: google)
- 도메인 이름 앞의 모든 세그먼트는 하위 도메인으로 간주된다. (예: www)

### 사용자 도메인 입력 시나리오

- 사용자가 브라우저에 `apps.google.com` 와 같은 도메인을 입력하였을 때의 시나리오.
- 도메인 입력 시 내부 네트워크의 로컬 DNS 서버가 해당 도메인의 IP 주소를 가지고 있는지 조회한다.
- 만약, 없다면 요청을 더 상위 DNS 서버로 전달한다.
- DNS 는 계층적 구조로 되어 있어, 먼저 최상위 루트 DNS 서버가 요청을 받고, 해당 도메인의 최상위 도메인 `.com` 을 관리하는 DNS 서버 주소를 알려준다.
- 그 다음 `.com` DNS 서버는 구글 도메인을 관리하는 권한 있는 DNS 서버 주소를 알려준다.
- 마지막으로 구글 DNS 서버가 `apps.google.com` 에 해당하는 실제 IP 주소를 반환한다.
- 해당 IP 주소는 네트워크 통신을 위해 사용되며, 로컬 DNS 서버는 이 응답을 일정 기간 캐싱하여 같은 요청이 올 경우 더 빠르게 처리할 수 있다.
- 또한, 조직읟 도메인([mycompany.com](http://mycompany.com/)) 도 이런 구조를 활용해 하위 도메인([mail.mycompany.com](http://mail.mycompany.com/)) 을 관리하며, 각각 다른 서비스에 연결할 수 있다.
- 즉, 도메인 이름이 계층적으로 구성되고, DNS 서버들이 순차적이고 협력적으로 요청을 처리하면서 IP 주소를 찾아내어 인터넷 접속이 가능하도록 하는 체계라는 것이다.

## 일반적인 DNS 레코드 유형 개요

- DNS 레코드는 호스트 이름을 IP 주소에 매핑하고 다양한 용도로 사용된다.
- A 레코드
    - 도메인 이름(호스트 이름)을 IPv4 주송에 직접 연결.
    - A 는 Address 를 의미
    - 에시
        - 사용자가 브라우저에 `web-server.com` 을입력
        - 컴퓨터가 DNS 서버에 도메인 질의
        - DNS 서버는 A 레코드를 찾아서 IPv4 주소 반환.
- AAAA 레코드
    - 도메인 이름을 IPv6 주소에 연결.
    - IPv4 주소 고갈 문제를 해결하기 위해 나온 것이 IPv6 이며, AAAA 레코드는 이 새로운 주소 체계를 찾기 위한 것.
    - 예시
        - 사용자가 `web-server.com` 을 입력
        - 컴퓨터가 DNS 서버에서 AAAA 레코드 질의
        - DNS 서버는 그 주소는 `2001:0db8:85a3::8a2e:0370:7334` 라고 대답
        - 컴퓨터는 해당 IPv6 주소로 접근
- CNAME (Canonical NAME, 정식 이름 레코드)
    - 하나의 도메인 이름을 다른 도메이 이름의 별칭으로 만든다.
    - CNAME 레코드는 절대 IP 주소를 직접 가르키지 않고, 항상 도메인 이름을 가르킨다.
    - 예시
        - `web-server.com` 이라는 주 도메인이 `192.168.1.1` (A 레코드)를 가르킨다 가정.
        - 사용자가 `food.web-server.com` 이라는 주소를 질의하면, DNS 서버는 `web-server.com` 의 별칭인 것을 확인하고 `web-server.com` 을 찾을 것을 답변
        - 컴퓨터는 다시 DNS 서버에 `web-server.com` 을 질의
        - DNS 서버는 A 레코드를 찾아서 `192.168.1.1` 을 응답.

## DNS 확인 도구 테스트

- `ping` 은 기본 DNS 확을 위한 가장 일반적인 도구이다.
- `nslookup` 및, `dig` 를 이용하면 더 자세한 정보를 얻을 수 있다.

### nslookup과 dig 사용

- `nslookup` 명령은 `/etc/hosts` 에 구성된 DNS 서버만 쿼리한다.
- `dig` 명령은 DNS 쿼리에 대한 포괄적인 세부 정보를 제공한다.

```bash
$ nslookup google.com
Server:		168.126.63.1
Address:	168.126.63.1#53

Non-authoritative answer:
Name:	google.com
Address: 142.250.207.110

```

```bash
% dig google.com

; <<>> DiG 9.10.6 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 6128
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		213	IN	A	142.250.76.46

;; Query time: 11 msec
;; SERVER: 168.126.63.1#53(168.126.63.1)
;; WHEN: Thu Nov 06 09:21:30 KST 2025
;; MSG SIZE  rcvd: 55

```