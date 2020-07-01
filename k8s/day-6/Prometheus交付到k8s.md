### Prometheus交付到`k8s`

[TOC]



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
          readOnly: false
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

##### 1.4、`blackbox-exporter`

准备镜像

```
docker pull prom/blackbox-exporter:v0.15.1

docker tag 81b70b6158be harbor.od.com/public/blackbox-exporter:v0.15.1

docker push harbor.od.com/public/blackbox-exporter:v0.15.1
```

准备资源配置文件

```shell
mkdir /data/k8s-yaml/blackbox-exporter
```

cm.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    app: blackbox-exporter
  name: blackbox-exporter
  namespace: kube-system
data:
  blackbox.yml: |-
    modules:
      http_2xx:
        prober: http
        timeout: 2s
        http:
          valid_http_versions: ["HTTP/1.1", "HTTP/2"]
          valid_status_codes: [200,301,302]
          method: GET
          preferred_ip_protocol: "ip4"
      tcp_connect: 
        prober: tcp
        timeout: 2s
```

dp.yaml

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: blackbox-exporter
  namespace: kube-system
  labels:
    app: blackbox-exporter
  annotations:
    deployment.kubernetes.io/revision: 1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: blackbox-exporter
  template:
    metadata:
      labels:
        app: blackbox-exporter
    spec:
      volumes:
      - name: config
        configMap:
          name: blackbox-exporter
          defaultMode: 420
      containers:
      - name: blackbox-exporter
        image: harbor.od.com/public/blackbox-exporter:v0.15.1
        imagePullPolicy: IfNotPresent
        args:
        - --config.file=/etc/blackbox_exporter/blackbox.yml
        - --log.level=info
        - --web.listen-address=:9115
        ports:
        - name: blackbox-port
          containerPort: 9115
          protocol: TCP
        resources:
          limits:
            cpu: 200m
            memory: 256Mi
          requests:
            cpu: 100m
            memory: 50Mi
        volumeMounts:
        - name: config
          mountPath: /etc/blackbox_exporter
        readinessProbe:
          tcpSocket:
            port: 9115
          initialDelaySeconds: 5
          timeoutSeconds: 5
          periodSeconds: 10
          successThreshold: 1
          failureThreshold: 3
```

svc.yaml

```yaml
kind: Service
apiVersion: v1
metadata:
  name: blackbox-exporter
  namespace: kube-system
spec:
  selector:
    app: blackbox-exporter
  ports:
    - name: blackbox-port
      protocol: TCP
      port: 9115
```

ingress.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: blackbox-exporter
  namespace: kube-system
spec:
  rules:
  - host: blackbox.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: blackbox-exporter
          servicePort: blackbox-port
```

解析域名

```shell
# 修改named
vim /var/named/od.com.zone

# 将 blackbox.od.com 解析到 10
blackbox           A    10.4.7.10

# 重启named
systemctl retart named

```

#### 2、安装 `prometheus-server`

准备镜像

```shell
docker pull prom/prometheus:v2.14.0
docker tag 7317640d555e harbor.od.com/infra/prometheus:v2.14.0
docker push harbor.od.com/infra/prometheus:v2.14.0
```

准备配置文件

```shell
# nfs持久存储
mkdir -pv /data/nfs-volume/prometheus/{etc,prom-db}

# 拷贝证书，需要与apiserver通信
cd /data/nfs-volume/prometheus/etc/
cp /opt/certs/ca.pem /opt/certs/client.pem /opt/certs/client-key.pem .
```

prometheus.yml

```yml
global:
  scrape_interval:     15s
  evaluation_interval: 15s
scrape_configs:

- job_name: 'etcd'
  tls_config:
    ca_file: /data/etc/ca.pem
    cert_file: /data/etc/client.pem
    key_file: /data/etc/client-key.pem
  scheme: https
  static_configs:        # 静态配置
  - targets:
    - '10.4.7.12:2379'
    - '10.4.7.21:2379'
    - '10.4.7.22:2379'

