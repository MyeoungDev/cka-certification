# Lecture 89 - Validating and Mutating Admission Controller

## Admission Controller 개요

- Admission Controller 는 API 요청 라이프사이클 동안 중요한 작업을 수행한다.
- 일반적으로 Mutating Controller 는 Validating Controller 보다 먼저 호출된다.
- 이 순서는 변경 중에 발생한 모든 변경 사항이 유효성 검사에 반영되도록 보장한다.
    - 예를 들어, NamespaceAutoProvisioning Controller 보다 NamespaceExists Controller 가 먼저 실행되어, 누락된 네임스페이스 검사거 먼저 진행되면 서 요청이 실패하게 되는 경우를 방지하는 것이다.
- Admission Controller 가에서 Reject 거절이 일어날 경우 오류 메시지가 반환되고 요청은 더 이상 처리되지 않는다.
- Mutating Admission Controller
    - 객체가 생성되기 전에 객체를 수정한다.
- Validating Admission Controller
    - 요청을 검사하여 허용할지 거부할지 결정한다.
- Dual-Purpose Controller
    - 몇개의 컨트롤러는 Mutating 과 Validating 모두를 지원한다.

## Namespace and Storage Class Admission Controllers

### Namespace Existence

- 네임스페이스 존재 여부(또는 네임스페이스 수명 주기) 승인 컨트롤러는 네임스페이스가 이미 존재하는지 확인
- 네임스페이스가 존재하지 않으면 요청이 거부된다. 

### Default Storage Class Plugin

- Default Storage Class Admission Controller 는 기본적으로 활성화 되어 있다.
- 이 컨트롤러는 기본 스트로지 클래스가 제공되지 않으면 해당 클래스를 추가하여 요청을 변경한다.
- 스토리지 클래스를 지정하지 않고, PersistentVolumeClaim(PVC)을 생성하면 요청이 인증 및 권한 부여 과정을 거친 후 Admission Controller 를 통해 전달 된다.

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  storageClassName: default

$ vi kubetl describe pvc myclaim

Name:           myclaim
Namespace:      default
StorageClass:   default
Status:         Pending
Volume:         <none>
Labels:         <none>
Annotations:    <none>

```

## External Admission Webhooks

- 클러스터 외부의 HTTP 서버(웹훅 서비스)를 호출하여 어드미션 로직을 처리하는 방식
- External Admission Webhook 를 사용하여 사용자 커스텀 로직으로 Admission Controller 기능을 확장할 수 있다.
- 웹훅을 사용하려면 `MutatingWebhookConfiguration` 또는 `ValidatingWebhookConfiguration` 을 생성하여 API 서버가 어떤 요청을 어떤 웹훅 서비스로 보낼지 지정해야 한다.

## External Admission Webhook 동작 방식

- kube-apiserver 서버 객체 생성 요청 받음.
- MutatingWebhookConfiguration
    - 내장된 Mutation Admission Controller 가 요청을 수정.
    - 등록된 `MutatingWebhookConfiguration` External Webhook 으로 `AdmissionReview` 객체 요청.
    - External Webhook Service 에서 `AdmissionReview` 객체 검토 및 수정 응답.
- ValidatingWebhookConfiguration
    - 내장된 Validating Admission Controller 가 유효섬 검증.
    - 등록된 `ValidatingWebhookConfiguration` External Webhook 으로 `AdmissionReview` 객체 요청.
    - External Webhook Service 에서 `AdmissionReview` 객체 검증 작업 진행.