# Lecture 178 - Cluster Roles

## Cluster Role

- 네임스페이스를 지정하지 않고 Role, RoleBinding 을 생성하면 기본 네임스페이스에 추가되고 해당 범위 내에서만 액세스 권한을 부여한다.
- 이 방식은 Pod, Deployment, Service 와 같은 네임스페이스 리소스에는 적합하지만, 클러스터 범위에는 적합하지 않다.
- Node 및 Persistent Volume 과 같은 클러스터 범위 리소스는 클러스터 수준에서 관리되므로 다른 접근 방식이 필요하다.
- 대부분의 리소스는 네임스페이스가 지정된다.
- 반면, Node, Persistent Volume 을 포함한 클러스터 범위 리소스는 어떤 네임스페이스에도 속하지 안흔다.

```bash
$ kubectl api-resources --namespaced=ture

$ kubectl api-resources --namespaced=false
```

- 위 명령어를 사용하여 네임스페이스가 지정된 리소스와 지정되지 않은 리소스를 알 수 있다.

## 리소스 범위 이해

- Role, RoleBinding 은 네임슾이스 수준 액세스 이상적이다.
- Cluster Role, RoleBinding 은 클러스터 전체에 걸쳐 권한을 지정한다.

## Cluster Role and Cluster RoleBinding

- Node 및 PV 같은 클러스터 범위 리소스를 승인하려면 `ClusterRole`, `ClusterRoleBinding` 을 생성해야 한다.
- 이는 전체 클러스터에 걸쳐 수행되는 작업에 맞게 조정된다.
- 예시
  - Node List, Create, Delete 권한을 부여하는 관리자 역할 정의
  - PV, PV Claim 관리를 위한 Storage Admin 역할 정의
- ClusterRole 은 네임스페이스에 한정되지 않고, 클러스터 전체에 걸쳐 권한을 부여하는 방식이다.
- 즉, 클러스터 역할을 특정 사용자나 서비스 계정에 바인딩하면(ClusterRoleBinding 사용), 그 계정은 모든 네임스페이스의 파드 등 리소스에 접근할 수 있다.

### ClusterRole YAML
```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list", "get", "create", "delete"]
```

### ClusterBinding YAML
```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
- kind: User
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io
```