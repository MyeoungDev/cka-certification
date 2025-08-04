# Lecture33 Certification Tip!

## 시험 꿀팁

- 시험 환경에서는 복사/붙여넣기가 어려울 수 있으므로, 이 방법이 매우 유용하다.
- `-dry-run=client -o yaml` 옵션을 사용하면 YAML 템플릿을 빠르게 생성하고, 이를 기반으로 필요한 수정을 한 후 kubectl create -f 명령으로 배포할 수 있어 매우 효율적

```bash

# 파드를 생성하지 않고 (--dry-run=client), YAML 파일(-o yaml)을 출력.
$ kubectl run nginx --image=nginx --dry-run=client -o yaml

# Deployment 생성하지 않고(--dry-run=client), YAML 파일(-o yaml)을 출력.
$ kubectl create deployment --image=nginx nginx --dry-run=client -o yaml

# Deployment YAML을 생성한 후 nginx-deployment.yaml 파일로 저장.
$ kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yml

```