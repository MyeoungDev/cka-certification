# Lecture 16 - ETCD Commands (Optional)

## `etcdctl` API 버전

- `etcdctl`은 ETCD 서버와 상호작용하기 위해 API 버전 2와 버전 3을 사용할 수 있다.
- `etcdctl`은 기본적으로 API 버전 2를 사용하도록 설정되어 있다.
- 원하는 API 버전을 사용하기 위해서는 `export ETCDCTL_API=3`  환경변수 설정 필요.
- **버전별 커맨드 차이**:
    - **버전 2 (`ETCDCTL_API=2`)**: `etcdctl backup`, `etcdctl cluster-health`, `etcdctl mk`, `etcdctl mkdir`, `etcdctl set` 등
    - **버전 3 (`ETCDCTL_API=3`)**: `etcdctl snapshot save`, `etcdctl endpoint health`, `etcdctl get`, `etcdctl put` 등

## ETCD

### ETCDCTL 유틸리티 (선택 사항) 요약

이 내용은 `etcdctl` 커맨드라인 도구를 사용하여 ETCD 서버와 상호작용하는 데 필요한 추가 정보를 다룹니다.

**1. `etcdctl` API 버전**

- **두 가지 버전**: `etcdctl`은 ETCD 서버와 상호작용하기 위해 API 버전 2와 버전 3을 사용할 수 있습니다.
- **기본 설정**: `etcdctl`은 기본적으로 API 버전 2를 사용하도록 설정되어 있습니다.
- **버전별 커맨드 차이**:
    - **버전 2 (`ETCDCTL_API=2`)**: `etcdctl backup`, `etcdctl cluster-health`, `etcdctl mk`, `etcdctl mkdir`, `etcdctl set` 등
    - **버전 3 (`ETCDCTL_API=3`)**: `etcdctl snapshot save`, `etcdctl endpoint health`, `etcdctl get`, `etcdctl put` 등
- **환경 변수 설정**: 원하는 API 버전을 사용하려면 `ETCDCTL_API` 환경 변수를 설정해야 합니다.
    - **예시**: `export ETCDCTL_API=3`
- **호환성**: API 버전 2로 설정된 상태에서는 버전 3 커맨드가 작동하지 않으며, 반대로 버전 3으로 설정된 상태에서는 버전 2 커맨드가 작동하지 않습니다.

**2. ETCDCTL 인증서**

- **필요성**: `etcdctl`이 ETCD API 서버와 통신하려면, 보안을 위해 인증서 파일의 경로를 지정하여 인증을 받아야 합니다.
- **인증서 경로**: 일반적으로 인증서 파일은 마스터 노드의 `/etc/kubernetes/pki/etcd/` 경로에 위치합니다.
    - `ca.crt`: 인증 기관(CA) 인증서
    - `server.crt`: 서버 인증서
    - `server.key`: 서버 키
- **커맨드 예시**: 인증서 파일을 지정하여 `etcdctl`을 실행하는 최종 형태는 다음과 같습니다.
    - `-cacert /etc/kubernetes/pki/etcd/ca.crt
    --cert /etc/kubernetes/pki/etcd/server.crt
    --key /etc/kubernetes/pki/etcd/server.key`
    - 실제 `kubectl` 명령과 함께 사용할 때는 `kubectl exec`를 이용하여 파드 내에서 실행합니다.

**3. 최종 커맨드 예시**
제공된 예시 커맨드는 `kubectl`을 사용하여 `etcd-master` 파드 내에서 `etcdctl`을 실행하는 방법을 보여줍니다.

`kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key"`

- **명령 설명**:
    - `kubectl exec etcd-master -n kube-system`: `kube-system` 네임스페이스의 `etcd-master` 파드에서 명령을 실행합니다.
    - `ETCDCTL_API=3`: API 버전을 3으로 설정합니다.
    - `etcdctl get / --prefix --keys-only --limit=10`: `/` 경로 아래의 모든 키를 접두사 기준으로 가져오며, 값은 제외하고 키 이름만 10개까지 표시합니다.
    - `-cacert ... --cert ... --key ...`: `etcdctl`이 서버에 인증하기 위한 인증서 경로를 지정합니다.