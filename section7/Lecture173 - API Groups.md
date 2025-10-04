# Lecture 173 - API-Groups

## Kubernetes API 이해하기

- 쿠버네티스 API는 클러스터와 상호 작용하는 기본 인터페이스이다.
- 명령줄 도구를 사용하든 `kubectl` REST를 통해 HTTP 요청을 직접 전송하든 모든 상호 작용은 API 서버와 통신한다.

## API 그룹과 그 목적

- 쿠버네티스는 특정 기능에 따라 API를 여러 그룹으로 구성한다.
- 이러한 그룹은 버전 관리, 상태 지표, 로깅 등을 관리하는 데 도움이 된다.
    - ex: `/version` - 클러스터 버전 정보, `/metrics` - 클러스터 성능 및 상태 정보

### Core API Group

- Namespace, Pod, Replica Controller, Service, Node, Persistent Volume 등과 같은 쿠버네티스의 필수 기능을 포함.

### Named API Groups

- 최신기능에 대한 체계적인 구조를 제공.
- 확장 프로그램, 네트워크 , 인증 및 권한 이 포함된다.

### API 서버 쿼리

- 사용 가능한 API 그룹 목록을 검색하려면 API 서버의 루트 엔드포인트에 액세스 하면된다.

```bash
$ curl <http://localhost:6443> -k

{
  "paths": [
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    "/healthz",
    "/logs",
    "/metrics",
    "/openapi/v2",
    "/swagger-2.0.0.json"
  ]
}

```

- 인증이 없이는 `/version` 엔드포인트만 접근 가능하다.
- 나머지 엔드포인트의 겨우 403 Forbidden 오류가 발생한다.

```bash
$ curl <http://localhost:6443> -k

{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "forbidden: User \\"system:anonymous\\" cannot get path \\"/\\"",
  "reason": "Forbidden",
  "details": {},
  "code": 403
}

```

- 따라서, 아래와 같이 인증을 추가해서 요청을 해야 한다.
- 혹은 `kubectl proxy` 를 이용하여 HTTP 요청에 대해서 proxy 를 적용시켜 인증을 매번 수동으로 사용하지 않게 할 수 있디.

```bash
curl <http://localhost:6443> -k \\
--key admin.key \\
--cert admin.crt \\
--cacert <your-ca-cert-file>

```