# Lecture 18 - Kube Controller Manager

## Kube Controller Manager

- 클러스터의 복원력과 안정성을 유지하는 데 중요한 이 컴포넌트의 역할과 구성을 이해하는게 중요하다.

## KubeController Manager의 개요

- 클러스터 내의 다양한 **컨트롤러(Controler)** 들을 관리하는 단일 프로세스이다.
- 조직의 부서처럼 각 컨트롤러는 **특정 업무를 담당**하며, **시스템의 변경 사항을 지속적으로 관찰**하여 k클러스터를 **“의도한 상태” 로 유지**한다.
- Depoly, Service, Namespace, Volume 등 모든 핵심 쿠버네티스 구성 요소는 컨트롤러에 의존한다.
- 기본적인 컨트롤러는 쿠버네티스 클러스터의 여러 작업의 “두뇌” 역할을 한다.
- 예시
    - 노드 콘트롤러(Node Controller)
        - `Kube API Server` 를 통해 노드 상태를 모니터링.
        - 노드의 하트비트가 멈추더라도 즉시 `NotReady` 로 표시되지 않고, 유예 기간 (40초) 와 복구 시간 (5분)을 거친 후 파드 재스케줄링.
    - 복제 컨트롤러(Relication Controller)
        - 지정된 수의 파드가 항상 실행중인지 확인하여, 필요할 때 새 파드를 생성하여 지정된 수 의 파드를 유지 관리.
        - 쿠버네티스 클러스터의 복원려과 안정성 강화

    ```bash
    $ kubectl get nodes

    NAME         STATUS   ROLES    AGE   VERSION
    worker-1     Ready    <none>   8d    v1.13.0
    worker-2     Ready    <none>   8d    v1.13.0

    # 노두
    $ kubectl get nodes
    NAME         STATUS     ROLES    AGE   VERSION
    worker-1     Ready      <none>   8d    v1.13.0
    worker-2     NotReady   <none>   8d    v1.13.0
    ```

    ## 컨트롤러 패키징 방법

    - `Kube Controller Manager` 는 모든 개별 컨트롤러를 하나의 단일 프로세스로 통합하여 관리의 효율성을 높인다.
    - `Kube Controller Manager` 를 배포하면 모든 관련 컨트롤러가 함께 시작된다.

    ```bash
    $ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-controller-manager

    $ vi /etc/system/systemd/kube-controller-manager.service

    ExecStart=/usr/local/bin/kube-controller-manager \
        --address=0.0.0.0 \
        --cluster-cidr=10.200.0.0/16 \
        --cluster-name=kubernetes \
        --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \
        --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \
        --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \
        --leader-elect=true \
        --root-ca-file=/var/lib/kubernetes/ca.pem \
        --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \
        --service-cluster-ip-range=10.32.0.0/24 \
        --use-service-account-credentials=true \
        --v=2
    ```

- 노드 컨트롤러에 대한 추가 옵션 (노드 모니터링 기간, 유예 기간, 제거 시간 등)이 포함되어 있다.
- `--controllers` 플래그를 통해 활성화된 컨트롤러를 제어할 수 있다.
- 기본적으로 모든 컨트롤러는 활성화 되어 있다.

## Kube Controllelr Manager 작동 모습

- `kubeadm` 기반 클러스터의 경우 `kube-system` 네임스페스 내의 파드(Pod) 로 실행된다.
- `/etc/kubernetes/manifests` 디렉토리에서 파드 정의 파일을 확인 할 수 있다.
- `kubeadm` 기반 클러스터가 아닐경우 `/etc/systemd/system/kube-controller-manager.service` 를 통해 데몬으로 실행된다.

![frame_150.jpg](attachment:ed384960-8c38-4cb0-9c11-8f6a6f480a4d:frame_150.jpg)