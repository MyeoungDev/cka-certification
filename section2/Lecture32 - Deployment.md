# Lecture32 - Deployments

## Deployment

- Deployment 는 Pod 및 ReplicaSet 과 직접 상호작용하며, 고급기능들을 제공한다.
- 여러 개의 애플리케이션 인스턴스를 배포하여 서비스의 안정성을 보장하고 부하를 분산한다.
- 새로운 인스턴스를 점진적으로 추가하고 기존 인스턴스를 제거하는 롤링 업데이트(Rolling Update) 방식을 사용하여, 업데이트 중 서비스 다운타밈을 최소화 한다.
- 업데이트 실패 시, 이전 버전으로 신속히 Rollback 한다.
- 배포를 일시 정지하여 스케일링, 버전 등의 변경 사항을 조정하고 다시 재개할 수 있다.

## Deployment 생성

```bash
$ vi deployment-definition.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      type: front-end
  template:
    metadata:
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx

$ kubectl create -f deployment-definition.yml

$ kubectl get deployments

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
myapp-deployment   3/3     3            3           8s

$ kubectl get replicasets

NAME                          DESIRED   CURRENT   READY   AGE
myapp-deployment-5dc6d7c855   3         3         3       51s

$ kubectl get pods

myapp-deployment-5dc6d7c855-768vt   1/1     Running   0          17s
myapp-deployment-5dc6d7c855-p2tgr   1/1     Running   0          20s
myapp-deployment-5dc6d7c855-rm4sp   1/1     Running   0          23s

```

## Deployment 동작 방식

- Deployment 를 생성하면 쿠버네티스가 자동으로 연결된 ReplicaSet 을 생성한다.
- 해당 ReplicaSet 은 Deployment 에서 정의한대로 Pod 의 생성 및 관리를 감독한다.

```bash

$ kubectl get all

NAME                                    READY   STATUS    RESTARTS   AGE
pod/myapp-deployment-5dc6d7c855-768vt   1/1     Running   0          56m
pod/myapp-deployment-5dc6d7c855-p2tgr   1/1     Running   0          56m
pod/myapp-deployment-5dc6d7c855-rm4sp   1/1     Running   0          57m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   150d

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/myapp-deployment   3/3     3            3           57m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-deployment-5dc6d7c855   3         3         3       57m

```