- job_name: 'kubernetes-apiservers'
  kubernetes_sd_configs:
  - role: endpoints
  scheme: https
  tls_config:
    ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
  bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
  relabel_configs:
  - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
    action: keep
    regex: default;kubernetes;https

- job_name: 'kubernetes-pods'
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
    action: keep
    regex: true
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
    action: replace
    target_label: __metrics_path__
    regex: (.+)
  - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
    action: replace
    regex: ([^:]+)(?::\d+)?;(\d+)
    replacement: $1:$2
    target_label: __address__
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name

- job_name: 'kubernetes-kubelet'
  kubernetes_sd_configs:
  - role: node
  relabel_configs:
  - action: labelmap
    regex: __meta_kubernetes_node_label_(.+)
  - source_labels: [__meta_kubernetes_node_name]
    regex: (.+)
    target_label: __address__
    replacement: ${1}:10255

- job_name: 'kubernetes-cadvisor'
  kubernetes_sd_configs:
  - role: node
  relabel_configs:
  - action: labelmap
    regex: __meta_kubernetes_node_label_(.+)
  - source_labels: [__meta_kubernetes_node_name]
    regex: (.+)
    target_label: __address__
    replacement: ${1}:4194
- job_name: 'kubernetes-kube-state'
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
  - source_labels: [__meta_kubernetes_pod_label_grafanak8sapp]
    regex: .*true.*
    action: keep
  - source_labels: ['__meta_kubernetes_pod_label_daemon', '__meta_kubernetes_pod_node_name']
    regex: 'node-exporter;(.*)'
    action: replace
    target_label: nodename

- job_name: 'blackbox_http_pod_probe'
  metrics_path: /probe
  kubernetes_sd_configs:
  - role: pod
  params:
    module: [http_2xx]
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_blackbox_scheme]
    action: keep
    regex: http
  - source_labels: [__address__, __meta_kubernetes_pod_annotation_blackbox_port,  __meta_kubernetes_pod_annotation_blackbox_path]
    action: replace
    regex: ([^:]+)(?::\d+)?;(\d+);(.+)
    replacement: $1:$2$3
    target_label: __param_target
  - action: replace
    target_label: __address__
    replacement: blackbox-exporter.kube-system:9115
  - source_labels: [__param_target]
    target_label: instance
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name

- job_name: 'blackbox_tcp_pod_probe'
  metrics_path: /probe
  kubernetes_sd_configs:
  - role: pod
  params:
    module: [tcp_connect]
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_blackbox_scheme]
    action: keep
    regex: tcp
  - source_labels: [__address__, __meta_kubernetes_pod_annotation_blackbox_port]
    action: replace
    regex: ([^:]+)(?::\d+)?;(\d+)
    replacement: $1:$2
    target_label: __param_target
  - action: replace
    target_label: __address__
    replacement: blackbox-exporter.kube-system:9115
  - source_labels: [__param_target]
    target_label: instance
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name

- job_name: 'traefik'
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scheme]
    action: keep
    regex: traefik
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
    action: replace
    target_label: __metrics_path__
    regex: (.+)
  - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
    action: replace
    regex: ([^:]+)(?::\d+)?;(\d+)
    replacement: $1:$2
    target_label: __address__
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
```

准备资源配置清单

```shell
# 资源配置清单
mkdir /data/k8s-yaml/prometheus
```

rbac.yaml

```
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: prometheus
  namespace: infra
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: prometheus
rules:
- apiGroups:
  - ""
  resources:
  - nodes
  - nodes/metrics
  - services
  - endpoints
  - pods
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - configmaps
  verbs:
  - get
