# Lecture 129 - (2025 Updates) Introduction to Autoscaling

## 수직적 확장 (Vertical scaling)

- 기존 서버의 스펙을 업그레이드
- 단일 서버의 리소스를 강화
- 기존 노드의 리소스(CPU, MEM) 를 늘리는 것.
- 기존 Pod dp 에 대한 리소스에 대한 `limit`, `request` 상향 조정.

## 수평적 확장 (Horizontal scaling)

- 서버를 추가하여 통합.
- 클러스터에 더 많은 노드를 추가하는 것.
- 더 많은 Pod를 생성.

## 수동 스케일링 (Manual Scaling)

- 직접 명령어을 입력해서 확장하는 방식.

### 클러스터 인프라 수평 확장.

- 수동으로 새 노드를 직접 추가하는 방법.

```bash
kubeadm join ...
```

### 워크로드 수평 확장

- 애플리케이션의 파드 수를 늘리는 것.

```bash
$ kubectl scale --replicase=<number> <workload-type>/<workload-name>
```

### 워크로드 수직 확장.

- 애플리케이션 파드 자체의 자원(CPU, MEM) 을 늘리는 것.

```bash
$ kubectl edit <workload-type>/<workload-name>
```

## 자동 스케일링 (Automated Scaling)

- Auto Scaling 은 시스템이 정해진 규직에 따라 자동으로 확장 축소하는 방식.
- 수동으로 관리하는 복잡성을 줄임.

### 클러스터 인프라

- 클러스터 인프라 오토스케일러(Cluster Autoscaler)가 관리.
- 워크로드를 실행할 충분한 자원이 없을 떄 자동으로 노드를 추가하고, 자원이 남을 때는 노드를 제거.

### 워크로드 수평 확장

- 수평 파드 오토스케일 (HPA, Horizontal Pod Autoscaler) 가 관리.
- HPA 는 파드의 리소스 사용량 측정 지표에 따라 파드 수를 자동으로 늘리거나 줄인다.

### 워크로드 수직 확장

- 수직 Pod 오토스케일러(VPA: Vertical Pod Autoscaler)가 관리.
- VPA는 Pod의 과거 사용량을 분석해서 가장 적절한 자원(CPU, 메모리) 요청 및 한도를 자동으로 설정.