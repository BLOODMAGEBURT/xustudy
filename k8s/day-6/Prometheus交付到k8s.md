### Prometheus交付到`k8s`

#### 1、交付4个 `exporter`

##### 1.1、`kube-state-metric`

下载镜像，制作本地镜像

```shell
docker pull quay.io/coreos/kube-state-metrics:v1.5.0

docker tag 91599517197a harbor.od.com/public/kube-state-metrics:v1.5.0

docker push harbor.od.com/public/kube-state-metrics:v1.5.0
```

准备资源配置清单

```shell
mkdir /data/k8s-yaml/kube-state-metrics
```

`rbac.yaml`

```
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: kube-state-metrics
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: kube-state-metrics
rules:
- apiGroups:
  - "" 
  resources:
  - configmaps
  - secrets
  - nodes
  - pods
  - services
  - resourcequotas
  - replicationcontrollers
  - limitranges
  - persistentvolumeclaims
  - persistentvolumes
  - namespaces
  - endpoints
  verbs:
  - list
  - watch
- apiGroups:
  - extensions
  resources:
  - daemonsets
  - deployments
  - replicasets
  - ingresses
  verbs:
  - list
  - watch
- apiGroups:
  - apps
  resources:
  - statefulsets
  - daemonsets
  - deployments
  - replicasets
  verbs:
  - list
  - watch
- apiGroups:
  - batch
  resources:
  - cronjobs
  - jobs
  verbs:
  - list
  - watch
- apiGroups:
  - autoscaling
  resources:
  - horizontalpodautoscalers
  verbs:
  - list
  - watch
- apiGroups:
  - policy
  resources:
  - poddisruptionbudgets
  verbs:
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: kube-state-metrics
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kube-state-metrics
subjects:
- kind: ServiceAccount
  name: kube-state-metrics
  namespace: kube-system
```

`dp.yaml`

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "2"
  labels:
    grafanak8sapp: "true"
    app: kube-state-metrics
  name: kube-state-metrics
  namespace: kube-system
spec:
  selector:
    matchLabels:
      grafanak8sapp: "true"
      app: kube-state-metrics
  template:
    metadata:
      labels:
        grafanak8sapp: "true"
        app: kube-state-metrics
    spec:
      containers:
      - image: harbor.od.com/public/kube-state-metrics:v1.5.0
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 5
          timeoutSeconds: 5
        name: kube-state-metrics
        ports:
        - containerPort: 8080
          name: http-metrics
      serviceAccountName: kube-state-metrics
```

##### 1.2、node-exporter

下载镜像，制作本地镜像

```shell
docker pull prom/node-exporter:v0.15.0
docker tag 12d51ffa2b22 harbor.od.com/public/node-exporter:v0.15.0
docker push harbor.od.com/public/node-exporter:v0.15.0
```

准备资源配置文件

```shell
mkdir /data/k8s-yaml/node-exporter
```

ds.yaml

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: kube-system
  labels:
    daemon: "node-exporter"
    grafanak8sapp: "true"
spec:
  selector:
    matchLabels:
      daemon: "node-exporter"
      grafanak8sapp: "true"
  template:
    metadata:
      labels:
        daemon: "node-exporter"
        grafanak8sapp: "true"
      name: node-exporter
    spec:
      volumes:
      - name: proc
        hostPath:
          path: /proc
          type: ""
      - name: sys
        hostPath:
          path: /sys
          type: ""
      containers:
      - image: harbor.od.com/public/node-exporter:v0.15.0
        name: node-exporter
        args:
        - --path.procfs=/host_proc
        - --path.sysfs=/host_sys
        ports:
        - containerPort: 9100
          hostPort: 9100
          name: scrape
        volumeMounts:
        - name: sys
          readOnly: true
          mountPath: /host_sys
        - name: proc
          readOnly: true
          mountPath: /host_proc
      hostNetwork: true
```

##### 1.3、cadvisor

下载镜像，制作本地镜像

```shell
docker pull google/cadvisor:v0.28.3
docker tag 75f88e3ec333 harbor.od.com/public/cadvisor:v0.28.3
docker push harbor.od.com/public/cadvisor:v0.28.3
```

修改所有运算节点软连接

```
mount -o remount,rw /sys/fs/cgroup/
ln -s /sys/fs/cgroup/cpu,cpuacct /sys/fs/cgroup/cpuacct,cpu
```

准备资源配置文件

```shell
mkdir /data/k8s-yaml/cadvisor
```

ds.yaml

```yaml
apiVersion: apps/v1 # for Kubernetes versions before 1.9.0 use apps/v1beta2
kind: DaemonSet
metadata:
  name: cadvisor
  namespace: kube-system
  labels:
      app: cadvisor
spec:
  selector:
    matchLabels:
      name: cadvisor
  template:
    metadata:
      labels:
        name: cadvisor
    spec:
      hostNetwork: true
      tolerations: 
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: cadvisor
        image: harbor.od.com/public/cadvisor:v0.28.3
        resources:
          requests:
            memory: 200Mi
            cpu: 150m
          limits:
            cpu: 300m
        volumeMounts:
        - name: rootfs
          mountPath: /rootfs
          readOnly: true
        - name: var-run
          mountPath: /var/run
          readOnly: true
        - name: sys
          mountPath: /sys
          readOnly: true
        - name: docker
          mountPath: /var/lib/docker
          readOnly: true
        ports:
          - name: http
            containerPort: 4194
            protocol: TCP
        readinessProbe:
          tcpSocket:
            port: 4194
          initialDelaySeconds: 5
          periodSeconds: 10
        args:
          - --housekeeping_interval=10s
          - --port=4194
      terminationGracePeriodSeconds: 30
      volumes:
      - name: rootfs
        hostPath:
          path: /
      - name: var-run
        hostPath:
          path: /var/run
      - name: sys
        hostPath:
          path: /sys
      - name: docker
        hostPath:
          path: /var/lib/docker
```