- nonResourceURLs:
  - /metrics
  verbs:
  - get
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: infra
```

dp.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "5"
  labels:
    name: prometheus
  name: prometheus
  namespace: infra
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 7
  selector:
    matchLabels:
      app: prometheus
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      nodeName: hdss7-21.host.com       # 因为22资源提交的太多
      containers:
      - name: prometheus
        image: harbor.od.com/infra/prometheus:v2.14.0
        imagePullPolicy: IfNotPresent
        command:
        - /bin/prometheus
        args:
        - --config.file=/data/etc/prometheus.yml  # 配置文件
        - --storage.tsdb.path=/data/prom-db       
        - --storage.tsdb.min-block-duration=10m   # 只加载10分钟数据到缓存，生产环境可以加大 例：2h 
        - --storage.tsdb.retention=72h            # tsdb存储多久时间数据，生产环境可以加大 例：100h
        - --web.enable-lifecycle                  # 后期修改配置，可以curl url重新加载配置
        ports:
        - containerPort: 9090
          protocol: TCP
        volumeMounts:
        - mountPath: /data
          name: data
        resources:
          requests:         # 容器启动资源
            cpu: "1000m"    
            memory: "1.5Gi"
          limits:           # 容器最大资源
            cpu: "2000m"
            memory: "3Gi"
      imagePullSecrets:
      - name: harbor
      securityContext:
        runAsUser: 0
      serviceAccountName: prometheus
      volumes:
      - name: data
        nfs:
          server: hdss7-200
          path: /data/nfs-volume/prometheus
```

svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: infra
spec:
  ports:
  - port: 9090
    protocol: TCP
    targetPort: 9090
  selector:
    app: prometheus
```

ingress.yaml

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: traefik
  name: prometheus
  namespace: infra
spec:
  rules:
  - host: prometheus.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: prometheus
          servicePort: 9090
```

解析域名

```shell
# 7-11上修改named
vim /var/named/od.com.zone

# 将 prometheus.od.com 解析到 10
prometheus           A    10.4.7.10

# 重启named
systemctl retart named
```

#### 3、 `prometheus添加traefik监控`

spec->template->metadata下，添加annotations注释

```shell
cd /data/k8s-yaml/traefik/
vim daemonset.yaml 
```

```yaml
spec:
  selector:
    matchLabels:
      app: traefik
  template:
    metadata:
      name: traefik
      labels:
        app: traefik
      annotations:
        prometheus_io_scheme: "traefik"
        prometheus_io_path: "/metrics"
        prometheus_io_port: "8080"
```

#### 4、dubbo服务接入 `blackbox`存活检查

spec->template->metadata下，添加annotations注释

```shell
cd /data/k8s-yaml/test/dubbo-demo-service
vim dp.yaml 
```

```yaml
annotations:
    blackbox_port: "20880"
    blackbox_scheme: "tcp"
```

dubbo消费者加入存活检测

spec->template->metadata下，添加annotations注释

```
cd /data/k8s-yaml/test/dubbo-demo-consumer
vim dp.yaml
```

```yaml
annotations:
  blackbox_path: "/hello?name=health"
  blackbox_port: "8080"
  blackbox_scheme: "http"
```

dubbo服务接入jvm监控

服务提供者

```shell
cd /data/k8s-yaml/test/dubbo-demo-service
vim dp.yaml 
```

spec->template->metadata下，添加annotations注释

```yaml
annotations:
  prometheus_io_scrape: "true"
  prometheus_io_port: "12346"
  prometheus_io_path: "/"
```

服务消费者

```shell
cd /data/k8s-yaml/test/dubbo-demo-consumer
vim dp.yaml
```

spec->template->metadata下，添加annotations注释

```yaml
annotations:
  prometheus_io_scrape: "true"
  prometheus_io_port: "12346"
  prometheus_io_path: "/"
```



#### 5、安装grafana

##### 5.1、准备镜像

```shell
docker pull grafana/grafana:5.4.2
docker tag 6f18ddf9e552 harbor.od.com/infra/grafana:v5.4.2
docker push harbor.od.com/infra/grafana:v5.4.2
```

