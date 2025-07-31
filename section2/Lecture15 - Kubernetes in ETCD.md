# Lecture15 - Kubernetes in ETCD

## Kubernetes 에서 ETCD 의 역할

- ETCD 는 kubernetes 크러스터의 모든 상태 정보를 저장하는 핵심 데이터 저장소
- 노드, 파드, 시크릿, 계정, 역할 등 클러스의 모든 중요한 정보가 ETCD에 저장된다.
- `kubectl` 명령어를 통해 조회되는 모든 정보는 ETCD 서버에서 가져오는 것이다.
- 클러스터에 변화가 생기면, 해당 변경 사항이 ETCD 서버에 업데이트될 때 완전히 완료된 것으로 간주.



## ETCD의 배포방식

쿠버네티스 클러스터를 설정하는 방법에 따라 ETCD 배포 방식이 달라질 수 있다.

- 스크래치 (Scratch) 배포
    - ETCD 바이너를 직접 다운로드하고, 마스터 노드에 수동으로 설치하여 서비스로 구성.
    - ETCD 서비스에 대한 TLS 인증서, 클러스터 구성, `kube-apiserver` 가 ETCD 에 접근하기위한 `advertiesed client URL` 등의 직접 설정이 필요.
    - 다중 마스터 노드 구성할 경우 ETCD 인스턴스가 서로 인식할 수 있게 `initial-cluster` 옵션 사용 필요.

```bash
$ wget -q --https-only \
"https://github.com/coreos/etcd/releases/download/v3.3.9/etcd-v3.3.9-linux-amd64.tar.gz"

# Example etcd service configuration
ExecStart=/usr/local/bin/etcd \
  --name ${ETCD_NAME} \
  --cert-file=/etc/etcd/kubernetes.pem \
  --key-file=/etc/etcd/kubernetes-key.pem \
  --peer-cert-file=/etc/etcd/kubernetes.pem \
  --peer-key-file=/etc/etcd/kubernetes-key.pem \
  --trusted-ca-file=/etc/etcd/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/ca.pem \
  --peer-client-cert-auth \
  --client-cert-auth \
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \
  --listen-peer-urls https://${INTERNAL_IP}:2380 \
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \
  --advertise-client-urls https://${INTERNAL_IP}:2379 \
  --initial-cluster-token etcd-cluster-0 \
  --initial-cluster controller-0=https://${CONTROLLER0_IP}:2380,controller-1=https://${CONTROLLER1_IP}:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
```

- `kubeadm` 도구를 이용한 배포
    - `kubeadm` 은 클러스터 설정을 자동화 해주는 도구.
    - `kubeadm` 을 사용하면 ETCD 는 마스터 노드 내에서 `kube-system` 네임스페이스에 있는 Pod 형태로 배포.
    - 해당 Pod  내의 `etcdctl` 을 이용해 ETCD 데이터 조회 가능.

```bash
$ kubectl get pods -n kube-system

NAMESPACE     NAME                                 READY   STATUS      RESTARTS   AGE
kube-system   coredns-78fcdf6894-prwl              1/1     Running     0          1h
kube-system   coredns-78fcdf6894-vqd9w             1/1     Running     0          1h
kube-system   etcd-master                          1/1     Running     0          1h
kube-system   kube-apiserver-master                1/1     Running     0          1h
kube-system   kube-controller-manager-master       1/1     Running     0          1h
kube-system   kube-proxy-f6k26                     1/1     Running     0          1h
kube-system   kube-proxy-hnzw                      1/1     Running     0          1h
kube-system   kube-scheduler-master                1/1     Running     0          1h
kube-system   weave-net-924k8                      2/2     Running     1          1h
kube-system   weave-net-hzfcz                      2/2     Running     1          1h

$ kubectl exec etcd-master -n kube-system -- etcdctl get / --prefix --keys-only

```

## Kubernetes 데이터 저장 구조

- 쿠버네티스는 ETCD 내부에 데이터를 트정 디렉토리 구조로 저장.
- `registry` 라는 루트 디렉토리 아래에 노드, 파트, replica set, deploy 등 다양한 쿠버네티스 객체들이 계층적으로 저장.

```bash
$ kubectl exec etcd-master -n kube-system -- etcdctl get / --prefix --keys-only

/registry/apiregistration.k8s.io/apiservices/v1
/registry/apiregistration.k8s.io/apiservices/v1.apps
/registry/apiregistration.k8s.io/apiservices/v1.authentication.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.authorization.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.autoscaling
/registry/apiregistration.k8s.io/apiservices/v1.batch
/registry/apiregistration.k8s.io/apiservices/v1.networking.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.rbac.authorization.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1beta1.admissionregistration.k8s.io

```