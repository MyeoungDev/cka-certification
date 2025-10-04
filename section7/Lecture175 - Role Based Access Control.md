# Lecture176 - Role Based Access Control

## Create a Role

- 역할을 정의하기 위해선 `rbac.authorization.k8s.io/v1` API 버전으로 YAML 파일을 생성해야 한다.
- Role, Role Binding 모두 네임스페이스 범위이다.
- 다른 네임스페이스에서 액세스를 관리하려면 YAML 파일 `metadat`에 명시해야 한다.

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
```

```bash
kubectl create -f developer-role.yaml
```

## Create a RoleBinding

- Role 을 정의한 후 해당 Role 을 사용자에게 바인딩해야 한다.
- 역할 바인딩은 사용자를 특정 네임스페이스 내의 역할에 연결한다.

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl create -f devuser-developer-binding.yaml
```

## Role, RoleBinding 확인

```bash
$ kubectl get roles

NAME        AGE
developer   4s

$ kubectl get rolebindings

NAME                      AGE
devuser-developer-binding 24s

$ kubectl describe role developer

Name:         developer
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources           Non-Resource URLs   Resource Names   Verbs
  -----------         ------------------   --------------   ----
  ConfigMap           []                   []               [create]
  pods                []                   []               [get watch list create delete]
  
$ kubectl describe rolebinding devuser-developer-binding

Name:         devuser-developer-binding
Labels:       <none>
Annotations:  <none>
Role:
  Kind:    Role
  Name:    developer
Subjects:
  Kind     Name      Namespace
  ----     ----      ---------
  User     dev-user
```

## kubectl auth 로 권한 테스트

- 명령을 사용하여 특정 작업을 수행하는데 필요한 권한이 있는지 테스트할 수 있다.
- `--as` 플래스를 사용해 특정 사용자 권한을 확인할 수 있다.
- 네임스페이스 플래그를 이용해 특정 네임스페이스에 대해서 확인할 수 있다.

```bash
$ kubectl auth can-i create deployments

yes

$ kubectl auth can-i delete nodes

no

$ kubectl auth can-i create deployments --as dev-user

no

$ kubectl auth can-i cretae pods --as dev-user

yes
```

## 특정 리소스에 대한 액세스 제한

- 특정 리소스 그룹에 대한 사용자 액세스를 제한해야 할 경우가 있다.
- `resourceNames` 필드를 사용해서 액세스 제한을 할 수 있다.

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "create", "update"]
  resourceNames: ["blue", "orange"]
```

- 이를통해 `blue`, `orange` 파드에 대해서만 액세스를 제한한다.