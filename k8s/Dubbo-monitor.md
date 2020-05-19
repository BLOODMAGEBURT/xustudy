### `Dubbo-monitor`配置文件

> 地址：[github](https://github.com/Jeromefromcn/dubbo-monitor)

`dp.yaml`

```yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: dubbo-monitor
  namespace: infra
  labels:
    name: dubbo-monitor
spec:
  selector:
    matchLabels:
      name: dubbo-monitor
  template:
    metadata:
      labels:
        app: dubbo-monitor
        name: dubbo-monitor
    spec:
      containers:
      - name: dubbo-monitor
        image: harbor.od.com/infra/dubbo-monitor:latest
        ports:
        - containerPort: 8080
          protocol: TCP
        - containerPort: 20880
          protocol: TCP
        imagePullPolicy: IfNotPresent
      imagePullSecrets:
      - name: harbor
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      securityContext:
        runAsUser: 0
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  revisionHistoryLimit: 7
  progressDeadlineSeconds: 600
```

`svc.yaml`

```yaml
kind: Service
apiVersion: v1
metadata: 
  name: dubbo-monitor
  namespace: infra
spec:
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
  selector:
    app: dubbo-monitor
```

`ingress.yaml`

```yaml
kind: Ingress
apiVersion: extensions/v1beta1
metadata:
  name: dubbo-monitor
  namespace: infra
spec:
  rules:
  - host: dubbo-monitor.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: dubbo-monitor
          servicePort: 8080
```

####  `Dubbo-demo-consumer`配置文件

`dp.yaml`

```yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: dubbo-demo-consumer
  namespace: app
  labels:
    name: dubbo-demo-consumer
spec:
  replicas: 1
  selector:
    matchLabels:
      name: dubbo-demo-consumer
  template:
    metadata:
      labels:
        app: dubbo-demo-consumer
        name: dubbo-demo-consumer
    spec:
      containers:
      - name: dubbo-demo-consumer
        image: harbor.od.com/app/dubbo-demo-consumer:master_20200519_1100
        ports:
        - containerPort: 8080
          protocol: TCP
        - containerPort: 20880
          protocol: TCP
        env: 
        - name: JAR_BALL
          value: dubbo-client.jar
        imagePullPolicy: IfNotPresent
      imagePullSecrets:
      - name: harbor
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      securityContext:
        runAsUser: 0
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  revisionHistoryLimit: 7
  progressDeadlineSeconds: 600
```

`svc.yaml`

```yaml
kind: Service
apiVersion: v1
metadata:
  name: dubbo-demo-consumer
  namespace: app
spec:
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
  selector:
    app: dubbo-demo-consumer
```

`ingress.yaml`

```yaml
kind: Ingress
apiVersion: extensions/v1beta1
metadata: 
  name: dubbo-demo-consumer
  namespace: app
spec:
  rules:
  - host: demo.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: dubbo-demo-consumer
          servicePort: 8080

```

