# Lecture 142  - Kubernetes Releases

## Cluster Version

- 쿠버네티스 클러스터를 설치하면 특정 버전의 쿠버네티스가 설치된다.

```bash
$ kubectl get nodes

kubectl get nodes
NAME     STATUS   ROLES    AGE    VERSION
master   Ready    master   1d     v1.11.3
node-1   Ready    <none>   1d     v1.11.3
node-2   Ready    <none>   1d     v1.11.3
```

- **Major versions: 상당한 변화**
- **Minor versions:  몇달 주기로 새로운 기능과 특성**
- **Patch versions: 중요한 버그 수정을 위한 자주 변경**
- **Alpha releases : 비활성화 된 기능들이 존재. 불안정 버전**
- **Beta releases: 철저한 테스트가 진행된 단계.**
- **Stable releases: 모든 테스트가 통과된 후 프로덕션 사용할 준비가 된 안전한 단계.**