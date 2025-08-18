# Lecture 76 - Static Pod

## Static Pod

- kubelet 과 컨테이너 런타임이 쿠버네티스 클러스터 없이 호스트에 직접 설치되면 kubelet이 노드를 독립적으로 관리할 수 있다.
- 이 경우, 파드 정보를 제공하는 API 서버가 없으므로 파드 정의 파일을 kubelet에 직접 제공해야 한다.
- 정적 파드(Static Pod) 는 API 서버나 다른 Control Plane 구성 요소의 개입 없이 kubelet에 의해 직접 생성된다.
- 정적 파드는 Control Plane 구성 요소 자체를 배포하는 데 특히 유용하다.
- kubelet 은 호스트에서 파드 정의 파일이 지정된 디렉토리를 모니터링하도록 구성된다.
- kubelect 은 디렉토리를 주기적으로 검사하고, 사용 가능한 파일을 읽고, 해당 파드를 생성, 모니터링, 실행 상태 유지를 진행한다.
- kubelet에 의해 단독으로 생성되는 이러한 파드를 정적 파드라고 한다.

## Static Pod Directory

- 호스트의 모든 디렉터리에 정적 파드를 배치할 수 있다.
- `-pod-manifest-path` 옵션을 사용한다.

```bash
ExecStart=/usr/local/bin/kubelet \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --pod-manifest-path=/etc/kubernetes/manifests \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2

```

- 또는, `-config` 옵션을 사용해서 메니페스트 디렉토리 경로를 포함하는 구성파일을 지정할 수 있다.

```bash
ExecStart=/usr/local/bin/kubelet \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --config=kubeconfig.yaml \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2

$ vi kubeconfig.yaml
staticPodPath: /etc/kubernetes/manifests

```

- 이를 통해서 생성된 파드는 컨테이너 런타임 명령어를 통해서 확인이 가능하다.
- 독립 실행 시나리오에서 kube-apiserver 를 통해 Kubernetes API 요청을 처리할 수 없기 때문이다.

## 클러스터의 경우 동작

- 노드가 쿠버네티스 클러스터에 속하면 kube-apiserver 는 kubelet 에게 HTTP API 엔드포인트를 통해 파드를 생성하도록 지시한다.
- 이 경우 kubelet 은 정적 파드 디렉터리와 API 서버 모두에게 제공되는 파드 정의를 처리한다.
- 또한, kubelet 은 정적 파드를 생성할 때마다 kube-apiserver 에 미러 객체도 생성한다.
- 이 미러 객체는 읽기 전용이며 kubectl 을 통해 수정하거나 삭제할 수 없다.

## Static Pod vs DaemonSets

### Static Pod

- 생성 주체
    - kubelet 에서 직접 관리.
- Control Plane
    - API 서버와 상호작용 하지 않음.
- 사용 사례
    - 일반적으로 중요한 Control Plane 구성 요소에 사용.

### DaemonSet

- 생성 주체
    - DaemonSet 컨트롤러에서 관리
- Control Plane
    - kube-apiserver 통신 필요.
- 사용 사례
    - 모든 노드에서 Pod 사본이 실행되도록 보장.