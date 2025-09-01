# Lecture 111 - Configuring ConfigMaps in Applications

## ConfigMap 을 사용한 구성 중앙화

- 환경 구성 관리를 간소화하기 위해 `ConfigMap` 을 사용해서 데이터를 외부화 할 수 있다.
- 이 접근 방식을 통해 쿠버네티스는 파드 생성 시 중앙에 저장된 `Key-Value` 쌍을 파드에 주입한다.

```bash
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp-color
spec:
	containers:
	- name: simple-webapp-color
		image: simple-webapp-color
		ports:
			- containerPort: 80
	envFrom:
		- configMapRef:
				name: app-color
```

## ConfigMap 생성

### **Imperative Approach (**명령형 접근 방식 )

- `ConfigMap` 정의 파일 없이 명령줄에 바로 사용할 수 있다.

```bash
kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MOD=prod
```

- 구성 파일을 참조하게 설정할 수도 있다.

```bash
kubectl create configmap app-config --from-file=app_config.properties
```

### **Declarative Approach (선언형 접근 방식)**

- YAML 파일에 `ConfigMap` 을 정의하고 적용할 수 있다.

```bash
$ vi config-map.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod

$ kubectl create -f config-map.yaml

$ kubectl get configmaps

$ kubectl describe configmaps

Name:           app-config
Namespace:      default
Labels:         <none>
Annotations:    <none>
Data
====
APP_COLOR:
----
blue
APP_MODE:
----
prod
Events:        <none>
```

## Pod 에 ConfigMap 주입

- Pod 정의 내에 `envFrom` 옵션을 통해 `ConfigMap` 을 주입하여 사용할 수 있다.

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
    - configMapRef:
        name: app-config

```

- `valueFrom` 으로 단일 환경 변수를 주입하여 사용하거나, `ConfigMap` 전체 볼륨을 마운트 해서 사용할 수 있다.

```bash
envFrom:
  - configMapRef:
      name: app-config

env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR

volumes:
  - name: app-config-volume
    configMap:
      name: app-config
```

- 각 메서드는 애플리케이션의 유연한 디자인 설계를 제공한다.