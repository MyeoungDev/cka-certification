# Lecture 110 - Configure Environment Variables in Applications

## Docker를 사용한 환경 변수 설정

- Docker 컨테이너를 실행할 때 `-e` 플래그를 사용해서 환경 변수를 설정할 수 있다.

```bash
docker run -e APP_COLOR=pink simple-webapp-color
```

## Kubernetes Pod 환경 변수 구성

- 쿠버네티스는 파드 정의 내에서 환경 변수를 정의할 수 있다.
- Pod 정의 내부에 `env` 배열 속성을 사용한다.

```bash
apiVersion: v1
kind: Pod
metdata:
	name: simple-webapp-color
sepc:
	containers:
	- name: simple-webapp-color
		image: simple-webapp-color
		ports:
			- containerPort: 8080
		env:
			- name: APP_COLOR
				value: pink
```

## 환경 변수에 대한 ConfigMaps 및 Secrets 활용

- 파드 내에 환경변수를 하드코딩 하는 대신 `ConfigMap` , `Secrets` 를 활용하여 외부 구성 소스를 참조하여 유연성과 보안을 강화할 수 있다.
- 민감 정보를 다룰 때는 항상 `Secrets` 를 사용해야 한다

```bash
env:
	- name: APP_COLOR
		value: pink

---

# ConfigMap

env:
	- name: APP_COLOR
		valueFrom:
			configMapKeyRef:
				name: app-config
				key: color
				
---

# Secrets

env:
	- name: APP_COLOR
		valueFrom:
			secretKeyRef:
				name: app-secrets
				key: color
```