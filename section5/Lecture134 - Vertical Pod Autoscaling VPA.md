# Lecture 134 - Vertical Pod Autoscaling VPA 2025 Updates

## Vertical Pod Autoscaler(VPA)

- 파드를 수동으로 수동으로 업데이트 하는 행위는 시간이 많이 걸리고 오류가 발생하기 쉽다.
- 쿠버네티스에서는 파드의 Vertical Pod Autoscaler(VPA) 를 제공하여 업데이트 프로세스 자동화를 제공한다.

## VPA 와 HPA 의 차이

- HPA
    - HPA 는 수요에 따라 파드를 추거하거나 제거 (변경사항에 따른 파드 재시작 X)
    - 더 많은 포드를 즉시 추가하여 급격한 트래픽 급증을 처리하는 데 이상적
    - 사용 사레: 빠른 확장이 필요한 상태 비저장 애플리케이션, 웹 서비스 및 마이크로서비스.
- VPA
    - VPA 는 파드의 CPU 및 메모리 할당을 자동으로 조정. (변경사항에 따른 파드 재시작 O)
    - 기본적으로 활성화되어 있지 않으므로 수동으로 설치 필요.
    - VPA는 명령형 명령을 통해 설정되지 않는다.
    - 재시작 지연으로 인한 갑작스러운 급증에는 효과가 떨어진다.
    - 사용 사례: 정확한 튜닝이 필요한 상태 저장 워크로드, 데이터베이스, JVM 기반 애플리케이션 및 AI 워크로드

## VPA 설치

```bash

$ kubectl apply -f <https://github.com/kubernetes/autoscaler/releases/latest/download/vertical-pod-autoscaler.yaml>

$ kubectl get pods -n kube-system | grep vpa
vpa-admission-controller-xxxx   Running
vpa-recommender-xxxx            Running
vpa-updater-xxxx                Running

```

- vpa-recommender
    - 쿠버네티스 메트릭 API 를 통해 리소스 사용량을 지속적으로 모니터링하고, CPU 및 메모리에 대한 최적화된 추천을 제공.
- vpa-updater
    - 현재 파드 리스소 설정을 권장 사항과 비교하고, 최적이 아닌 리로스로 실행 중인 파드를 제거. 제거 과정을 통해 업데이트된 구성을 가진 새 파드가 생성.
- vpa-admission-controller
    - 파드 생성 요청을 가로채고 recommender 에 따라 파드 사양을 변형하여 새로운 파드가 이상적인 리소스 구성으로 시작되도록 보장.

## VPA 정의

```bash
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
      - containerName: "my-app"
        minAllowed:
          cpu: "250m"
        maxAllowed:
          cpu: "2"
        controlledResources: ["cpu"]

```

`spec.updatePolicy.updateMode: "Auto"` 는 최적이 아닌 리소스로 실행되는 파드를 종료하고, 권장 값으로 새 파드를 생성할 수 있도록 한다.