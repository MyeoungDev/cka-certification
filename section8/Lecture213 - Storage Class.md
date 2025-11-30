# Lecture 213 - Storage Class

- 기존에는 관리자가 PV 와 PVC 를 수동으로 생성하여 파드에 마운트했다.
- Storage Class 를 이용한 Dynamic Provisioning 을 통해 쿠버네티스 스토리지 관리의 효율성을 높일 수 있다.

## Static Provisioning

- Static Provisioning 을 사용하면 기본 스토리지를 수동으로 생성한 다음 해당 디스크를 참조하는 PV 를 구성한다.
- 애플리케이션에 스토리지가 필요할 때마다 디스크를 프로비저닝하고 해당 PV 정의를 생성해야 한다.

예를 들어, Google Cloud에 영구 디스크를 만들려면 다음과 같이 진행한다.

```bash
gcloud beta compute disks create \\
  --size 1GB \\
  --region us-east1 \\
  pd-disk

```

```bash
# pv-definition.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 500Mi
  gcePersistentDisk:
    pdName: pd-disk
    fsType: ext4

```

```bash
# pvc-definition.yaml
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
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
    - image: alpine
      name: alpine
      command: ["/bin/sh", "-c"]
      args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
      volumeMounts:
        - mountPath: /opt
          name: data-volume
  volumes:
    - name: data-volume
      persistentVolumeClaim:
        claimName: myclaim

```

## Storage Class를 사용한 Dynamic Provisioning

- Dynamic Provisioning 을 사용하게 되면 수동으로 사전에 Provisioning 작없이 필요 없다.
- PVC 를 생성하면 연결된 Storage Class 가 정의된 Provisioner 를 사용하여 필요한 PV 를 자동으로 프로비저닝한다.

예시, Google Cloud 디스크를 사용한 Dynamic Provisioning

```bash
# sc-definition.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd

```

```bash
# pvc-definition.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: google-storage
  resources:
    requests:
      storage: 500Mi

```

```bash
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
    - image: alpine
      name: alpine
      command: ["/bin/sh", "-c"]
      args: ["shuf -i 0-100 -n 1 >> /opt/"]
      volumeMounts:
        - mountPath: /opt
          name: data-volume
  volumes:
    - name: data-volume
      persistentVolumeClaim:
        claimName: myclaim

```

- Storage Class 로 PVC 를 생성하면 쿠버네티스는 정의된 Provisioner 를 활용하여 요청된 크기의 새로운 Persistent Disk 를 동적으로 생성하고, 자동으로 PV 를 생성하여 PVC 에 바인딩한다.
- Dynamic Provisioning 을 사용하면 수동 작업이 줄어들고, 잠재적인 오류가 최소화되어 스토리기 관리가 간소화된다.

## 스토리지 클래스 사용자 정의

- 쿠버네티스의 Storage Class는 다양한 매개변수를 지원하여 애플리케이션의 성능 및 안정성 요구 사항에 맞춰 프로비저닝된 스토리지를 미세 조정할 수 있다.
- 많은 프로비저너가 사용자 지정 매개변수를 지원한다.
- 이를 통해 PVC 정의에서 적절한 스토리지 클래스를 지정하면 애플리케이션의 요구 사항에 맞게 스토리지의 성능과 안정성을 조절할 수 있다.
- 예를 들어, GCE 프로비저너를 사용하면 디스크 유형과 복제 모드를 지정할 수 있다.
- 이를 통해 실버(표준 디스크), 골드(SSD 드라이브), 플래티넘(지역 SSD 드라이브)과 같은 여러 서비스 클래스를 생성할 수 있다.

```bash
# silver storage class: standard disk without replication
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: silver
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
  replication-type: none

```

```bash

# gold storage class: SSD disk without replication
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gold
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  replication-type: none

```

```bash
# platinum storage class: SSD disk with regional replication
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: platinum
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  replication-type: regional-pd

```