# Lecture 130 - (2025 Updates) Horizontal Pod Autoscaler (HPA)

## 애플리케이션 수동 확장.

- 수동 확장에는 지속적인 모니터링과 시기적절한 개입이 필요한데, 이는 예상치 못한 트래픽 급증 시에는 적합하지 않을 수 있다.

```bash
# Pod 리소스 사용량 확인
$ kubectl top pod my-app-pod

NAME         CPU(cores)   MEMORY(bytes)
my-app-pod   450m         350Mi

$ vi my-app.yaml

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
          resources:
            requests:
              cpu: "250m"
            limits:
              cpu: "500m"

$ kubectl scale deployment my-app --replicas=3

```

## 수평 파드 자동 확장기(HPA, Horizontal Pod Autoscaler)

- 수동 확장의 단점을 해결하기 위한 솔루션.
- HPA는 `metrics-server`를 사용하여 CPU, 메모리 또는 사용자 지정 지표와 같은 파드 지표를 지속적으로 모니터링 한다.
- 이러한 지표를 기반으로 HPA는 배포, 상태 저장 세트 또는 복제본 세트의 파드 복제본 수를 자동으로 조정한다.
- 리소스 사용량이 사전 설정된 임계값을 초과하면 HPA는 파드 수를 늘리고, 사용량이 감소하면 리소스를 절약하기 위해 축소한다.

```bash
$ kubectl autoscale deployment my-app --cpu-percent=50 --min=1 --max=10

```

- 쿠버네티스는 `metrics-server` 를 통해 CPU 메트릭을 모니터링하는 HPA 를 생성한다.
- 평균 CPU 사용률이 50%를 초과하면 HPA 는 자동으로 Replica 수를 조정한다.

## HPA 사용

```bash
$ vi my-app-hpa.yaml

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50

$ kubectl get hpa

$ kubectl delete hpa my-app

```