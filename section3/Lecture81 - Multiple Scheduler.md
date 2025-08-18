# Lecture 81 - Multiple Schedulers

## Multiple Scheduler

- 쿠버네티스의 기본 스케줄러(default-scheduler)는 Taints, Toleration, Node Affinity 같은 요소를 고려하여 여러 노드에 파드를 균등하게 분배한다.
- 하지만, 특정 사용 사례에는 맞춤형 스케줄링 알고리즘이 필요할 수 있다. (ex: 파드 배치 전 노드의 추가 검증 작업)
- 자체 스케줄러를 작성하고 패키징하여 기본 스케줄러와 함께 배포하면 특정 요구에 맞게 파드 배치를 조정할 수 있다.

## Scheduler 구성

```bash
# my-scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
  profiles:
  - schedulerName: my-scheduler

# scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
  profiles:
  - schedulerName: default-scheduler

```

- 파드에서 특정 스케줄러를 지정할 경우 `schedulerName` 속성을 사용하면 된다.

```bash
$ vi custom-scheduler-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
  schedulerName: my-custom-scheduler

$ kubectl create -f custom-scheduler-pod.yaml

$ kubectl get events -o wide

LAST SEEN   COUNT   NAME        KIND   TYPE    REASON      SOURCE                  MESSAGE
9s          1       nginx.15    Pod    Normal  Scheduled   my-custom-scheduler     Successfully assigned default/nginx to node01
8s          1       nginx.15    Pod    Normal  Pulling     kubelet, node01         pulling image "nginx"
2s          1       nginx.15    Pod    Normal  Pulled      kubelet, node01         Successfully pulled image "nginx"
2s          1       nginx.15    Pod    Normal  Created     kubelet, node01         Created container
2s          1       nginx.15    Pod    Normal  Started     kubelet, node01         Started container

```

## 스케줄러 배포

- 기존 `kube-scheduler` 바이너리를 사용하여 추가 스케줄러를 배포하고 특정 서비스 파일을 통해 설정을 할 수 있다.

```bash
$ wget <https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-scheduler>

$ vi my-scheduler.service

ExecStart=/usr/local/bin/kube-scheduler --config=/etc/kubernetes/config/my-scheduler-2-config.yaml

$ vi my-scheduler-config.yaml

apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler

```

## 사용자 정의 스케줄러 Pod 배포

- 스케줄러를 서비스로 실행하는 것 외에도 파드 형태로 배포할 수 있다.

```bash

$ vi my-scheduler-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
    - name: kube-scheduler
      image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
      command:
        - kube-scheduler
        - --address=127.0.0.1
        - --kubeconfig=/etc/kubernetes/scheduler.conf
        - --config=/etc/kubernetes/my-scheduler-config.yaml

apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler

```

## 사용자 정의 스케줄러 Deployment 배포

```bash

$ vi Dockerfile

FROM busybox
ADD ./.output/local/bin/linux/amd64/kube-scheduler /usr/local/bin/kube-scheduler

$ docker build -t gcr.io/my-gcp-project/my-kube-scheduler:1.0 .
$ gcloud docker -- push gcr.io/my-gcp-project/my-kube-scheduler:1.0

$ vi rbac.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-scheduler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: my-scheduler-as-kube-scheduler
subjects:
  - kind: ServiceAccount
    name: my-scheduler
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: system:kube-scheduler
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: my-scheduler-as-volume-scheduler
subjects:
  - kind: ServiceAccount
    name: my-scheduler
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: system:volume-scheduler
  apiGroup: rbac.authorization.k8s.io

$ configmap.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: my-scheduler-config
  namespace: kube-system
data:
  my-scheduler-config.yaml: |
    apiVersion: kubescheduler.config.k8s.io/v1beta2
    kind: KubeSchedulerConfiguration
    profiles:
      - schedulerName: my-scheduler
        leaderElection:
          leaderElect: false

$ deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-scheduler
  namespace: kube-system
  labels:
    component: scheduler
    tier: control-plane
spec:
  replicas: 1
  selector:
    matchLabels:
      component: scheduler
      tier: control-plane
  template:
    metadata:
      labels:
        component: scheduler
        tier: control-plane
        version: second
    spec:
      serviceAccountName: my-scheduler
      containers:
        - name: kube-second-scheduler
          image: gcr.io/my-gcp-project/my-kube-scheduler:1.0
          command:
            - /usr/local/bin/kube-scheduler
            - --config=/etc/kubernetes/my-scheduler/my-scheduler-config.yaml
          livenessProbe:
            httpGet:
              path: /healthz
              port: 10259
              scheme: HTTPS
            initialDelaySeconds: 15
          readinessProbe:
            httpGet:
              path: /healthz
              port: 10259
              scheme: HTTPS
          volumeMounts:
            - name: config-volume
              mountPath: /etc/kubernetes/my-scheduler
      volumes:
        - name: config-volume
          configMap:
            name: my-scheduler-config

```