##### 5.2、准备资源配置清单

```shell
mkdir /data/k8s-yaml/grafana
cd /data/k8s-yaml/grafana
mkdir /data/nfs-volume/grafana
```

rbac.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: grafana
rules:
- apiGroups:
  - "*"
  resources:
  - namespaces
  - deployments
  - pods
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: grafana
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: grafana
subjects:
- kind: User
  name: k8s-node
```

dp.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: grafana
    name: grafana
  name: grafana
  namespace: infra
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 7
  selector:
    matchLabels:
      name: grafana
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: grafana
        name: grafana
    spec:
      containers:
      - name: grafana
        image: harbor.od.com/infra/grafana:v5.4.2
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3000
          protocol: TCP
        volumeMounts:
        - mountPath: /var/lib/grafana
          name: data
      imagePullSecrets:
      - name: harbor
      securityContext:
        runAsUser: 0
      volumes:
      - nfs:
          server: hdss7-200
          path: /data/nfs-volume/grafana
        name: data
```

svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: infra
spec:
  ports:
  - port: 3000
    protocol: TCP
    targetPort: 3000
  selector:
    app: grafana
```

ingress.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: grafana
  namespace: infra
spec:
  rules:
  - host: grafana.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: grafana
          servicePort: 3000
```

解析域名

```shell
# 7-11上修改named
vim /var/named/od.com.zone

# 将 grafana.od.com 解析到 10
grafana     A    10.4.7.10

# 重启named
systemctl retart named
```

账号密码

admin:admin123

##### 5.3、安装插件

 安装方法一：

```shell
# 进入grafana容器，执行以下命令
grafana-cli plugins install grafana-kubernetes-app
grafana-cli plugins install grafana-clock-panel
grafana-cli plugins install grafana-piechart-panel
grafana-cli plugins install briangann-gauge-panel
grafana-cli plugins install natel-discrete-panel

# 退出容器，然后重启grafana安装生效
kubectl delete pods xxxx -n infra
```

安装方法二：

```shell
# 进入 /data/nfs-volume/grafana/plugins
# 分别wget这些插件
```

##### 5.4、添加数据源

添加prometheus

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200701154932.png)

复制证书,save and test

```shell
cat /opt/certs/ca.pem
cat /opt/certs/client.pem
cat /opt/certs/client-key.pem
```

k8s插件配置

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200701172430.png)

复制证书

```shell
cat /opt/certs/ca.pem
cat /opt/certs/client.pem
cat /opt/certs/client-key.pem
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200701172515.png)

#### 6、安装alertmanager

准备镜像

```shell
docker pull quay.io/prometheus/alertmanager:v0.14.0
docker tag 23744b2d645c harbor.od.com/infra/alertmanager:v0.14.0
docker push harbor.od.com/infra/alertmanager:v0.14.0
```

准备资源配置清单

```shell
mkdir /data/k8s-yaml/alertmanager

