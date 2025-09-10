# Lecture 145 - Demo Cluster Upgrade

- Practice Test 를 진행하면서 클러스터 업그레이드 과정 정리. 아래 두 문서 그대로 따라할 것.
    - https://v1-33.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
    - https://v1-33.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/change-package-repository/#verifying-if-the-kubernetes-package-repositories-are-used

## apt 패키지 도구 업데이트

https://v1-33.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/change-package-repository/#verifying-if-the-kubernetes-package-repositories-are-used

```bash
$ vim /etc/apt/sources.list.d/kubernetes.list

# 해당 kubernetes.list 파일 내용 아래로 변경
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] <https://pkgs.k8s.io/core:/stable:/v1.33/deb/> /

$ apt upgrade

# 설치 가능한 kubeadm 패키지  버전 확인
$ apt-cache madison kubeadm

```

- apt 패키지 관리자는 sources.list.d 파일에 정의된 저장소(repository) 에서 패키지 목록을 가져온다.
- 만약, 해당 저장소 URL 을 변경하지 않고 강제로 v1.33 설치를 진행할 경우 `Unable to locate package` 오류가 발생할 수 있다.
- 따라서, 해당 저장소 URL 을 변경하고 `apt-cache madison kubeadm` 명령어로 캐시에 저장된 kubeadm 패키지 버전까지 확인하면 좋다.

## kubeadm 업그레이드

```bash

# kubeadm 패키지 설치
$ apt-get install kubeadm=1.33.0-1.1

# 업그레이드 계획 확인
$ kubeadm upgrade plan v1.33.0

# 컨트롤 플레인 컴포넌트 업그레이드 진행.
$ kubeadm upgrade apply v1.33.0

```

## kubelet 업그레이드

```bash

$ apt-get install kubelet=1.33.0-1.1

$ systemctl daemon-reload

$ systemctl restart kubelet

```

- `kubelet` 의 경우 Linux 의 System Daemon 으로 관리됨.
- 따라서, `apt-get` 을 통해 새로운 패키지 버전으로 변경 후 `daemon-reload`, `restart kubelet` 진행.