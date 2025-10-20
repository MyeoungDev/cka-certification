# Lecture 198 - Custom Controllers 2025 Updates

- Custom Controller 는 Cluster Resource 를 지속적으로 모니터링하고 변경 사항이 발생할 때 조치를 취한다.
- Custom Controller 는 클러스터 내 객체의 상태가 원하는 상태와 일치하는지 확인하다.
- 리소스 변경사항에 따라 특정 작업을 자동화하는데 필수적으로 사용된다.

## Controller Implementation in Go

- Kubernetes API 를 사용하여 Python 으로 Controller 를 구현 가능하다.
- 그러나, Kubernetes Go Client 를 사용하여 Go 로 컨트롤러를 구축하는 것이 더 강력하다.
- Go Client 는 캐싱 및 큐잉 매커니즘을 간소화하는 built-in shared informer 를 제공한다.
- 이를 통해 컨트롤러의 효율성과 관리 용이성이 향상된다.


```bash
$ git clone https://github.com/kubernetes/sample-controller.git

$ cd sample-controller

$ vi controller.go

package flightticket


var controllerKind = apps.SchemeGroupVersion.WithKind("Flightticket")


// Run begins watching and syncing.
func (dc *FlightTicketController) Run(workers int, stopCh <-chan struct{}) {}


// Call BookFlightAPIReplicaSet
func (dc *FlightTicketController) callBookFlightAPI(obj interface{}) {}

$ go build -o sample-controller .


$ ./sample-controller --kubeconfig=$HOME/.kube/config
```