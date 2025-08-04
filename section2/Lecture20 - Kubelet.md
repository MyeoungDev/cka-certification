# Lecture 20 - Kubelet

## Kubelet 역할

- kubelet 은 쿠버네티스 클러스터 내의 각 워커 노드에서 실행되는 에이전트이다.
- 노드에서 이뤄지는 모든 활동을 지휘하는 "선장" 과 같다.
- 클러스터에 합류하기 위해 스스로를 쿠버네티스 클러스터에 노드로 등록한다.
- 마스터 노드(kube-apiserver) 와 유일한 통신체다.
- 클러스터의 파드와 컨테이너 상태를 지속적으로 모니터링하고, kube-apiserver 에 보고한다.
- 컨테이너 또는 파드 실행 명령을 수신하면, 컨테이너 런타임과 통신하여 필요한 이미지를 다운로드하고 컨테이너를 실행한다.

## Kubelet 설치

- 다른 쿠버네티스 구성 요소와 달리 Kubelet 은 `kubeadm` 과 같은 도구를 사용하여 클러스터를 설정할 떄 자동으로 배포되지 않는다.
- 각 워커 노드에 수동으로 설치해야 한다.

```
$ wget <https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kubelet>

ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2

# Process grep 확인
$ ps -aux | grep kubelet

...

```