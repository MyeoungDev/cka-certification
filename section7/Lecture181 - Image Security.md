# Lecture 181 - Image Security

## 컨테이너 이미지 명명 이해

```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx

```

- 이미지 이름이 `nginx` 로 지정되어 있다. 이는 Docker의 이미지 명명 규칙을 따른다.
- 기본적으로 Docker는 다른 레지스트리가 지정되지 않으면 Docker Hub(DNS 이름: [docker.io](http://docker.io/))에서 이미지를 가져온다.
- 사용자나 계정 없이 저장소 이름을 지정하면 Docker 는 기본적으로 `library` 계정을 사용한다.
    - 즉, `nginx` -> `library/nginx` 으로 해석되며, 이는 Docker 공식 이미지를 나타낸다.
- 만역, 특정 레지스트리를 이용할 경우 그에 맞게 명시해야 한다. (ex: `private-registry/nginx`)

## Private Registry 사용

- 보안을 강화하기 위해 Private Registry 를 사용할 수 있다.
    - 인기 클라우드 서비스는 플랫폼에 내장된 Private Registry 를 제공하기도 하며, Google Container Registry([gcr.io](http://gcr.io/)) 는 쿠버네티스 관련 이미지 및 테스트 목적으로 자주 사용된다.
- Private Registry 이미지를 사용할 경우 전체 이미지 경로를 지정해야 한다. (ex: `docker.io/library/nginx`)

### Private Registry Authenticate

- Private Registry 에 접근하기 위해서는 인증이 필요하다.

```bash
$ docker login private-registry.io

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to <https://hub.docker.com> to create one.
Username: registry-user
Password:
WARNING! Your password will be stored unencrypted in /home/vagrant/.docker/config.json.
Login Succeeded

```

## Configuring Kubernetes Pods for Private Registries

- 쿠버네티스 Worker Node 는 이미지 검색을 위해 Docker Runtime 을 사용하므로 인증 정보를 제공해둬야 한다.
- Kubernetes Secrets 을 생성하여 레지시스트리에 대한 인증 정보를 등록 사용 가능하다.
- 파드가 생성되면 Worker Node 의 `kubelet` 은 `Secrets` 에 저장된 자격 증명을 사용하여 인증을 수행하고 Private Registry 에서 이미지를 가져온다.

```bash
kubectl create secret docker-registry regcred \\
--docker-server=private-registry.io \\
--docker-username=registry-user \\
--docker-password=registry-password \\
--docker-email=registry-user@org.com

```

```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: private-registry.io/apps/internal-app
  imagePullSecrets:
    - name: regcred

```