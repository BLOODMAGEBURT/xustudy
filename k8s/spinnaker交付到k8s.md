### spinnaker交付到k8s

#### 1、安装基础服务

安装之一：minio

```shell
docker pull minio/minio:latest
docker tag 33fd513a5dd2 harbor.od.com/armory/minio:latest
docker push harbor.od.com/armory/minio:latest
mkdir -p /data/k8s-yaml/armory/minio
mkdir -p /data/nfs-volume/minio
```

准备资源配置清单

```shell
# 创建armory命名空间
kubectl create ns armory

# 创建harbor仓库secrets
 kubectl create secret docker-registry harbor --docker-server=harbor.od.com --docker-username=admin --docker-password=Harbor12345 -n armory
```

dp.yaml

```yaml
kind: Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    name: minio
  name: minio
  namespace: armory
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 7
  selector:
    matchLabels:
      name: minio
  template:
    metadata:
      labels:
        app: minio
        name: minio
    spec:
      containers:
      - name: minio
        image: harbor.od.com/armory/minio:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9000
          protocol: TCP
        args:
        - server
        - /data
        env:
        - name: MINIO_ACCESS_KEY
          value: admin
        - name: MINIO_SECRET_KEY
          value: admin123
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /minio/health/ready
            port: 9000
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 5
        volumeMounts:
        - mountPath: /data
          name: data
      imagePullSecrets:
      - name: harbor
      volumes:
      - nfs:
          server: hdss7-200
          path: /data/nfs-volume/minio
        name: data 
```

svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: minio
  namespace: armory
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 9000
  selector:
    app: minio
```

Ingress.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: monio
  namespace: armory
spec:
  rules:
    - host: minio.od.com
      http: 
        paths:
        - path: /
          backend:
            serviceName: minio
            servicePort: 80
```

安装之二：redis

```shell
docker pull redis:4.0.14                                                              
docker tag 6e221e67453d harbor.od.com/armory/redis:v4.0.14
docker push harbor.od.com/armory/redis:v4.0.14

mkdir /data/k8s-yaml/armory/redis
```

dp.yaml

```yaml
kind: Deployment
apiVersion: apps:v1
kind: Deployment
metadata:
  labels:
    name: redis
  name: redis
  namespace: armory
spec:
  replicas: 1
  revisionHistoryLimit: 7
  selector:
    matchLabels:
      name: redis
  template:
    metadata:
      labels:
        app: redis
        name: redis
    spec:
      containers:
      - name: redis
        image: harbor.od.com/armory/redis:v4.0.14
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 6379
          protocol: TCP
      imagePullSecrets:
      - name: harbor
```

svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: armory
spec:
  ports:
  - port: 6379
    protocol: TCP
    targetPort: 6379
  selector:
    app: redis
```

#### 2、部署CloudDriver

准备镜像

```shell
docker pull armory/spinnaker-clouddriver-slim:release-1.8.x-14c9664
docker tag xxxxxx harbor.od.com/armory/clouddriver:v1.8.x
docker push harbor.od.com/armory/clouddriver:v1.8.x
mkdir /data/k8s-yaml/armory/clouddriver

vim credentials
[default]
aws_access_key_id=admin
aws_secret_access_key=admin123
```

创建secret

任意运算节点

```shell
wget http://k8s-yaml.od.com/armory/clouddriver/credentials
kubectl create secret generic credentials --from-file=./credentials -n armory
```

