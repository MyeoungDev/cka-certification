# Lecture 156 - Authentication

## Authentication

- 쿠버네티스는 사용자 계정을 기본적으로 관리하지 않는다.
- 자격 증명, 인증서 등을 통해 외부 시스템을 사용하여 인증을 수행한다.
- kubectl 명령줄 도구, API 호출 등 모든 요청은 Kube API 서버에서 처리되며, API 서버는 하나 이상의 사용 가능한 인증 매커니즘을 수행하여 수신 요청은 인증한다.

## 정적 암호 파일을 사용한 기본 인증

- 간단한 인증 방법 중 하나는 정적 비밀번호 파일을 사용하는 것이다.

```bash

$ vi user-details.csv

password123,user1,u0001
password123,user2,u0002
password123,user3,u0003
password123,user4,u0004
password123,user5,u0005

$ kube-apiserver --basic-auth-file=user-details.csv

```

## 정적 토큰 파일을 사용한 토큰 기반 인증

- 정적 토크 파일을 사용해서 인증을 할 수 있다.
- 파일에는 토큰, 사용자 이름, 사용자 ID, 그리고 선택적 그룹 할당이 포함된다.
- 토큰 파일을 생성하면 `-token-auth-file` 옵션과 함께 토큰 파일을 전달하여 API 서버를 시작할 수 있다.

```bash
# 예시 CSV 파일 형식
$ vi user-token-details.csv

KpjCVbI7cFAHYPkByTIzRb7gulcUc4B,user10,u0010,group1
rJjncHmvtXHc6MlWQddhtvNyyhgTdxSC,user11,u0011,group1
mjpoFTEiFOkL9toikaRNTt59ePtczZSq,user12,u0012,group2
PG41IXhs7QjqWkmBkvGT9gclOyUqZj,user13,u0013,group2

```

```bash
$ kube-apiserver --token-auth-file=user-token-details.csv

$ curl -v -k <https://master-node-ip:6443/api/v1/pods> --header "Authorization: Bearer KpjCVbI7cFAHYPkByTIzRb7gulcUc4B"

```

## 보안 고려 사항

- 프로덕션 환경에서는 정적 파일을 사용하여 보안 관련 내용을 일반 텍스트로 저장 사용하는 것은 권장되지 않는다.
- 보안 강화를 위해 인증서 기반 인증을 사용하거나 외부 ID Provider 와 통합하는 것을 권장한다.
- `kubeadm` 을 사용하여 클러스터를 구성하는 경우, `API Server` 파드에 인증 파일 볼륨을 마운트 한 후 재실행 해야 한다.
- 이능이 완료된 후 RBAC 와 같은 권한부여를 통해 올바른 사용자 권한 제어를 진행해야 한다.

# TLS-Basics

- 쿠버네티스 보안 통신에 TLS 인증서가 필수적으로 사용된다.
- 이를 이해하기 위해서는 TLS 에 대한 이해가 필수적이다.

### 초기 애플리케이션 (보안 X)

- 평문의 데이터를 이용해서 통신
- 해커가 민감 데이터를 가로채고 악용 가능. (로그인 자격, 금융 등등)

### 대칭키 암호화 적용

- 전송 데이터를 암호화 키를 사용하여 암호화. 애플리케이션은 동일한 키로 복호화를 진행.
- 이를 시행하기 위해 네트워크를 통해 키를 전송하는 방식 또한 공격자가 키를 가로채 암호화된 데이터를 복호화해서 사용할 수 있다는 취약점 존재.

### 비대칭 암호화 적용

- 비대칭 암호화는 개인 키와 공개 키 한 쌍의 키 조합을 사용.
- 개인 키는 애플리케이션에서 관리되고, 공개 키는 사용자들에게 제공.
- 공개 키로 암호화된 데이터는 개인 키로만 복호화 할 수 있으므로, 공격자가 가로챈 데이터는 안전하게 보호 가능.

### SSH 접근에 대한 비대칭 암호화 적용

```bash

$ ssh-keygen

# ex: id_rsa (private key), id_rsa.pub (public key) 와 같은 파일 두개가 생성됨.

$ cat ~/.ssh/authorized_keys

ssh-rsa AAAAB3NzaC1yc...KhtUBfoTz1BqRV1NThvO0apzEwRQo1mWx user1

# 키 쌍을 사용한 SSH 접근
$ ssh -i id_rsa user1@server1

```

## CA 인증서 동작 방식

- 서버는 키 쌍 (private, public) 생성.
- 사용자가 HTTPS 요청하면 서버는 인증서에 포함된 공캐 키 전송.
- 클라이언트의 브라우전느 서버의 공개 키를 사용하여 새로 생성된 대칭 키를 암호화.
- 암호화된 대칭 키는 서버로 다시 전송.
- 서버는 개인 키를 사용하여 대칭 키 복호화.
- 이후 모든 통신은 이 대칭키를 사용하여 암호화.
- 공격자가 유효하지 않은 인증서를 생성하여 브라우저가 신뢰되는 애플리케이션과 연결된 것처럼 착각하게 할 수 있지만,
- 최신 브라우저는 인증서가 신뢰할 수 없는 경우 사용자에게 경고한다.

```bash

# Generate a private key
$ openssl genrsa -out my-bank.key 1024

# Extract the public key
$ openssl rsa -in my-bank.key -pubout > mybank.pem

```

- 또한, 인증서에는 진위 여부를 확인하는 데 도움이 되는 필수 세부 정보가 포함되어 있다.
    - 발금 기관의 신원
    - 서버의 공개 키
    - 도메인 및 기타 관련 정보

```bash
Certificate:
Data:
Serial Number: 420327018966204255
Signature Algorithm: sha256WithRSAEncryption
Issuer: CN=kubernetes
Validity
Not After : Feb  9 13:41:28 2020 GMT
Subject: CN=my-bank.com
X509v3 Subject Alternative Name:
DNS:mybank.com, DNS:i-bank.com,
DNS:we-bank.com,
Subject Public Key Info:
00:b9:b0:55:24:fb:a4:ef:77:73:7c:9b

```

- 브라우저는 인증서 서명 및 유효성 검사를 위해 인증 기관(CA) 를 사용한다.
- Symantec, DigiCert, Komodo, GlobalSign 과 같은 유명 CA는 개인 키를 사용하여 인증서 서명 요청(CSR) 에 서명한다.
- 웹 서버용 CSR을 생성하면 서명을 위해 CA로 전송된다.
- 그 이후 사용자의 정보가 검증되면 CA 가 인증서에 서명하고 웹 서버에 설치되도록 다시 전송한다.

```bash
$ openssl req -new -key my-bank.key -out my-bank.csr -subj "/C=US/ST=CA/O=MyOrg, Inc./CN=my-bank.com"

```

## 요약

- 비댕칭 암호화는 공개 키와 개인 키 쌍을 사용하여 대칭 키를 안전한게 교환하는데 사용한다.
- SSH 엑세스는 Key Pair 를 사용하여 보안해야한다.
- 웹 서버는 CA 서명 인증서를 사용하여 HTTPS 연결을 설정한다.
- 인증서 서명 요청(CSR)이 생성되어 서명을 위해 CA 로 전송된다.
- 서명된 인증서는 서버의 키 쌍과 결합되어 통신 세션을 보호한다.
- 비대칭 키 쌍의 두 키 모두 데이터를 암호화할 수 있지만, 개인키로 암호화된 데이터는 공개 키를 가진 누구든 복호활 수 있다는 점을 주의해야 한다.