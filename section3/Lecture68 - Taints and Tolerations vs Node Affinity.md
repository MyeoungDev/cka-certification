# Lecture 68 - Taints and Tolerations vs Node Affinity

## Taints and Tolerations vs Node Affinity

- 쿠버네티스에서 Taints, Tolerations, Node Affinity 를 모두 활용하면 정확한 파드 스케줄링을 보장할 수 있다.
- 이 접근 방식은 특히 노드의 독점적인 사용이 중요한 멀티 테넌트 클러스터에서 유용하다.

### Taints and Tolerations
- 쿠버네티스에서 Taints 와 Tolerations 은 Taints 를 명시적으로 허용하지 않는 한 노드에서 파드를 밀어내는 데 주로 사용된다.
- 쿠버네티스 스케줄러는 톨러레이션을 허용하는 노드에 파드를 배치한다.
- taint와 toleration은 일치하는 toleration을 가진 파드가 노드에 의해 허용되도록 보장하지만, 배타적 스케줄링을 보장하지는 않는다.
- 파드가 taint되지 않은 노드에 여전히 스케줄링 되어 원치 않는 배치가 발생할 수 있다.
-

### Node Affinity
- Affinity 는 특정 레이블 기준을 충족하는 파드를 유지하는 데 사용된다.
- Taints 와 Tolerations 의 한계를 극복하기 위해 Node Affinity 를 활용 한다.
- Node Affinity 는 파드가 일치하는 레이블을 가진 노드에만 도달하도록 보장합니다.
- Node Affinity 는 Pod를 올바른 노드로 안내하지만, 다른 Pod가 해당 노드에 예약되는 것을 제한하지는 않는다.
- 즉, 원하는 Pod가 올바르게 배치되었더라도 노드가 해당 Pod에 적합하지 않은 Pod를 호스팅할 수 있다.

## Taints, Tolerations 과 Node Affinity 결합

- 단독 노드 사용의 경우, 두 전략을 결합하는 것이 최적의 솔루션이다.
- 노드에 Tatins 를 적용하고, Pod 구성에서 해당 Toleration 을 지정하여 적절한 허용 범위가 없는 모든 파드를 차단한다.
- Node Affinity 규칙을 사용하여 각 Pod 가 일치하는 레이블이 있는 노드에만 예약되도록 한다.
- 이러면 결합된 접근 방식은 노드를 의도한 파드에만 전용으로 할당하여 올바른 파드 할당을 보장하고, 다른 작업 부하로 인한 간섭을 방지한다.