```

cm.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: alertmanager-config
  namespace: infra
data:
  config.yml: |
    global:
      resolve_timeout: 5m
      smtp_from: 'XXXX@qq.com'
      smtp_smarthost: 'smtp.qq.com:465'
      smtp_auth_username: 'XXXX@qq.com'
      smtp_auth_password: 'passwd'
      smtp_require_tls: false
      smtp_hello: 'XXXX@qq.com'
    templates:   
      - '/etc/alertmanager/*.tmpl'
    route:
      group_by: ['alertname', 'cluster']
      group_wait: 20s     
      group_interval: 20s 
      repeat_interval: 12h 
      receiver: 'email' 
    receivers:
    - name: 'email'
      email_configs:
      - to: 'xubobovip@gmail.com'   
        send_resolved: true 
        html: '{{ template "email.to.html" . }}' 
        headers: { Subject: " {{ .CommonLabels.instance }} {{ .CommonAnnotations.summary }}" }   
  email.tmpl: |
    {{ define "email.to.html" }}
    {{- if gt (len .Alerts.Firing) 0 -}}
    {{ range .Alerts }}
    告警程序: prometheus_alert <br>
    告警级别: {{ .Labels.severity }} <br>
    告警类型: {{ .Labels.alertname }} <br>
    故障主机: {{ .Labels.instance }} <br>
    告警主题: {{ .Annotations.summary }}  <br>
    触发时间: {{ .StartsAt.Format "2006-01-02 15:04:05" }} <br>
    {{ end }}{{ end -}}
    
    {{- if gt (len .Alerts.Resolved) 0 -}}
    {{ range .Alerts }}
    告警程序: prometheus_alert <br>
    告警级别: {{ .Labels.severity }} <br>
    告警类型: {{ .Labels.alertname }} <br>
    故障主机: {{ .Labels.instance }} <br>
    告警主题: {{ .Annotations.summary }} <br>
    触发时间: {{ .StartsAt.Format "2006-01-02 15:04:05" }} <br>
    恢复时间: {{ .EndsAt.Format "2006-01-02 15:04:05" }} <br>
    {{ end }}{{ end -}}
    
    {{- end }}
```

dp.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: alertmanager
  namespace: infra
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alertmanager
  template:
    metadata:
      labels:
        app: alertmanager
    spec:
      containers:
      - name: alertmanager
        image: harbor.od.com/infra/alertmanager:v0.14.0
        args:
          - "--config.file=/etc/alertmanager/config.yml"
          - "--storage.path=/alertmanager"
        ports:
        - name: alertmanager
          containerPort: 9093
        volumeMounts:
        - name: alertmanager-cm
          mountPath: /etc/alertmanager
      volumes:
      - name: alertmanager-cm
        configMap:
          name: alertmanager-config
      imagePullSecrets:
      - name: harbor
```

svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: alertmanager
  namespace: infra
spec:
  selector:
    app: alertmanager
  ports:
    - port: 80
      targetPort: 9093
```

报警配置

```shell
vim /data/nfs-volume/prometheus/etc/rules.yml
```

rules.yml

