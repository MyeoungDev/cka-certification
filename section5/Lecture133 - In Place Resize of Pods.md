# Lecture 133 - In place Resize of Pods 2025 Updates

## In-Place Update Mechanism

- 리소스 요구 사항을 업데이트하면 쿠버네티스는 기존 Pod를 종료하고 업데이트된 리소스 사양을 사용하여 새 Pod를 생성한다. 이러한 동작으로 인해 일시적인 서비스 중단이 발생할 수 있습니다.
- `In-Place Update Mechansim` 은 전체 파드를 다시 생성하지 않고도 리소스 변경 사항에 대한 업데이트를 간소화한다.
- 파드 리소스 업데이트에 대한 다운타임을 줄이며 상태관리에 효과적이다.
- 쿠버네티스 1.27 버전부터 알파 버전으로 제공되었으며 기본적으로 활성화되어 있지 않다.
- 베타 버전으로 전환되고, 개발이 완료됨에 따라 궁극적으로 기본적으로 활성화될 것으로 예상된다.
    - https://kubernetes.io/blog/2025/05/16/kubernetes-v1-33-in-place-pod-resize-beta/
    - kubernetes v1.33 버전에 베타버전 출시 후 기본적으로 활성화 되었다는 것을 확인 가능. (2025-05-16)

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: nginx
        resizePolicy:
          - resourceName: cpu
            restartPolicy: NotRequired
        resources:
          requests:
            cpu: "250m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"

```

### In-Place Update Mechanism 활성화

```bash
$ FEATURE_GATES=InPlacePodVerticalScaling=true

```

- 활성화되면 각 리소스의 재시작 동작을 제어하기 위해 추가적인 크기 조정 정책 매개변수를 지정할 수 있다.
- 정책을 통해 CPU 리소스를 업데이트해도 Pod 재시작이 발생하지 않도록 할 수 있지만, 메모리를 업데이트하는 경우에는 Pod 재시작이 필요할 수 있디.
- (강의 시점) 현재 내부 크기 조정 기능은 CPU와 메모리 리소스만 지원한다.

## In-Place Update Mechanism 한계

- CPU와 메모리 리소스만 그 자리에서 업데이트할 수 있다.
- Pod QoS 클래스 및 특정 다른 속성에 대한 변경은 지원되지 않는다.
- Init 컨테이너와 임시 컨테이너는 내부 크기 조정에 적합하지 않는다.
- 리소스 요청 및 제한은 한 컨테이너에 할당되면 다른 컨테이너로 이동할 수 없다.
- 컨테이너의 메모리 한도는 현재 사용량보다 낮아질 수 없다. 이러한 요청이 이루어지면 새 메모리 한도에 도달할 때까지 크기 조정 작업이 계속 진행된다.
- Windows Pod는 이 기능을 지원하지 않는다.