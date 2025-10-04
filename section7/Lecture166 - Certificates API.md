# Lecture 166 - Certificates-API

- 일반적인 쿠버네티스 클러스터 설정에서 관리자는 먼저 인증 기관(CA) 서버를 구성하고 다양한 구성 요소에 대한 인증서를 생성한다.
- 이러한 인증서를 사용하여 서비스가 시작되면 클러스터가 동작하기 시작한다.
- 처음에는 한 명의 관리자만 개인 관리자 인증서와 키를 통해 엑세스할 수 있다.
- 그러나, 새로운 팀원이 합류하면 각 구성원은 클러스터에 액세스하기 위해 자신의 인증서와 키 쌍을 획득해야 한다.

## Certificate Life Cycle

- Kube API 서버는 클러스터에서 중추적인 역할을 하지만 인증 기관(CA)의 일부는 아니다.
- CA는 초기화 과정에서 생성되는 키와 인증서, 두 개의 파일로만 구성된다.
- 이 파일들은 인증서 서명 및 모든 권한을 가진 사용자 생성을 허용하므로, 일반적으로 전용 CA 서버에 안전하게 저장되어야 한다.
- 많은 구현에서 쿠버네티스 마스터 노드는 CA 서버 역할도 수행한다.
- 예를 들어, kubeadm 도구는 마스터 노드에 CA 파일을 생성하고 저장한다.

### 인증서 발급 프로세스

- 새 관리자가 가입하면 자신의 개인 키를 생성하고 인증서 서명 요청(CSR)을 생성.
- 이 CSR 은 기존 관리자에게 전송.
- 단독 관리자는 CSR을 검토하고 CA 서버의 개인 키와 루트 인증서를 사용하여 서명한 후 서명된 인증서를 반환.
- 새 관리자는 클러스터에 액세스할 수 있는 유효한 인증서와 키 쌍 획득.
- 인증서에는 만료 기간이 정의되어 있으므로 만료되면 이 프로세스가 반복.

## Certificate Rotation Automation

- 사용자 수가 증가함에 따라 인증서 요청에 수동으로 서명하는 것은 비현실적이 된다.
- 쿠버네티스는 CSR 관리 및 인증서 순환을 자동화하는 내장 인증서 API를 통해 이러한 문제를 해결한다.

## 인증서 서명 요청(CSR) 관리

- `Kubernetes Certificates API` 를 사용하면 사용자가 API 호출을 통해 CSR을 제출하고 `CertificateSigningRequest` 객체를 생성할 수 있다.
- 관리자는 `kubectl` 명령을 사용하여 이러한 요청을 검토하고 승인할 수 있다.
- 승인되면 쿠버네티스는 CA의 키 쌍을 사용하여 인증서에 서명한다.
- 서명된 인증서는 추출되어 요청 사용자에게 배포될 수 있다.

## User Generates Private Key and CSR

- 사용자는 다음 명령을 사용하여 개인 키를 생성하고 인증서 서명 요청을 생성한다.

```bash
$ openssl genrsa -out jane.key 2048

```

- 그 이후 사용자는 CSR을 관리자에게 보낸다.

### Administrator Creates a CSR Object

- 관리자는 매니페스트 파일을 사용하여 `CertificateSigningRequest` 객체를 생성.
- 매니페스트에서 는 `kindCertificateSigningRequest`로 설정되고, 해당 `spec`섹션에는 인코딩된 인증서 서명 요청(CSR은 base64로 인코딩되어야 함)이 포함된다.

```bash
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  expirationSeconds: 600 # seconds
  usages:
    - digital signature
    - key encipherment
    - server auth
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNUUw0tLS0KTUl1Q1dEQ0NBVUFDQVFBd0V6RVJHQTFVdU0R6VjRkNHTQ0RzU0aU1yY3I0d11qYXl0c1RUVFRlQiVtNS0tLS0tLkRvd25nUIDhUnQ0dXJ0YW50YmlsZWdslNQZHYR0W1nNHh1RVFLdLtJPG0tLkFUTUJQS0w0UlRqS1JlTVUyZUl3bTJaSE44TG5NQ2czTWc9PQ==

```

```bash
$ kubectl get csr

NAME      AGE   SIGNERNAME                                   REQUESTOR                  REQUESTEDDURATION   CONDITION
jane      10m   kubernetes.io/kube-apiserver-client         admin@example.com          10m                 Pending

```

### Approving the CSR

```bash
$ kubectl certificate approve jane

```

- 승인 후, 쿠버네티스는 CA 키 쌍으로 CSR에 서명하고, 인증서는 `CertificateSigningRequest` 객체의 YAML 출력에 base64로 인코딩된 문자열로 포함된다.
- base64 유틸리티를 사용하여 디코딩하면 일반 텍스트 인증서를 볼 수 있다.

```bash
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  creationTimestamp: 2019-02-13T16:36:43Z
  name: new-user
spec:
  groups:
    - system:masters
    - system:authenticated
  expirationSeconds: 600
  usages:
    - digital signature
    - key encipherment
    - server auth
  username: kubernetes-admin
status:
  certificate: L$0tL1CRUdJTiBDRVJUSUZJQ0FURS9tL0t1SURDakNDQWZLZ0F3SUJBZ0lVRmwyQ2wXYXoxalW5M3JNVisreFRYQYouW3dnd0RWpL1pJaHZjTkFRRUwkQkFBd0ZVUnRVMVhQTFRUVF4TUhM1ZpHkdVpMjxkF1RncweE9UQ1NVE14TmpNeU1EQmFgdGl0dY0ZFBl2ajNuSXY3eFd3I1Rm5u440c0t520vXukwTFM5V29ge1hHZdWCMlEZ2FOMVVMRFBXTVhjN09FVnVjSk1k4weRUVtR5tD11zWeHVjS1h6g1dV0pMediMUGbXYFKWVKWMVmBjRVRTY3dod2xiO1ND0kLS0tL1F0kQg0V5VElSGUNBVEUt
  conditions:
    - lastUpdateTime: 2019-02-13T16:37:21Z
      message: This CSR was approved by kubectl certificate approve.
      reason: KubectlApprove
      type: Approved

```

## The Role of the Controller Manager

- 쿠버네티스 controlplane 내에서는 API 서버, 스케줄러, 컨트롤러 관리자와 같은 구성 요소가 함께 작동.
- 그러나 CSR 승인 및 서명과 같은 모든 인증서 관련 작업은 컨트롤러 관리자가 관리한다.
- Controller Manager 에는 CSR 승인 및 CSR 서명 작업을 위한 전용 컨트롤러가 포함되어 있다.
- 인증서 서명에는 CA의 루트 인증서와 개인 키에 대한 접근 권한이 필요하므로, 컨트롤러 관리자의 구성은 이러한 자격 증명의 파일 경로를 지정한다.

```bash
$ cat /etc/kubernetes/manifests/kube-controller-manager.yaml

spec:
  containers:
  - command:
      - kube-controller-manager
      - --address=127.0.0.1
      - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
      - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
      - --controllers=*,bootstrapsigner,tokencleaner
      - --kubeconfig=/etc/kubernetes/controller-manager.conf
      - --leader-elect=true
      - --root-ca-file=/etc/kubernetes/pki/ca.crt
      - --service-account-private-key-file=/etc/kubernetes/pki/sa.key
      - --use-service-account-credentials=true

```