```yml
groups:
- name: hostStatsAlert
  rules:
  - alert: hostCpuUsageAlert
    expr: sum(avg without (cpu)(irate(node_cpu{mode!='idle'}[5m]))) by (instance) > 0.85
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "{{ $labels.instance }} CPU usage above 85% (current value: {{ $value }}%)"
  - alert: hostMemUsageAlert
    expr: (node_memory_MemTotal - node_memory_MemAvailable)/node_memory_MemTotal > 0.85
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "{{ $labels.instance }} MEM usage above 85% (current value: {{ $value }}%)"
  - alert: OutOfInodes
    expr: node_filesystem_free{fstype="overlay",mountpoint ="/"} / node_filesystem_size{fstype="overlay",mountpoint ="/"} * 100 < 10
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Out of inodes (instance {{ $labels.instance }})"
      description: "Disk is almost running out of available inodes (< 10% left) (current value: {{ $value }})"
  - alert: OutOfDiskSpace
    expr: node_filesystem_free{fstype="overlay",mountpoint ="/rootfs"} / node_filesystem_size{fstype="overlay",mountpoint ="/rootfs"} * 100 < 10
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Out of disk space (instance {{ $labels.instance }})"
      description: "Disk is almost full (< 10% left) (current value: {{ $value }})"
  - alert: UnusualNetworkThroughputIn
    expr: sum by (instance) (irate(node_network_receive_bytes[2m])) / 1024 / 1024 > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual network throughput in (instance {{ $labels.instance }})"
      description: "Host network interfaces are probably receiving too much data (> 100 MB/s) (current value: {{ $value }})"
  - alert: UnusualNetworkThroughputOut
    expr: sum by (instance) (irate(node_network_transmit_bytes[2m])) / 1024 / 1024 > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual network throughput out (instance {{ $labels.instance }})"
      description: "Host network interfaces are probably sending too much data (> 100 MB/s) (current value: {{ $value }})"
  - alert: UnusualDiskReadRate
    expr: sum by (instance) (irate(node_disk_bytes_read[2m])) / 1024 / 1024 > 50
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual disk read rate (instance {{ $labels.instance }})"
      description: "Disk is probably reading too much data (> 50 MB/s) (current value: {{ $value }})"
  - alert: UnusualDiskWriteRate
    expr: sum by (instance) (irate(node_disk_bytes_written[2m])) / 1024 / 1024 > 50
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual disk write rate (instance {{ $labels.instance }})"
      description: "Disk is probably writing too much data (> 50 MB/s) (current value: {{ $value }})"
  - alert: UnusualDiskReadLatency
    expr: rate(node_disk_read_time_ms[1m]) / rate(node_disk_reads_completed[1m]) > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual disk read latency (instance {{ $labels.instance }})"
      description: "Disk latency is growing (read operations > 100ms) (current value: {{ $value }})"
  - alert: UnusualDiskWriteLatency
    expr: rate(node_disk_write_time_ms[1m]) / rate(node_disk_writes_completedl[1m]) > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual disk write latency (instance {{ $labels.instance }})"
      description: "Disk latency is growing (write operations > 100ms) (current value: {{ $value }})"
- name: http_status
  rules:
  - alert: ProbeFailed
    expr: probe_success == 0
    for: 1m
    labels:
      severity: error
    annotations:
      summary: "Probe failed (instance {{ $labels.instance }})"
      description: "Probe failed (current value: {{ $value }})"
  - alert: StatusCode
    expr: probe_http_status_code <= 199 OR probe_http_status_code >= 400
    for: 1m
    labels:
      severity: error
    annotations:
      summary: "Status Code (instance {{ $labels.instance }})"
      description: "HTTP status code is not 200-399 (current value: {{ $value }})"
  - alert: SslCertificateWillExpireSoon
    expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 30
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "SSL certificate will expire soon (instance {{ $labels.instance }})"
      description: "SSL certificate expires in 30 days (current value: {{ $value }})"
  - alert: SslCertificateHasExpired
    expr: probe_ssl_earliest_cert_expiry - time()  <= 0
    for: 5m
    labels:
      severity: error
    annotations:
      summary: "SSL certificate has expired (instance {{ $labels.instance }})"
      description: "SSL certificate has expired already (current value: {{ $value }})"
  - alert: BlackboxSlowPing
    expr: probe_icmp_duration_seconds > 2
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Blackbox slow ping (instance {{ $labels.instance }})"
      description: "Blackbox ping took more than 2s (current value: {{ $value }})"
  - alert: BlackboxSlowRequests
    expr: probe_http_duration_seconds > 2
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Blackbox slow requests (instance {{ $labels.instance }})"
      description: "Blackbox request took more than 2s (current value: {{ $value }})"
  - alert: PodCpuUsagePercent
    expr: sum(sum(label_replace(irate(container_cpu_usage_seconds_total[1m]),"pod","$1","container_label_io_kubernetes_pod_name", "(.*)"))by(pod) / on(pod) group_right kube_pod_container_resource_limits_cpu_cores *100 )by(container,namespace,node,pod,severity) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Pod cpu usage percent has exceeded 80% (current value: {{ $value }}%)"
```

prometheus添加报警

```shell
vim /data/nfs-volume/prometheus/etc/prometheus.yml
```

prometheus.yml

```yml
global:
  scrape_interval:     15s
  evaluation_interval: 15s
alerting:
  alertmanagers:
    - static_configs:
        - targets: ["alertmanager"]
rule_files:
- "/data/etc/rules.yml"
```

刷新配置

```shell
 curl -X POST http://prometheus.od.com/-/reload
```

测试报警

把dubbo服务的提供者缩容为0

把dubbo服务的提供者恢复为1