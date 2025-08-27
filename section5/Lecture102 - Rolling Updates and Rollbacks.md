# Lecture 102 - Rolling Updates and Rollbacks

## Rollout and Versioning

- 쿠버네티스는 Deployment 를 새성하면 첫 번째 배포 Revision 1 을 설정하는 Rollout 을 시작한다.
- 애플리케이션을 업데이트하면 쿠버네티스는 또 다른 Rollout Trigger 하여 새로운 Reivison 2 를 생성한다.
- 이러한 Revision 은 변경 사항을 추적하고 문제 발생 시 이전 버전으로 롤백하는데 도움이 된다.
- Kubernetes Deployment 를 최소한의 가동 중지 시간과 업데이트 및 롤백에 대한 안정적인 프로세스를 보장할 수 있디.

```bash

### 롤아웃 상태를 확인
$ kubectl rollout status deployment/myapp-deployment

### 롤아웃 내역
$ kubectl rollout history deployment/myapp-deployment

```

## Deployment Strategy

### Recreate Strategy

- 새 인스턴스를 배포하기 전에 기존 인스턴스를 모두 종료하는 전략
- 업데이트 중에 애플리케이션에 액세스할 수 없게 되어 일시적인 다운타임이 발생
- Deployment 의 `Events` 를 확인하면 애플리케이션 변경 사항을 반영하기 이전에 ReplicaSet 이 0 으로 되는 것을 확인 가능.

```bash

$ kubectl apply -f myapp-deployment.yaml

$ kubectl descirbe deployment myapp-deployment.yaml

...
Events:
  Type    Reason             Age   From                    Message
  -----   ------             ----  ----                    -------
  Normal  ScalingReplicaSet  11m   deployment-controller  Scaled up replica set myapp-deployment-6795844b58 to 5
  Normal  ScalingReplicaSet  11m   deployment-controller  Scaled down replica set myapp-deployment-6795844b58 to 0
  Normal  ScalingReplicaSet  56s   deployment-controller  Scaled up replica set myapp-deployment-54c7d6ccc to 5

```

### Rolling Update Strategy

- 인스턴스가 한 번에 하나씩 업데이트되어 프로세스 전반에 걸쳐 지속적인 애플리케이션 가용성을 보장
- 배포를 생성할 때 전략을 지정하지 않으면 Kubernetes는 기본적으로 롤링 업데이트 전략

```bash

$ kubectl apply -f myapp-deployment.yaml

$ kubectl descirbe deployment myapp-deployment.yaml

...
Events:
  Type    Reason             Age   From                    Message
  -----   ------             ----  ----                    -------
  Normal  ScalingReplicaSet  1m    deployment-controller   Scaled up replica set myapp-deployment-67c749c58c to 5
  Normal  ScalingReplicaSet  1m    deployment-controller   Scaled down replica set myapp-deployment-75d7bdbd8d to 2
  Normal  ScalingReplicaSet  1m    deployment-controller   Scaled up replica set myapp-deployment-67c749c58c to 4
  Normal  ScalingReplicaSet  1m    deployment-controller   Scaled down replica set myapp-deployment-75d7bdbd8d to 3
  Normal  ScalingReplicaSet  0s    deployment-controller   Scaled down replica set myapp-deployment-75d7bdbd8d to 1
  Normal  ScalingReplicaSet  0s    deployment-controller   Scaled down replica set myapp-deployment-67c749c58c to 0

```

## Upgrade and Rollback

### Upgrade

- 업그레이드 중에 쿠버네티스는 업데이트된 컨테이너에 대한 새로운 ReplicaSet을 생성한다.
- 또한, 기존 ReplicaSet은 이전 버전을 계속 실행한다.
- 이러한 롤링 업데이트 프로세스는 다운타임 없이 새 파드가 이전 파드를 점진적으로 대체하도록 보장한다.

### Rollback

- 업그레이드 후 문제가 발견되면 롤백 기능을 사용하여 이전 버전으로 되돌릴 수 있다.

```bash

$ kubectl rollout undo deployment/myapp-deployment

$ kubectl get replicasets

NAME                                 DESIRED   CURRENT   READY   AGE
myapp-deployment-67c749c58c          0         0         0       22m
myapp-deployment-7d57dbd8d           5         5         5       20m

```