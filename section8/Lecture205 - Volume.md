# Lecture 205 - Volume

## Docker의 볼륨 개요

- Docker 컨테이너는 일시적인 상태로 되었다.
- 필요에 따라 생성되어 데이터를 처리한 후 삭제된다.
- 따라서, 컨테이너가 제거되면 컨테이너에 저장된 모든 데이터가 손실됩니다.
- 이러한 한계를 극복하기 위해 Docker는 컨테이너에 볼륨을 연결하여 컨테이너 수명 주기가 종료된 후에도 데이터가 유지되도록 할 수 있다.

## 쿠버네티스의 볼륨

- 마찬가지로, 쿠버네티스 파드는 설계상 수명이 짧다.
- 파드가 데이터를 처리한 후 삭제되면 파드 내부의 데이터는 일반적으로 손실된다.
- 이 데이터를 보존하기 위해 파드에 볼륨이 연결된다. 
- 파드에서 생성된 모든 데이터는 볼륨에 저장되며 파드가 종료된 후에도 계속 사용할 수 있다.

## 볼륨 사용의 간단한 예

- 파드가 0에서 100 사이의 난수를 생성하여 파드의 `/opt/number.out` 파일에 기록하는 단일 노드 클러스터.  
- 볼륨이 없으면 파드가 삭제될 때 이 파일이 손실된다. 
- 호스트의 디렉터리를 저장소로 사용하는 볼륨을 연결하여 숫자 파일 보존.
- 볼륨은 Host 의 `/data` 디렉토리 사용.
- 파드가 삭제되어도 파일은 호스트에 유지된다.

```bash
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
    hostPath:
      path: /data
      type: Directory
```

## 볼륨 저장 옵션

- 각 노드의 `/data` 디렉토리가 다를 수 있는 다중 노드 클러스터 방식에는 `hostPath` 볼륨이 적절하지 않다.
- 쿠버네티스는 여러 외부 및 복제 스토리지 솔루션을 지원한다.
  - NFS 
  - GlusterFS
  - Flocker
  - Fibre Channel
  - CephFS
  - ScaleIO
  - AWS EBS, Azure Disk or File, and Google Persistent Disk 와 같은 Public Cloud Storage Solution
