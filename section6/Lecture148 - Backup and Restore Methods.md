# Lecture 148 - Backup and Restore Methods

## Kubernetes Backup

- 쿠버네티스의 경우 아래의 중요 구성요소를 백업하는 것이 좋다.
    - Declarative Configuration Files
        - Deployment, Pod, Service 와 같은 Resource 정의 파일
    - 클러스터 상태
        - `etcd` 클러스터에 저장된 정보.
    - Imperative Objects
        - 위의 파일에 문서화 되지 않는, 즉석에서 생성된 필수 리소스 파일(Namespace, Secrets, ConfigMap ...)

## 명령형 백업 vs 선언형 백업

- 선언적 방법이 선호되지만, 때로는 명령형 명령을 사용하여 리소스를 생성하는 경우가 있다.
- 이러한 변경 사항은 버전 제어 시스템에 저장되지 않을 수 있으며, 이로 인해 백업에 공백이 발생할 수 있다.
- 모든 구성을 캡처하려면 Kubernetes API 서버에 직접 쿼리 하는 방식으로 백업이 가능하다.

```bash
$ kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```


## etcd 클러스터 백업
- etcd 클러스터는 쿠버네티스 시스템의 중추 역할을 하며, 중요한 상태 및 구성 정보를 저장
- 반적으로 마스터 노드에 위치 한다.
- etcd 데이터는 설정 과정에서 지정된 전용 디렉터리에 저장된다.

```bash
ExecStart=/usr/local/bin/etcd \\
   --name ${ETCD_NAME} \\
   --cert-file=/etc/etcd/kubernetes.pem \\
   --key-file=/etc/etcd/kubernetes-key.pem \\
   --peer-cert-file=/etc/etcd/kubernetes.pem \\
   --peer-key-file=/etc/etcd/kubernetes-key.pem \\
   --trusted-ca-file=/etc/etcd/ca.pem \\
   --peer-trusted-ca-file=/etc/etcd/ca.pem \\
   --client-cert-auth \\
   --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
   --listen-peer-urls https://${INTERNAL_IP}:2380 \\
   --advertise-client-urls https://${INTERNAL_IP}:2379 \\
   --initial-cluster etcd-cluster-0 \\
   --initial-cluster-token etcd-cluster-0 \\
   --initial-cluster controller-0=https://${CONTROLLER0_IP}:2379 \\
   --initial-cluster-state new \\
   --data-dir=/var/lib/etcd
```

- etcd 는 etcdctl 명령을 통해 기본 제공 스냅샷 기능을 제공한다.

```bash
# 스냅샷 생성
$ ETCDCTL_API=3 etcdctl snapshot save snapshot.db

# 스냅샷 상태 확인
$ ETCDCTL_API=3 etcdctl snapshot status snapshot.db
```

## etcd 백업에서 복원 프로세스

- 백업 및 복원 작업 중에는 항상 필수 인증서 파일(CA 인증서, etcd 서버 인증서 및 키)을 제공헤야 힌디.

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/etcd/ca.crt \
--cert=/etc/etcd/etcd-server.crt \
--key=/etc/etcd/etcd-server.key
```

### A. Kubernetes API 서버 중지
- 복원 프로세스를 진행하기 전 API 서버를 중지해야 한다.

### B. 스냅샷 복원
- 스냅샹을 새 데이터 디렉토리로 복원한다.

### C. etcd 구성 업데이트
- etcd 구성 파일을 수정하여 새 데이터 디렉토리를 가르키도록 한다.

### D. 서비스 재시작
- 시스템 데몬을 다시 로드하고, etcd 서비스를 재시작한 후, 마지막으로 Kubernetes API 서버를 재시작한다.
