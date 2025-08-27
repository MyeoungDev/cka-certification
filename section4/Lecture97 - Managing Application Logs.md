# Lecture 97 - Managing Application Logs

## 쿠버네티스 파드 로그 확인하기

- Docker 의 컨테이너 로그 확인하는것과 유사. (`docker logs -f <container_id>`)

```bash

$ vi event-simulator.yaml

apiVersion: v1
kind: Pod
metadata:
  name: event-simulator-pod
spec:
  containers:
    - name: event-simulator
      image: kodekloud/event-simulator

$ kubectl create -f event-simulator.yaml

$ kubectl logs -f event-simulator-pod

2018-10-06 15:57:15,937 - root - INFO - USER1 logged in
2018-10-06 15:57:16,943 - root - INFO - USER2 logged out
2018-10-06 15:57:17,944 - root - INFO - USER2 is viewing page2
2018-10-06 15:57:18,951 - root - INFO - USER3 is viewing page3
2018-10-06 15:57:20,095 - root - INFO - USER4 is viewing page1
2018-10-06 15:57:21,956 - root - INFO - USER2 logged out
2018-10-06 15:57:21,956 - root - INFO - USER1 logged in
2018-10-06 15:57:23,093 - root - INFO - USER3 is viewing page2
2018-10-06 15:57:24,959 - root - INFO - USER1 logged out
2018-10-06 15:57:25,961 - root - INFO - USER2 is viewing page2

```

- 만약, Pod 에 여러 컨테이너가 실행되는 상황의 경우에는 명확한 컨테이너 지정 필요.

```bash
$ vi event-simulator.yaml

apiVersion: v1
kind: Pod
metadata:
  name: event-simulator-pod
spec:
  containers:
    - name: event-simulator
      image: kodekloud/event-simulator
    - name: image-processor
      image: some-image-processor

$ kubectl logs -f event-simulator-pod event-simulator

```