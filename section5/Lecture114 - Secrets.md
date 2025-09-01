# Lecture 114 - Secrets

## Kubernetes Secrets

- Kubernetes Secrets 는 데이터를 인코딩하여 민감한 정보를 안전하게 저장하는 메커니즘을 제공한다.
- `Secrets` 는 `Base64` 를 사용하여 데이터를 인코딩한다.
    - Base64 인코딩은 데이터를 전송 가능한 텍스트 형식으로 변환하는 방식 이지 암호화가 아니다.
    - 누구나 인코딩, 디코딩을 손쉽게 할 수 있다.
    - 따라서, 공개된 저장소에 `Secret` 파일을 관리해서는 안된다.
    - 보안 강화를 위해서는 `etcd` 에 데이터 암호화를 활성화 해야 한다.
    - 추가로, RBAC 를 이용하여 `Secret` 에 대한 접근을 제한 해야한다

```bash

# Encode Value

DB_Host: bXlzcWw=
DB_User: cm9vdA==
DB_Password: cGFzd3Jk

# Decode Value

DB Host: mysql
DB User: root
DB Password: paswrd
```

## **Imperative Secret**

- 명령형을 통해 `Key-Value` 의 `Secret` 을 직접 주입할 수 있다.

```bash
kubectl create secret generic app-secret \
  --from-literal=DB_Host=mysql \
  --from-literal=DB_User=root \
  --from-literal=DB_Password=paswd
```

```bash
kubectl create secret generic app-secret --from-file=app_secret.properties
```

## **Declarative Secret**

- 매니페스트 파일을 통해 Secret 을 관리할 수 있다.

```bash
$ vi secret-data.yaml

apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: bXlzcWw=
  DB_User: cm9vdA==
  DB_Password: cGFzd3Jk
  
  
$ kubectl create -f secret-data.yaml

$ kubectl get secrets

$ kubectl describe secret app-secret

```

## Pod 에 Secret 주입

### `envFrom` 을 통한 환경변수로 주입

```bash
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
    - containerPort: 8080
    envFrom:
    - secretRef:
        name: app-secret
```

### Pod 내부에 Volum 마운트 사용

```bash
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
    - containerPort: 8080
		volumes:
		- name: app-secret-volume
		  secret:
		    secretName: app-secret
		    
$ ls /opt/app-secret-volumes
# Output: DB_Host  DB_Password  DB_User

$ cat /opt/app-secret-volumes/DB_Password
# Output: paswrd
```