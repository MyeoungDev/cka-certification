# Lecture 144 - Cluster Upgrade Process

## Cluster Upgrade Introduction

https://v1-33.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

- 핵심 control plane 구성 요소를 중심으로 업그레이드 프로세스 설명.
- 모든 구성 요소가 동일한 버전에서 실행될 필요는 없다.
- 각 구성 요소는 서로 다른 릴리즈 버전에서 동작할 수 있지만, `Kube API` 서버는 다른 모든 구성 요소가 통신하는 주요 제어 영역 구성 요소로 남아 있다.
- 따라서, 어떤 구성 요소도 API 서버보다 높은 버전에서 실행되어서는 안된다.
- 예를 들어, `Kube API` 버전이 1.10 일 경우 `Kubelet`, `Kube Proxy` 는 1.10 보다 하위 는 가능하지만 상위로 실행되서는 안된다.

## When to Upgrade

- 쿠버네티스는 공식적으로 최신 마이너 버전 3개까지 지원한다.
    - 예를 들어, 1.13 버전이 출시되면 1.13, 1.12, 1.11 까지 지원된다.
- 만약, 해당 마이너 버전보다 낮은 버전을 사용중인 경우 지원이 중단되기 때문에 중단되기 이전에 다음 릴리스로 업그레이드하는 것이 좋다.

## Upgrade Process

- 일반적인 업그레이드 프로세스는 마스터 노드 업그레이드 -> 워커 노드 업그레이드 이다.
- 마스터 노드 업그레이드 중에는 ControlPlane 구성 요소 (API Server, Scheduler, Controller Manager) 가 잠시 중단된다.
    - 워커 노드는 정상적으로 애플리케이션을 서빙할 수 있다.
    - 단, 이 기간 동안 파드의 장애 발생 시 재시작되지 않을 수 있다.
- 워커 노드의 업그레이드 프로세스는 아래 3가지가 존재한다.
    - 모든 워커 노드를 동시에 업그레이드 (다운타임 발생)
    - 한 번에 하나의 노드를 업그레이드.
    - 업데이트 된 새로운 노드를 추가하고, 기존의 노드를 해제하면서 새로운 노드로 마이그레이션

## Kubeadm 을 사용한 마스터 노드 업그레이드

- `kubeadm` 을 사용하면 클러스터 업그레이드 계획 및 실행이 간소화 된다.
    - `kubeadm upgrade plan`
- 클러스터 업그레이드를 시작하기 전에 `kubeadm` 도구 자체를 업그레이드해야 한다.
- `kubeadm`은 `kubelet` 업그레이드를 관리하지 않는다.
- 따라서, 각 노드에서 `kubelet`을 수동으로 업그레이드 해야 한다.

```bash

# OS 배포 버전 확인.
$ cat /etc/*release*

# kubeadm 버전 확인.
$ kubeadm version

$ kubeadm upgrade plan

$ apt-get upgrade -y kubeadm=1.12.0-00

$ kubeadm upgrade apply v1.12.0

# Output:
# [upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.12.0". Enjoy!
# [upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.

$ kubeadm version

```

## 워커 노드 업그레이드

- 워커 노드 업그레이드는 한 번에 하나씩 수행해야 한다.
- 권장되는 방법은 다른 노드에서 애플리케이션을 계속 사용할 수 있도록 워커 노드 하나를 비우는 것이다.
- 따라서, `drain` 을 이용해서 워커 노드를 차단하고 파드를 안전하게 제거 후 사용해야 한다.
- 아래 방식을 일관되게 각 노드에서 진행해서. 각 노드가 안전하게 업그레이드 될 수 있도록 해야 한다.

```bash
$ kubectl drain node-1

# replace x in 1.33.x-* with the latest patch version
$ sudo apt-mark unhold kubelet kubectl && \\
$ sudo apt-get update && sudo apt-get install -y kubelet='1.33.x-*' kubectl='1.33.x-*' && \\
$ sudo apt-mark hold kubelet kubectl

$ sudo systemctl daemon-reload
$ sudo systemctl restart kubelet

$ kubectl uncordon node-1

```