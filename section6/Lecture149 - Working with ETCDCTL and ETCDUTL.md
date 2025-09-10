# Lecture 149 - Working with ETCDCTL and ETCDUTL

## etcdctl 및 etcdutl

- etcdctl
    - etcd 와 상호작용하는 커맨드라인 클라이언트.
    - 실행 중인 etcd 서버에 연결하여 스냅샷 기반 백업 및 확인을 할 때 사용.
    - API 버전 3.x 를 사용해야 한다.
- etcdutl
    - etcd 의 데이터 디렉토리(data directory) 를 파일 기반으로 백업하고, 스냅샷 파일을 복원할 때 사용.

## etcd 백업 방법

### etcdctl 을 사용한 스냅샷 백업

- etcd 서버의 `endpoint`, `-cacert`, `-cert`, `-key` 정보가 필요하다.

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \\
--endpoints=https://127.0.0.1:2379 \\
--cacert=/etc/etcd/ca.crt \\
--cert=/etc/etcd/etcd-server.crt \\
--key=/etc/etcd/etcd-server.key

```

### etcdutl을 사용한 파일 백업

- etcd의 데이터 디렉터리(data-dir)에 있는 모든 파일(db 및 WAL 파일)을 대상 디렉터리로 복사.
- etcd가 실행 중이지 않아도 가능하다.
- etcd의 데이터 디렉터리 경로만 지정하면 된다.

```bash
$ etcdutl backup --data-dir /var/lib/etcd --backup-dir /backup/etcd-backup

```

## 스냅샷 상태 확인

- 생성된 스냅샷 파일의 메타데이터(크기, 버전, 키 개수 등)를 확인
- 복원 전에 스냅샷 파일의 무결성을 검증하는 데 유용

```bash
$ etcdctl snapshot status /backup/etcd-snapshot.db \\
  --write-out=table

```

## etcdutl 을 사용한 복원

- `etcdutl backup`으로 생성한 백업은 단순히 백업 내용을 `/var/lib/etcd` 디렉터리에 복사한 후 etcd 서비스를 재시작하면 된다.

```bash
$ etcdutl snapshot restore /backup/etcd-snapshot.db \\
  --data-dir /var/lib/etcd-restored

```