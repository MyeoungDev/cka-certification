# Lecture 162 - View Certificate Details

## Understand Cluster Configuration

- 인증서에 대한 검증을 시작하기 전에 클러스터 설정을 이해하는 것이 중요하다.

### 네이티브 쿠버네티스 구성

- 쿠버네티스 구성 요소가 네이티브 서비스로 배포될 경우 `.service` 파일을 확인하여 인증서 구성을 파악할 수 있다.

```bash
$ cat /etc/systemd/system/kube-apiserver.service

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\\\
--advertise-address=172.17.0.32 \\\\
--allow-privileged=true \\\\
--apiserver-count=3 \\\\
--authorization-mode=Node,RBAC \\\\
--bind-address=0.0.0.0 \\\\
--client-ca-file=/var/lib/kubernetes/ca.pem \\\\
--enable-swagger-ui=true \\\\
--etcd-cafile=/var/lib/kubernetes/ca.pem \\\\
--etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\\\
--etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\\\
--event-ttl=1h \\\\
--kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\\\
--kubelet-client-certfile=/var/lib/kubernetes/kubelet-client.crt \\\\
--kubelet-client-key=/var/lib/kubernetes/kubelet-client.key \\\\
--kubelet-https=true \\\\
--service-node-port-range=30000-32767 \\\\
--tls-cert-file=/var/lib/kubernetes/kube-apiserver.crt \\\\
--tls-private-key-file=/var/lib/kubernetes/kube-apiserver-key.pem \\\\
--v=2

```

### kubeadm 을 사용한 쿠버네티스 구성

- `kubeadm` 과 같은 자동화된 프로비저닝 도구를 사용하면 인증서 생성 및 구성이 자동으로 처리된다.
- 이 경우 쿠버네티스 구성 요소는 서비스 데몬 대신 파드 형태로 배포된다.
- 따라서, 구성 요소는 매니페스트 파일에 포드로 정의된다.

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
spec:
containers:
- command:
- kube-apiserver
- --authorization-mode=Node,RBAC
- --advertise-address=172.17.0.32
- --allow-privileged=true
- --client-ca-file=/etc/kubernetes/pki/ca.crt
- --disable-admission-plugins=PersistentVolumeLabel
- --enable-admission-plugins=NodeRestriction
- --enable-bootstrap-token-auth=true
- --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
- --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
- --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
- --insecure-port=0
- --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
- --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
- --proxy-client-certfile=/etc/kubernetes/pki/apiserver-kubelet-client.crt
- --proxy-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
- --request-timeout=30s

```

## 인증서 인벤토리 생성

- 인증서 상태 점검을 수행할 때는 스프레드시트를 사용하여 다음과 같은 세부 정보 체크리스트를 만들어서 관리하는 것이 필수적이다.
    - 인증서 파일 경로
    - 구성된 이름 및 대체 이름
    - 관련 조직
    - 인증서 소유자
    - 인증 기관
    - 만료일

kube-apiserver 메니페스트는 `/etc/kubernetes/manifests` 해당 경로에 위치하는 파일을 통해 구성을 확인할 수 있다.

```bash

spec:
  containers:
    - command:
      - kube-apiserver
      - --authorization-mode=Node,RBAC
      - --advertise-address=172.17.0.32
      - --allow-privileged=true
      - --client-ca-file=/etc/kubernetes/pki/ca.crt
      - --disable-admission-plugins=PersistentVolumeLabel
      - --enable-admission-plugins=NodeRestriction
      - --enable-bootstrap-token-auth=true
      - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
      - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
      - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
      - --etcd-servers=https://127.0.0.1:2379
      - --insecure-port=0
      - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
      - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
      - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
      - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
      - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
      - --secure-port=6443
      - --service-account-key-file=/etc/kubernetes/pki/sa.pub
      - --service-cluster-ip-range=10.96.0.0/12
      - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
      - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key

```

## Inspecting Certificate Details

- 인증서 파일을 확인한 후 OpenSSL을 사용하여 파일을 디코딩하고 세부 정보를 확인해야 한다.

```bash
$ openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout

```

- 위 명령어는 아래와 같은 항목을 확인할 수 있다.
    - 주제 이름 및 대체 이름
    - 유효기간 (만료일 포함)
    - 발급 인증 기관
- 이를 통해 인증서가 올바른 기관 및 만료 여부 등을 확인해야 한다.
- 인증서가 만료되었거나 잘못 구성되면 클러스터 운영에 심각한 장애가 발생할 수 있다.

## Troubleshooting with Logs

- 쿠버네티스 구성요소들의 로그를 확인하는 습관은 트러블 슈팅에 큰도움이 된다.

### Native Cluster

- Native Cluster 구성은 Linux System Daemon 으로 되어 있기 때문에 `journalctl` 을 통해 시스템 로그를 확인하자.

```bash

$ journalctl -u etcd.service -l

...

WARNING: 2019/02/13 02:53:30 Failed to serve client requests on 127.0.0.1:2379
Failed to dial 127.0.0.1:2379: connection error: desc = "transport: authentication handshake failed: remote error: tls: bad certificate"; please retry.

```

### kubeadm Cluster

- `kubeadm` 을 통해 쿠버네티스트 를 구성할 경우 핵심 구성 요소가 Pod로 배포되므로 아래와 같이 진행한다.
    - `kubectl logs <pod-name>`
- 그러나 장애로 인해 API 서버나 ETCD 가 다운되어 `kubectl` 이 응답하지 않는경우 컨테이너 레벨에서 로그를 확인하면 된다.
    - `docker ps -a`
    - `docker logs <container-id>`