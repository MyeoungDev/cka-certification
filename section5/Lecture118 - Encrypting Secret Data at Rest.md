# Lecture 118 - Demo: Encrypting Secret Data at Rest

## Secret 생성 다양한 방법

- Secret 는 파일, literal, env file 등을 사용해서 손쉽게 생성 가능하다.

```bash
# Create a secret from all files within a directory:
$ kubectl create secret generic my-secret --from-file=path/to/bar

# Create a secret using specified keys from files:
$ kubectl create secret generic my-secret --from-file=ssh-privatekey=path/to/id_rsa --from-file=ssh-publickey=path/to/id_rsa.pub

# Create a secret from literal key-value pairs:
$ kubectl create secret generic my-secret --from-literal=key1=supersecret --from-literal=key2=topsecret

# Create a secret combining a file and a literal:
$ kubectl create secret generic my-secret --from-file=ssh-privatekey=path/to/id_rsa --from-literal=passphrase=topsecret

# Create a secret from environment files:
$ kubectl create secret generic my-secret --from-env-file=path/to/foo.env --from-env-file=path/to/bar.env

$ kubectl create secret generic my-secret-1 --from-literal=key1=su
persecret --dry-run=client -o yaml

apiVersion: v1
data:
  key1: c3VwZXJzZWNyZXQ=
kind: Secret
metadata:
  creationTimestamp: null
  name: my-secret-1
  
```

## 인코딩된 Secret 확인하기

```bash
$ kubectl describe secret my-secret

Name:         my-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
key1:  11 bytes

$ kubectl get secret my-secret -o yaml

apiVersion: v1
data:
  key1: c3VwZXJzWmNyZVQ=
kind: Secret
metadata:
  creationTimestamp: "2022-10-24T05:34:13Z"
  name: my-secret
  namespace: default
  resourceVersion: "2111"
  uid: dfe97c62-5aa1-46a8-b71c-ffa0cd4c08ec
type: Opaque
```

## ETCD 에서 Secret 데이터 검사

- etcd는 쿠버네티스가 클러스터 데이터를 저장하는 키-값 저장소
- 암호화가 해제된 상태에서는 Secret 값이 base64로만 인코딩되어 etcd에 액세스할 수 있는 모든 사용자가 디코딩할 수 있다.

```bash
ETCDCTL_API=3 etcdctl \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/my-secret | hexdump -C
```

## ETCD 데이터 암호화

- ETCD 에서는 저장된 Secret 데이터를 보호하기 위해 Encrypt Provider 를 구성하여 디스크 저장 데이터 암호화(Encryption at Rest) 메커니즘을 제공한다.

### 암호화 구성 파일 생성

- `resources` 어떤 리소스를 암호화할지 지정
- `providers` 암호화 방식을 지정
    - `aescbc` 실제 암호화를 수행하는 제공자
    - `identity` 암호화 없이 일반 텍스트로 저장하는 제공자.

```bash

$ head -c 32 /dev/urandom | base64

EfGSCSt8R+4Cc3WqoyiL462f6hmtt9ajygFhNt88Z5E=

$ vi enc.yaml

apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources:
  - secrets
    providers:
  - aescbc:
    keys:
    - name: key1
    secret: EfGSCSt8R+4Cc3WqoyiL462f6hmtt9ajygFhNt88Z5E=
  - identity: {}

```

### Kube API 서버에 구성 파일 적용

- 생성한 암호화 파일을 `kube-apiserver` 가 읽을 수 있도록 설정 필요.

```bash
$ mkdir -p /etc/kubernetes/enc
$ mv enc.yaml /etc/kubernetes/enc/

```

- `kube-apiserver.yaml` 파일 수정을 통해 암호화 구성 파일을 읽을 수 있도록 수정.

```bash
spec:
  containers:
  - command:
    - kube-apiserver
    # ... 기존 플래그 ...
    - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml
  volumeMounts:
  # ... 기존 마운트 ...
  - name: enc
    mountPath: /etc/kubernetes/enc
    readOnly: true
  volumes:
  # ... 기존 볼륨 ...
  - name: enc
    hostPath:
      path: /etc/kubernetes/enc
      type: DirectoryOrCreate

```

## 암호화 확인

- `Encryption at Rest` 과정을 거친 후 새로 생성되는 `Secrets` 는 암호화 된다.

```bash
$ kubectl create secret generic my-secret-2 --from-literal=key2=topsecret

ETCDCTL_API=3 etcdctl \\
--cacert=/etc/kubernetes/pki/etcd/ca.crt \\
--cert=/etc/kubernetes/pki/etcd/server.crt \\
--key=/etc/kubernetes/pki/etcd/server.key \\
get /registry/secrets/default/my-secret-2 | hexdump -C

```

### 기존 Secret 암호화

- `Encryption at Rest` 이전에 생성된 `Secrets` 를 암호화 하기 위해서는 아래 명령어를 사용해야 한다.
- 모든 Secrets를 조회한 후, 별다른 변경 없이 다시 적용(replace)
- 이 과정에서 kube-apiserver는 각 Secret를 읽고 자동으로 암호화하여 etcd에 다시 저장

```bash
$ kubectl get secret --all-namespaces -o json | kubectl replace -f -

```