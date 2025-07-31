# Lecture 13 - A note on Docker deprecation

## Docker 지원 중단에 대한 오해

- 원래 Docker는 쿠버네티스에서 지원되는 유일한 컨테이너 런타임
- 다른 컨테이너 런타임과의 호환성을 위해 컨테이너 런타임 인터페이스(CRI)가 도입
- Containerd는 이제 CRI와 호환되며 Kubernetes와 직접 상호 작용하는 독립형 구성 요소로 작동
- 이러한 변경 덕분에 Kubernetes는 컨테이너 작업을 기본적으로 관리하므로 더 이상 Docker의 추가 도구에 의존하지 않는다.
- 기존 Docker 명령을 `nerdctl` 명령으로 대체하여 동일한 컨테이너 작업 가능.
- `containerd` 는 CRI 호환이 가능하며, 이제 Docker 와 별도로 독립적인 런타임으로 쿠버네티스와 동작