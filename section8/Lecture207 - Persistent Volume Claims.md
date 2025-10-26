# Lecture 207 - Persistent Volume Claims

## Persistent Volume Claim 

- Persistent Volume 과 Persistent Volume Claim 은 서로 다룬 객체이다.
- 관리자는 PV 를 생성하고, 사용자는 PVC 를 생성하여 스토리지 리소스를 요청한다.
- 즉, PVC 는 스토리지를 요청하는 객체, PV 는 실제 스토리지 리소스를 나타낸다.
- 쿠버네티스는 PVC 가 생성될 때, PVC 의 요구사항(용량, 접근 모드 등)에 맞는 PV 를 자동으로 찾아 바인딩한다.
- 만약, 여러 PV 가 조건을 만족한다면 `labels`, `selectors` 를 사용해 특정 PV 와 명시적으로 매칭할 수 있다.
- 반대로 PVC의 요구조건을 만족하는 PV가 없으면, 해당 PVC 는 `Pending` 상태가 되고 적절한 PV 가 나중에 클러스터에 추가될 떄 자동으로 바인딩된다.
- PVC 가 요청한 용량보다 큰 PV에 연결될 수도 있다.
- 이 경우, 남은 공간은 다른 PVC 들이 사용할 수 없다.
- 즉, 하나의 PV 는 오직 하나의 PVC 에만 바인딩 될 수 있다.

### Persistent Volume Claim 생성

```bash
$ vi pvc-definition.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

```bash
$ kubectl create -f pvc-definition.yaml
```

```bash
kubectl get persistentvolumeclaim
NAME      STATUS   VOLUME   CAPACITY   ACCESS MODES
myclaim   Pending
```

- 쿠버네티스는 사용 가능한 PV 를 찾는다.
- PVC 의 조건에 만족되는 PV 가 존재하지 않을 경우 `Pending` 상태로 대기한다.
- 아래와 같이 PV 를 생성해주게 되면 성공적으로 바인딩 된다.

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-voll
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

## PVC 삭제 및 Persistent Volume Reclaim Policy 

```bash
$ kubectl delete persistentvolumeclaim myclaim
```

- PVC 가 삭제되면 Reclaim Policy 회수 정책에 따라 달라진다.
- `persistentVolumeReclaimPolicy` 옵션을 통해 설정할 수 있다.

- Retain : PVC 가 삭제된 후에도 PV 는 클러스터에 유지. 관리자가 수동으로 삭제해야 함.
- Delete : PV 는 PVC 와 함께 삭제되어 물리적 장치의 저장 공한 해제.
- Recycle : PV 데이터는 새로운 Claim 에 의해 재사용되기 전에 삭제된다. 
    - Recycle 정책은 최신 쿠버네티스 버전에서는 더 이상 사용되지 않는다.
