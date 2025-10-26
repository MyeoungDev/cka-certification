# Lecture 206 - Persistent Volumes


- 스토리지 설정이 각 Pod 정의에 직접 포함되는 방식이 있다.

```bash
volumes:
- name: data-volume
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

- 이 방식은 많은 사용자가 여러 개의 파드를 배포하는 환경에서 모든 파드 파일에 스토리지 구성을 관리하기에 부적합하다.
- 문제를 해결하기 위해 관리자는 중앙 집중식 스토리지 풀을 생성할 수 있다.
- 사용자는 Persistent Volume Claim 을 생성하여 필요에 따라 스토리지의 일부를 요청할 수 있다.
- 이 개념은 Persistent Volume 을 통해 구현되었다.
- Persistent Volume 은 관리자가 정의하고 관리하는 클러스터 전체 스토리지 리소스 이다.
- 클러스터에서 실행되는 애플리케이션은 Persistent Volume Claim 을 통해 PV에 바인딩하여 이를 활용한다.

## Persistent Volume 생성

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
```

- `spec.accessModes` 를 통해 노드에 볼륨을 마운트 하는 방법을 결정한다.
  - `ReadWriteOnce`: 볼륨을 단일 노드에서 읽기-쓰기로 마운트.
  - `ReadOnlyMany`: 볼륨은 여러 노드에서 읽기 전용으로 마운트.
  - `ReadWriteMany`: 볼륨은 여러 노드에서 읽기-쓰기로 마운트.
- `spect.capacity.storage` 를 통해 용량을 정의할 수 있다.

```bash
$ kubectl create -f pv-d

$ kubectl get persistentvolume
```