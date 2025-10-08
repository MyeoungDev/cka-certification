# Lecture 180 - Service Accounts

- 쿠버네티스에는 사용자 계정, 서비스 계정 두 가지 계정 유형이 존재
- 사용자 계정
    - 관리자나 개발자와 같은 인간 사용자를 위해 설계
- 서비스 계정
    - 머신 간 상호작용 또는 애플리케이션 작업에 사용.

## 예시: Kubernetes 대시보드 애플리케이션

- Kubernetes 클러스터에서 Pod 목록 제공 애플리케이션

### ServiceAccount 생성

- Account 생성 시 자동으로 Token 을 생성하여 Secrets 으로 저장하고 계정에 연결한다.

```bash
$ kubectl create serviceaccount dashboard-sa

$ kubectl get serviceaccount

NAME           SECRETS   AGE
default        1         218d
dashboard-sa   1         4d

$ kubectl describe serviceaccount dashboard-sa

Name:                dashboard-sa
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   dashboard-sa-token-kbbdm
Tokens:              dashboard-sa-token-kbbdm
Events:              <none>

$ kubectl describe secret dashboard-sa-token-kbbdm

Name:                dashboard-sa-token-kbbdm
Namespace:           default
Labels:              <none>
Type:                kubernetes.io/service-account-token
Data
  token: eyJhbGciOiJSUzI1NiIsImtpZCI6Ij...  (truncated for privacy)
```

## 서비스 계정 토큰 자동 마운트

- 쿠버네티스트 클러스터에 타사 애플리케이션(`Prometheus`) 을 사용할 경우 서비스가 계정 토큰을 Pod에 볼륨으로 자동 마운트하도록 설정할 수 있다.
- 일반적으로 `/var/run/secrets/kubernetes.io/serviceaccount` 해당 경로를 사용한다.
- 모든 네임스페이스에는 Pod 에 자동으로 삽입되는 기본 서비스 계정이 포함되어 있다.
    - `default-token-xxxx` 이름으로 표현되어 마운트된 볼륨을 확인 가능하다.

```bash
$ kubectl describe pod my-kubernetes-dashboard

Name:           my-kubernetes-dashboard
Namespace:      default
Status:         Running
IP:             10.244.0.15
Containers:
  nginx:
    Image:        my-kubernetes-dashboard
    Mounts:       /var/run/secrets/kubernetes.io/serviceaccount from default-token-j4hkv (ro)
Volumes:
  default-token-j4hkv:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-j4hkv
    Optional:    false
```

## 다른 서비스 계정 할당

- 기본 `default` 계정을 사용하지 않고 다른 서비스 계정을 할당할 수 있다.
- `serviceAccountName` 옵션을 사용하면 된다.
- 기존 파드의 서비스 계정은 수정할 수 없다.
- 새 서비스 계정을 사용하려면, 파드를 삭제 후 재생성 해야 한다.
- Pod 템플릿이 변경되면 배포 시 새 Pod가 자동으로 롤아웃된다.
- 서비스 계정 토큰의 자동 마운트를 비활성화 하려면 `automountServiceAccountToken : false` 옵션을 설정하면 된다.

```bash
apiVersion: v1
kind: Pod
metadata:
	name: my-kubernetes-dashboard
spec:
	serviceAccountName: dashboard-sa
	containers:
		- name: my-kubernetes-dashboard
			image: my-kubernetes-dashboard
```

## Kubernetes 버전 1.22 및 1.24의 변경 사항

- 쿠버네티스 v1.22 이전에는 서비스 계정 토큰이 만료일 없이 Secrets에서 자동으로 마운트되었습니다
- v1.22부터 `TokenRequest API(KEP-1205)`가 도입되어 대상, 시간 및 객체에 따라 달라지는 토큰을 생성하여 보안을 크게 강화되었다.
- Kubernetes v1.24부터 Kubernetes는 더 이상 만료되지 않는 서비스 계정 토큰을 자동으로 생성하여 비밀로 저장하지 않는다.
- 대신, 새 서비스 계정을 생성한 후 다음을 사용하여 토큰을 명시적으로 생성해야 한다.
- API 로 생성된 토큰은 만료, 대상 제한, 향상된 관리 용이성 과 같은 추가적인 보안 기능을 제공하므로, `ToeknRequest API` 를 사용하는 것이 권장된다.
- `Secret` 을 수동으로 생성하여 만료되지 않는 토큰을 만들 수는 있지만, 권장되지 않는다.

```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: default
spec:
  containers:
    - image: nginx
      name: nginx
      volumeMounts:
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
          name: kube-api-access-6mtg8
          readOnly: true
  volumes:
    - name: kube-api-access-6mtg8
      projected:
        defaultMode: 420
        sources:
          - serviceAccountToken:
              expirationSeconds: 3607
            path: token
          - configMap:
              name: kube-root-ca.crt
              items:
                - key: ca.crt
                  path: ca.crt
          - downwardAPI:
              items:
                - fieldRef:
                    apiVersion: v1
                    fieldPath: metadata.namespace
```

```bash
$ kubectl create token dashboard-sa
```