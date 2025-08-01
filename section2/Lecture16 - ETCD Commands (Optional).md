# Lecture 16 - ETCD Commands (Optional)

## `etcdctl` API 버전

- `etcdctl`은 ETCD 서버와 상호작용하기 위해 API 버전 2와 버전 3을 사용할 수 있다.
- `etcdctl`은 기본적으로 API 버전 2를 사용하도록 설정되어 있다.
- 원하는 API 버전을 사용하기 위해서는 `export ETCDCTL_API=3`  환경변수 설정 필요.
- **버전별 커맨드 차이**
    - **버전 2 (`ETCDCTL_API=2`)**: `etcdctl backup`, `etcdctl cluster-health`, `etcdctl mk`, `etcdctl mkdir`, `etcdctl set` 등
    - **버전 3 (`ETCDCTL_API=3`)**: `etcdctl snapshot save`, `etcdctl endpoint health`, `etcdctl get`, `etcdctl put` 등

## ETCDCTL 인증서
- `etcdctl` 이 ETCD API 서버오 통신하려면, 보안을 위해 인증서 파일의 경로를 지정하여 인증을 받아야 한다.
- 일반적으로 인증서 파일은 마스터 노드의 `/etc/kubernetes/pki/etcd/` 경로에 있다.
    - `ca.crt`: 인증 기관(CA) 인증서
    - `server.crt`: 서버 인증서
    - `server.key`: 서버 키
- 예시
    - `-cacert /etc/kubernetes/pki/etcd/ca.crt
    --cert /etc/kubernetes/pki/etcd/server.crt
    --key /etc/kubernetes/pki/etcd/server.key`
## 최종 예시

```
kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key"
```
- 명령 설명
    - `kubectl exec etcd-master -n kube-system`: `kube-system` 네임스페이스의 `etcd-master` 파드에서 명령을 실행.
    - `ETCDCTL_API=3`: API 버전을 3으로 설정.
    - `etcdctl get / --prefix --keys-only --limit=10`: `/` 경로 아래의 모든 키를 접두사 기준으로 가져오며, 값은 제외하고 키 이름만 10개까지 표시.
    - `-cacert ... --cert ... --key ...`: `etcdctl`이 서버에 인증하기 위한 인증서 경로를 지정.
