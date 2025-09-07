# Lecture 139 - OS Upgrades

## OS Upgrades

- 노드가 다운되거나, 배포 전략에 다라 사용자에게 영향을 미칠 수 있다.
- 쿠버네티스는 이러한 상황을 처리하기 위해 노드가 다시 온라인 상태가 되면 파드를 다시 시작하낟.
- 노드가 5분 이상 다운 상태로 유지되면, 해당 노드의 파드를 `dead` 상태로 표시한다.
- `ReplicaSet` 에서 관리하는 파드의 경우, 다른 노드에 새 파드가 생성된다.
- 그러나, `ReplicaSet` 에 속하지 않는 파드ㅡㄴ 재시작 되지 않아 다운타임이 발생할 수 있다.

## **Draining a Node**

- 노드의 복구 시간이 불확실한 경우 노드를 Draining 하는 것이 더 안전하다.
- Draining 은 해당 노드에서 실행 중인 파드를 정상적으로 종료하여, 다른 노드에서 재생성되도록 하는 과정이다.
- 또한, Draining은 해당 노드를 스케줄링 불가 상태 `unschedulable` 로 표시하여 명시적으로 허용될 때까지 새로운 파드가 스케줄링 되지 않도록 한다.
- 노드가 다시 온라인 상태가 되면, `uncordon` 명령을 사용해서 노드를 복구해야 한다.
    - 드레이닝으로 다른 노드로 이동한 파드는 노드를 `uncordon` 해도 원래 노드로 자동으로 돌아오지 않는다.

```bash
# node-1 draining
$ kubectl drain node-1

# node-1 uncordon
$ kubectl uncordon node-1
```

## **Cordon vs. Drain**

### 공통점

- 두 명령어 모두 노드에 새로운 파드가 스케줄링되는 것을 막는다.

### 차이점

`cordon`

- `kubectl cordon node-1`
- 단순히 노드를 `unschedulable` 상태로만 만든다.
- 현재 노드에서 실행 중인 파드들은 그대로 유지된다.

`drain`

- `cordon` 기능에 더해, 현재 노드에서 실행 중인 파드들을 정상적으로 종료시키고, 다른 노드로 재배치한다.