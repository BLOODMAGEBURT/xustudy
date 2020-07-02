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
docker tag 191c4017dcdd harbor.od.com/armory/redis:v4.0.14
docker push harbor.od.com/armory/redis:v4.0.14

mkdir /data/k8s-yaml/armory/redis
```

dp.yaml

```yaml
kind: Deployment
apiVersion: apps/v1
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

##### 2.1、准备镜像

```shell
docker pull armory/spinnaker-clouddriver-slim:release-1.8.x-14c9664
docker tag edb2507fdb62 harbor.od.com/armory/clouddriver:v1.8.x
docker push harbor.od.com/armory/clouddriver:v1.8.x
mkdir /data/k8s-yaml/armory/clouddriver
cd /data/k8s-yaml/armory/clouddriver
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

##### 2.2、准备k8s用户

签发证书与私钥

```shell
cd /opt/certs
vim admin-csr.json
```

admin-csr.json

```json
{
    "CN": "cluster-admin",
    "hosts": [
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "beijing",
            "L": "beijing",
            "O": "od",
            "OU": "ops"
        }
    ]
}
```

签发证书

```shell
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client admin-csr.json |cfssl-json -bare admin
```

做kubeconfig配置

任意运算节点

```shell
scp root@hdss7-200:/opt/certs/ca.pem .
scp root@hdss7-200:/opt/certs/admin.pem .
scp root@hdss7-200:/opt/certs/admin-key.pem .
```

创建UA四步骤

```shell
# set-cluster
kubectl config set-cluster myk8s --certificate-authority=./ca.pem --embed-certs=true --server=https://10.4.7.10:7443 --kubeconfig=config

# set-credentials
kubectl config set-credentials cluster-admin --client-certificate=./admin.pem --client-key=./admin-key.pem --embed-certs=true --kubeconfig=config

# set-context
kubectl config set-context myk8s-context --cluster=myk8s --user=cluster-admin --kubeconfig=config

# use-context
kubectl config use-context myk8s-context --kubeconfig=config

# 集群角色绑定用户
kubectl create clusterrolebinding myk8s-admin --clusterrole=cluster-admin --user=cluster-admin
```

创建ConfigMap配置

```shell
cp config default-kubeconfig
kubectl create cm default-kubeconfig --from-file=default-kubeconfig -n armory
```

##### 3、准备资源配置清单

init-env.yaml    --环境变量配置文件

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: init-env
  namespace: armory
data:
  API_HOST: http://spinnaker.od.com/api
  ARMORY_ID: c02f0781-92f5-4e80-86db-0ba8fe7b8544
  ARMORYSPINNAKER_CONF_STORE_BUCKET: armory-platform
  ARMORYSPINNAKER_CONF_STORE_PREFIX: front50
  ARMORYSPINNAKER_GCS_ENABLED: "false"
  ARMORYSPINNAKER_S3_ENABLED: "true"
  AUTH_ENABLED: "false"
  AWS_REGION: us-east-1
  BASE_IP: 127.0.0.1
  CLOUDDRIVER_OPTS: -Dspring.profiles.active=armory,configurator,local
  CONFIGURATOR_ENABLED: "false"
  DECK_HOST: http://spinnaker.od.com
  ECHO_OPTS: -Dspring.profiles.active=armory,configurator,local
  GATE_OPTS: -Dspring.profiles.active=armory,configurator,local
  IGOR_OPTS: -Dspring.profiles.active=armory,configurator,local
  PLATFORM_ARCHITECTURE: k8s
  REDIS_HOST: redis://redis:6379
  SERVER_ADDRESS: 0.0.0.0
  SPINNAKER_AWS_DEFAULT_REGION: us-east-1
  SPINNAKER_AWS_ENABLED: "false"
  SPINNAKER_CONFIG_DIR: /home/spinnaker/config
  SPINNAKER_GOOGLE_PROJECT_CREDENTIALS_PATH: ""
  SPINNAKER_HOME: /home/spinnaker
  SPRING_PROFILES_ACTIVE: armory,configurator,local
```

custom-config.yaml   --组件配置

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: custom-config
  namespace: armory
data:
  clouddriver-local.yml: |
    kubernetes:
      enabled: true
      accounts:
        - name: cluster-admin
          serviceAccount: false
          dockerRegistries:
            - accountName: harbor
              namespace: []
          namespaces:
            - test
            - prod
          kubeconfigFile: /opt/spinnaker/credentials/custom/default-kubeconfig
      primaryAccount: cluster-admin
    dockerRegistry:
      enabled: true
      accounts:
        - name: harbor
          requiredGroupMembership: []
          providerVersion: V1
          insecureRegistry: true
          address: http://harbor.od.com
          username: admin
          password: Harbor12345
      primaryAccount: harbor
    artifacts:
      s3:
        enabled: true
        accounts:
        - name: armory-config-s3-account
          apiEndpoint: http://minio
          apiRegion: us-east-1
      gcs:
        enabled: false
        accounts:
        - name: armory-config-gcs-account
  custom-config.json: ""
  echo-configurator.yml: |
    diagnostics:
      enabled: true
  front50-local.yml: |
    spinnaker:
      s3:
        endpoint: http://minio
  igor-local.yml: |
    jenkins:
      enabled: true
      masters:
        - name: jenkins-admin
          address: http://jenkins.od.com
          username: admin
          password: admin123
      primaryAccount: jenkins-admin
  nginx.conf: |
    gzip on;
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/vnd.ms-fontobject application/x-font-ttf font/opentype image/svg+xml image/x-icon;
    server {
           listen 80;
           location / {
                proxy_pass http://armory-deck/;
           }
           location /api/ {
                proxy_pass http://armory-gate:8084/;
           }
           rewrite ^/login(.*)$ /api/login$1 last;
           rewrite ^/auth(.*)$ /api/auth$1 last;
    }
  spinnaker-local.yml: |
    services:
      igor:
        enabled: true
```

default-config.yaml

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: default-config
  namespace: armory
data:
  barometer.yml: |
    server:
      port: 9092

    spinnaker:
      redis:
        host: ${services.redis.host}
        port: ${services.redis.port}
  clouddriver-armory.yml: |
    aws:
      defaultAssumeRole: role/${SPINNAKER_AWS_DEFAULT_ASSUME_ROLE:SpinnakerManagedProfile}
      accounts:
        - name: default-aws-account
          accountId: ${SPINNAKER_AWS_DEFAULT_ACCOUNT_ID:none}

      client:
        maxErrorRetry: 20

    serviceLimits:
      cloudProviderOverrides:
        aws:
          rateLimit: 15.0

      implementationLimits:
        AmazonAutoScaling:
          defaults:
            rateLimit: 3.0
        AmazonElasticLoadBalancing:
          defaults:
            rateLimit: 5.0

    security.basic.enabled: false
    management.security.enabled: false
  clouddriver-dev.yml: |

    serviceLimits:
      defaults:
        rateLimit: 2
  clouddriver.yml: |
    server:
      port: ${services.clouddriver.port:7002}
      address: ${services.clouddriver.host:localhost}

    redis:
      connection: ${REDIS_HOST:redis://localhost:6379}

    udf:
      enabled: ${services.clouddriver.aws.udf.enabled:true}
      udfRoot: /opt/spinnaker/config/udf
      defaultLegacyUdf: false

    default:
      account:
        env: ${providers.aws.primaryCredentials.name}

    aws:
      enabled: ${providers.aws.enabled:false}
      defaults:
        iamRole: ${providers.aws.defaultIAMRole:BaseIAMRole}
      defaultRegions:
        - name: ${providers.aws.defaultRegion:us-east-1}
      defaultFront50Template: ${services.front50.baseUrl}
      defaultKeyPairTemplate: ${providers.aws.defaultKeyPairTemplate}

    azure:
      enabled: ${providers.azure.enabled:false}

      accounts:
        - name: ${providers.azure.primaryCredentials.name}
          clientId: ${providers.azure.primaryCredentials.clientId}
          appKey: ${providers.azure.primaryCredentials.appKey}
          tenantId: ${providers.azure.primaryCredentials.tenantId}
          subscriptionId: ${providers.azure.primaryCredentials.subscriptionId}

    google:
      enabled: ${providers.google.enabled:false}

      accounts:
        - name: ${providers.google.primaryCredentials.name}
          project: ${providers.google.primaryCredentials.project}
          jsonPath: ${providers.google.primaryCredentials.jsonPath}
          consul:
            enabled: ${providers.google.primaryCredentials.consul.enabled:false}

    cf:
      enabled: ${providers.cf.enabled:false}

      accounts:
        - name: ${providers.cf.primaryCredentials.name}
          api: ${providers.cf.primaryCredentials.api}
          console: ${providers.cf.primaryCredentials.console}
          org: ${providers.cf.defaultOrg}
          space: ${providers.cf.defaultSpace}
          username: ${providers.cf.account.name:}
          password: ${providers.cf.account.password:}

    kubernetes:
      enabled: ${providers.kubernetes.enabled:false}
      accounts:
        - name: ${providers.kubernetes.primaryCredentials.name}
          dockerRegistries:
            - accountName: ${providers.kubernetes.primaryCredentials.dockerRegistryAccount}

    openstack:
      enabled: ${providers.openstack.enabled:false}
      accounts:
        - name: ${providers.openstack.primaryCredentials.name}
          authUrl: ${providers.openstack.primaryCredentials.authUrl}
          username: ${providers.openstack.primaryCredentials.username}
          password: ${providers.openstack.primaryCredentials.password}
          projectName: ${providers.openstack.primaryCredentials.projectName}
          domainName: ${providers.openstack.primaryCredentials.domainName:Default}
          regions: ${providers.openstack.primaryCredentials.regions}
          insecure: ${providers.openstack.primaryCredentials.insecure:false}
          userDataFile: ${providers.openstack.primaryCredentials.userDataFile:}

          lbaas:
            pollTimeout: 60
            pollInterval: 5

    dockerRegistry:
      enabled: ${providers.dockerRegistry.enabled:false}
      accounts:
        - name: ${providers.dockerRegistry.primaryCredentials.name}
          address: ${providers.dockerRegistry.primaryCredentials.address}
          username: ${providers.dockerRegistry.primaryCredentials.username:}
          passwordFile: ${providers.dockerRegistry.primaryCredentials.passwordFile}

    credentials:
      primaryAccountTypes: ${providers.aws.primaryCredentials.name}, ${providers.google.primaryCredentials.name}, ${providers.cf.primaryCredentials.name}, ${providers.azure.primaryCredentials.name}
      challengeDestructiveActionsEnvironments: ${providers.aws.primaryCredentials.name}, ${providers.google.primaryCredentials.name}, ${providers.cf.primaryCredentials.name}, ${providers.azure.primaryCredentials.name}

    spectator:
      applicationName: ${spring.application.name}
      webEndpoint:
        enabled: ${services.spectator.webEndpoint.enabled:false}
        prototypeFilter:
          path: ${services.spectator.webEndpoint.prototypeFilter.path:}

      stackdriver:
        enabled: ${services.stackdriver.enabled}
        projectName: ${services.stackdriver.projectName}
        credentialsPath: ${services.stackdriver.credentialsPath}

    stackdriver:
      hints:
        - name: controller.invocations
          labels:
          - account
          - region
  dinghy.yml: ""
  echo-armory.yml: |
    diagnostics:
      enabled: true
      id: ${ARMORY_ID:unknown}

    armorywebhooks:
      enabled: false
      forwarding:
        baseUrl: http://armory-dinghy:8081
        endpoint: v1/webhooks
  echo-noncron.yml: |
    scheduler:
      enabled: false
  echo.yml: |
    server:
      port: ${services.echo.port:8089}
      address: ${services.echo.host:localhost}

    cassandra:
      enabled: ${services.echo.cassandra.enabled:false}
      embedded: ${services.cassandra.embedded:false}
      host: ${services.cassandra.host:localhost}

    spinnaker:
      baseUrl: ${services.deck.baseUrl}
      cassandra:
         enabled: ${services.echo.cassandra.enabled:false}
      inMemory:
         enabled: ${services.echo.inMemory.enabled:true}

    front50:
      baseUrl: ${services.front50.baseUrl:http://localhost:8080 }

    orca:
      baseUrl: ${services.orca.baseUrl:http://localhost:8083 }

    endpoints.health.sensitive: false

    slack:
      enabled: ${services.echo.notifications.slack.enabled:false}
      token: ${services.echo.notifications.slack.token}

    spring:
      mail:
        host: ${mail.host}

    mail:
      enabled: ${services.echo.notifications.mail.enabled:false}
      host: ${services.echo.notifications.mail.host}
      from: ${services.echo.notifications.mail.fromAddress}

    hipchat:
      enabled: ${services.echo.notifications.hipchat.enabled:false}
      baseUrl: ${services.echo.notifications.hipchat.url}
      token: ${services.echo.notifications.hipchat.token}

    twilio:
      enabled: ${services.echo.notifications.sms.enabled:false}
      baseUrl: ${services.echo.notifications.sms.url:https://api.twilio.com/ }
      account: ${services.echo.notifications.sms.account}
      token: ${services.echo.notifications.sms.token}
      from: ${services.echo.notifications.sms.from}

    scheduler:
      enabled: ${services.echo.cron.enabled:true}
      threadPoolSize: 20
      triggeringEnabled: true
      pipelineConfigsPoller:
        enabled: true
        pollingIntervalMs: 30000
      cron:
        timezone: ${services.echo.cron.timezone}

    spectator:
      applicationName: ${spring.application.name}
      webEndpoint:
        enabled: ${services.spectator.webEndpoint.enabled:false}
        prototypeFilter:
          path: ${services.spectator.webEndpoint.prototypeFilter.path:}

      stackdriver:
        enabled: ${services.stackdriver.enabled}
        projectName: ${services.stackdriver.projectName}
        credentialsPath: ${services.stackdriver.credentialsPath}

    webhooks:
      artifacts:
        enabled: true
  fetch.sh: |+
  
    CONFIG_LOCATION=${SPINNAKER_HOME:-"/opt/spinnaker"}/config
    CONTAINER=$1

    rm -f /opt/spinnaker/config/*.yml

    mkdir -p ${CONFIG_LOCATION}

    for filename in /opt/spinnaker/config/default/*.yml; do
        cp $filename ${CONFIG_LOCATION}
    done

    if [ -d /opt/spinnaker/config/custom ]; then
        for filename in /opt/spinnaker/config/custom/*; do
            cp $filename ${CONFIG_LOCATION}
        done
    fi

    add_ca_certs() {
      ca_cert_path="$1"
      jks_path="$2"
      alias="$3"

      if [[ "$(whoami)" != "root" ]]; then
        echo "INFO: I do not have proper permisions to add CA roots"
        return
      fi

      if [[ ! -f ${ca_cert_path} ]]; then
        echo "INFO: No CA cert found at ${ca_cert_path}"
        return
      fi
      keytool -importcert \
          -file ${ca_cert_path} \
          -keystore ${jks_path} \
          -alias ${alias} \
          -storepass changeit \
          -noprompt
    }

    if [ `which keytool` ]; then
      echo "INFO: Keytool found adding certs where appropriate"
      add_ca_certs "${CONFIG_LOCATION}/ca.crt" "/etc/ssl/certs/java/cacerts" "custom-ca"
    else
      echo "INFO: Keytool not found, not adding any certs/private keys"
    fi

    saml_pem_path="/opt/spinnaker/config/custom/saml.pem"
    saml_pkcs12_path="/tmp/saml.pkcs12"
    saml_jks_path="${CONFIG_LOCATION}/saml.jks"

    x509_ca_cert_path="/opt/spinnaker/config/custom/x509ca.crt"
    x509_client_cert_path="/opt/spinnaker/config/custom/x509client.crt"
    x509_jks_path="${CONFIG_LOCATION}/x509.jks"
    x509_nginx_cert_path="/opt/nginx/certs/ssl.crt"

    if [ "${CONTAINER}" == "gate" ]; then
        if [ -f ${saml_pem_path} ]; then
            echo "Loading ${saml_pem_path} into ${saml_jks_path}"
            openssl pkcs12 -export -out ${saml_pkcs12_path} -in ${saml_pem_path} -password pass:changeit -name saml
            keytool -genkey -v -keystore ${saml_jks_path} -alias saml \
                    -keyalg RSA -keysize 2048 -validity 10000 \
                    -storepass changeit -keypass changeit -dname "CN=armory"
            keytool -importkeystore \
                    -srckeystore ${saml_pkcs12_path} \
                    -srcstoretype PKCS12 \
                    -srcstorepass changeit \
                    -destkeystore ${saml_jks_path} \
                    -deststoretype JKS \
                    -storepass changeit \
                    -alias saml \
                    -destalias saml \
                    -noprompt
        else
            echo "No SAML IDP pemfile found at ${saml_pem_path}"
        fi
        if [ -f ${x509_ca_cert_path} ]; then
            echo "Loading ${x509_ca_cert_path} into ${x509_jks_path}"
            add_ca_certs ${x509_ca_cert_path} ${x509_jks_path} "ca"
        else
            echo "No x509 CA cert found at ${x509_ca_cert_path}"
        fi
        if [ -f ${x509_client_cert_path} ]; then
            echo "Loading ${x509_client_cert_path} into ${x509_jks_path}"
            add_ca_certs ${x509_client_cert_path} ${x509_jks_path} "client"
        else
            echo "No x509 Client cert found at ${x509_client_cert_path}"
        fi

        if [ -f ${x509_nginx_cert_path} ]; then
            echo "Creating a self-signed CA (EXPIRES IN 360 DAYS) with java keystore: ${x509_jks_path}"
            echo -e "\n\n\n\n\n\ny\n" | keytool -genkey -keyalg RSA -alias server -keystore keystore.jks -storepass changeit -validity 360 -keysize 2048
            keytool -importkeystore \
                    -srckeystore keystore.jks \
                    -srcstorepass changeit \
                    -destkeystore "${x509_jks_path}" \
                    -storepass changeit \
                    -srcalias server \
                    -destalias server \
                    -noprompt
        else
            echo "No x509 nginx cert found at ${x509_nginx_cert_path}"
        fi
    fi

    if [ "${CONTAINER}" == "nginx" ]; then
        nginx_conf_path="/opt/spinnaker/config/default/nginx.conf"
        if [ -f ${nginx_conf_path} ]; then
            cp ${nginx_conf_path} /etc/nginx/nginx.conf
        fi
    fi

  fiat.yml: |-
    server:
      port: ${services.fiat.port:7003}
      address: ${services.fiat.host:localhost}

    redis:
      connection: ${services.redis.connection:redis://localhost:6379}

    spectator:
      applicationName: ${spring.application.name}
      webEndpoint:
        enabled: ${services.spectator.webEndpoint.enabled:false}
        prototypeFilter:
          path: ${services.spectator.webEndpoint.prototypeFilter.path:}
      stackdriver:
        enabled: ${services.stackdriver.enabled}
        projectName: ${services.stackdriver.projectName}
        credentialsPath: ${services.stackdriver.credentialsPath}

    hystrix:
     command:
       default.execution.isolation.thread.timeoutInMilliseconds: 20000

    logging:
      level:
        com.netflix.spinnaker.fiat: DEBUG
  front50-armory.yml: |
    spinnaker:
      redis:
        enabled: true
        host: redis
  front50.yml: |
    server:
      port: ${services.front50.port:8080}
      address: ${services.front50.host:localhost}

    hystrix:
      command:
        default.execution.isolation.thread.timeoutInMilliseconds: 15000

    cassandra:
      enabled: ${services.front50.cassandra.enabled:false}
      embedded: ${services.cassandra.embedded:false}
      host: ${services.cassandra.host:localhost}

    aws:
      simpleDBEnabled: ${providers.aws.simpleDBEnabled:false}
      defaultSimpleDBDomain: ${providers.aws.defaultSimpleDBDomain}

    spinnaker:
      cassandra:
        enabled: ${services.front50.cassandra.enabled:false}
        host: ${services.cassandra.host:localhost}
        port: ${services.cassandra.port:9042}
        cluster: ${services.cassandra.cluster:CASS_SPINNAKER}
        keyspace: front50
        name: global

      redis:
        enabled: ${services.front50.redis.enabled:false}

      gcs:
        enabled: ${services.front50.gcs.enabled:false}
        bucket: ${services.front50.storage_bucket:}
        bucketLocation: ${services.front50.bucket_location:}
        rootFolder: ${services.front50.rootFolder:front50}
        project: ${providers.google.primaryCredentials.project}
        jsonPath: ${providers.google.primaryCredentials.jsonPath}

      s3:
        enabled: ${services.front50.s3.enabled:false}
        bucket: ${services.front50.storage_bucket:}
        rootFolder: ${services.front50.rootFolder:front50}

    spectator:
      applicationName: ${spring.application.name}
      webEndpoint:
        enabled: ${services.spectator.webEndpoint.enabled:false}
        prototypeFilter:
          path: ${services.spectator.webEndpoint.prototypeFilter.path:}

      stackdriver:
        enabled: ${services.stackdriver.enabled}
        projectName: ${services.stackdriver.projectName}
        credentialsPath: ${services.stackdriver.credentialsPath}

    stackdriver:
      hints:
        - name: controller.invocations
          labels:
          - application
          - cause
        - name: aws.request.httpRequestTime
          labels:
          - status
          - exception
          - AWSErrorCode
        - name: aws.request.requestSigningTime
          labels:
          - exception
  gate-armory.yml: |+
    lighthouse:
        baseUrl: http://${DEFAULT_DNS_NAME:lighthouse}:5000

  gate.yml: |
    server:
      port: ${services.gate.port:8084}
      address: ${services.gate.host:localhost}

    redis:
      connection: ${REDIS_HOST:redis://localhost:6379}
      configuration:
        secure: true

    spectator:
      applicationName: ${spring.application.name}
      webEndpoint:
        enabled: ${services.spectator.webEndpoint.enabled:false}
        prototypeFilter:
          path: ${services.spectator.webEndpoint.prototypeFilter.path:}

      stackdriver:
        enabled: ${services.stackdriver.enabled}
        projectName: ${services.stackdriver.projectName}
        credentialsPath: ${services.stackdriver.credentialsPath}

    stackdriver:
      hints:
        - name: EurekaOkClient_Request
          labels:
          - cause
          - reason
          - status
  igor-nonpolling.yml: |
    jenkins:
      polling:
        enabled: false
  igor.yml: |
    server:
      port: ${services.igor.port:8088}
      address: ${services.igor.host:localhost}

    jenkins:
      enabled: ${services.jenkins.enabled:false}
      masters:
        - name: ${services.jenkins.defaultMaster.name}
          address: ${services.jenkins.defaultMaster.baseUrl}
          username: ${services.jenkins.defaultMaster.username}
          password: ${services.jenkins.defaultMaster.password}
          csrf: ${services.jenkins.defaultMaster.csrf:false}
          
    travis:
      enabled: ${services.travis.enabled:false}
      masters:
        - name: ${services.travis.defaultMaster.name}
          baseUrl: ${services.travis.defaultMaster.baseUrl}
          address: ${services.travis.defaultMaster.address}
          githubToken: ${services.travis.defaultMaster.githubToken}


    dockerRegistry:
      enabled: ${providers.dockerRegistry.enabled:false}


    redis:
      connection: ${REDIS_HOST:redis://localhost:6379}

    spectator:
      applicationName: ${spring.application.name}
      webEndpoint:
        enabled: ${services.spectator.webEndpoint.enabled:false}
        prototypeFilter:
          path: ${services.spectator.webEndpoint.prototypeFilter.path:}
      stackdriver:
        enabled: ${services.stackdriver.enabled}
        projectName: ${services.stackdriver.projectName}
        credentialsPath: ${services.stackdriver.credentialsPath}

    stackdriver:
      hints:
        - name: controller.invocations
          labels:
          - master
  kayenta-armory.yml: |
    kayenta:
      aws:
        enabled: ${ARMORYSPINNAKER_S3_ENABLED:false}
        accounts:
          - name: aws-s3-storage
            bucket: ${ARMORYSPINNAKER_CONF_STORE_BUCKET}
            rootFolder: kayenta
            supportedTypes:
              - OBJECT_STORE
              - CONFIGURATION_STORE

      s3:
        enabled: ${ARMORYSPINNAKER_S3_ENABLED:false}

      google:
        enabled: ${ARMORYSPINNAKER_GCS_ENABLED:false}
        accounts:
          - name: cloud-armory
            bucket: ${ARMORYSPINNAKER_CONF_STORE_BUCKET}
            rootFolder: kayenta-prod
            supportedTypes:
              - METRICS_STORE
              - OBJECT_STORE
              - CONFIGURATION_STORE
              
      gcs:
        enabled: ${ARMORYSPINNAKER_GCS_ENABLED:false}
  kayenta.yml: |2

    server:
      port: 8090

    kayenta:
      atlas:
        enabled: false

      google:
        enabled: false

      aws:
        enabled: false

      datadog:
        enabled: false

      prometheus:
        enabled: false

      gcs:
        enabled: false

      s3:
        enabled: false

      stackdriver:
        enabled: false

      memory:
        enabled: false

      configbin:
        enabled: false

    keiko:
      queue:
        redis:
          queueName: kayenta.keiko.queue
          deadLetterQueueName: kayenta.keiko.queue.deadLetters

    redis:
      connection: ${REDIS_HOST:redis://localhost:6379}

    spectator:
      applicationName: ${spring.application.name}
      webEndpoint:
        enabled: true

    swagger:
      enabled: true
      title: Kayenta API
      description:
      contact:
      patterns:
        - /admin.*
        - /canary.*
        - /canaryConfig.*
        - /canaryJudgeResult.*
        - /credentials.*
        - /fetch.*
        - /health
        - /judges.*
        - /metadata.*
        - /metricSetList.*
        - /metricSetPairList.*
        - /pipeline.*

    security.basic.enabled: false
    management.security.enabled: false
  nginx.conf: |
    user  nginx;
    worker_processes  1;

    error_log  /var/log/nginx/error.log warn;
    pid        /var/run/nginx.pid;

    events {
        worker_connections  1024;
    }

    http {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';
        access_log  /var/log/nginx/access.log  main;
        
        sendfile        on;
        keepalive_timeout  65;
        include /etc/nginx/conf.d/*.conf;
    }

    stream {
        upstream gate_api {
            server armory-gate:8085;
        }

        server {
            listen 8085;
            proxy_pass gate_api;
        }
    }
  nginx.http.conf: |
    gzip on;
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/vnd.ms-fontobject application/x-font-ttf font/opentype image/svg+xml image/x-icon;

    server {
           listen 80;
           listen [::]:80;

           location / {
                proxy_pass http://armory-deck/;
           }

           location /api/ {
                proxy_pass http://armory-gate:8084/;
           }

           location /slack/ {
               proxy_pass http://armory-platform:10000/;
           }

           rewrite ^/login(.*)$ /api/login$1 last;
           rewrite ^/auth(.*)$ /api/auth$1 last;
    }
  nginx.https.conf: |
    gzip on;
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/vnd.ms-fontobject application/x-font-ttf font/opentype image/svg+xml image/x-icon;

    server {
        listen 80;
        listen [::]:80;
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl;
        listen [::]:443 ssl;
        ssl on;
        ssl_certificate /opt/nginx/certs/ssl.crt;
        ssl_certificate_key /opt/nginx/certs/ssl.key;

        location / {
            proxy_pass http://armory-deck/;
        }

        location /api/ {
            proxy_pass http://armory-gate:8084/;
            proxy_set_header Host            $host;
            proxy_set_header X-Real-IP       $proxy_protocol_addr;
            proxy_set_header X-Forwarded-For $proxy_protocol_addr;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /slack/ {
            proxy_pass http://armory-platform:10000/;
        }
        rewrite ^/login(.*)$ /api/login$1 last;
        rewrite ^/auth(.*)$ /api/auth$1 last;
    }
  orca-armory.yml: |
    mine:
      baseUrl: http://${services.barometer.host}:${services.barometer.port}

    pipelineTemplate:
      enabled: ${features.pipelineTemplates.enabled:false}
      jinja:
        enabled: true

    kayenta:
      enabled: ${services.kayenta.enabled:false}
      baseUrl: ${services.kayenta.baseUrl}

    jira:
      enabled: ${features.jira.enabled:false}
      basicAuth:  "Basic ${features.jira.basicAuthToken}"
      url: ${features.jira.createIssueUrl}

    webhook:
      preconfigured:
        - label: Enforce Pipeline Policy
          description: Checks pipeline configuration against policy requirements
          type: enforcePipelinePolicy
          enabled: ${features.certifiedPipelines.enabled:false}
          url: "http://lighthouse:5000/v1/pipelines/${execution.application}/${execution.pipelineConfigId}?check_policy=yes"
          headers:
            Accept:
              - application/json
          method: GET
          waitForCompletion: true
          statusUrlResolution: getMethod
          statusJsonPath: $.status
          successStatuses: pass
          canceledStatuses:
          terminalStatuses: TERMINAL

        - label: "Jira: Create Issue"
          description:  Enter a Jira ticket when this pipeline runs
          type: createJiraIssue
          enabled: ${jira.enabled}
          url:  ${jira.url}
          customHeaders:
            "Content-Type": application/json
            Authorization: ${jira.basicAuth}
          method: POST
          parameters:
            - name: summary
              label: Issue Summary
              description: A short summary of your issue.
            - name: description
              label: Issue Description
              description: A longer description of your issue.
            - name: projectKey
              label: Project key
              description: The key of your JIRA project.
            - name: type
              label: Issue Type
              description: The type of your issue, e.g. "Task", "Story", etc.
          payload: |
            {
              "fields" : {
                "description": "${parameterValues['description']}",
                "issuetype": {
                   "name": "${parameterValues['type']}"
                },
                "project": {
                   "key": "${parameterValues['projectKey']}"
                },
                "summary":  "${parameterValues['summary']}"
              }
            }
          waitForCompletion: false

        - label: "Jira: Update Issue"
          description:  Update a previously created Jira Issue
          type: updateJiraIssue
          enabled: ${jira.enabled}
          url: "${execution.stages.?[type == 'createJiraIssue'][0]['context']['buildInfo']['self']}"
          customHeaders:
            "Content-Type": application/json
            Authorization: ${jira.basicAuth}
          method: PUT
          parameters:
            - name: summary
              label: Issue Summary
              description: A short summary of your issue.
            - name: description
              label: Issue Description
              description: A longer description of your issue.
          payload: |
            {
              "fields" : {
                "description": "${parameterValues['description']}",
                "summary": "${parameterValues['summary']}"
              }
            }
          waitForCompletion: false

        - label: "Jira: Transition Issue"
          description:  Change state of existing Jira Issue
          type: transitionJiraIssue
          enabled: ${jira.enabled}
          url: "${execution.stages.?[type == 'createJiraIssue'][0]['context']['buildInfo']['self']}/transitions"
          customHeaders:
            "Content-Type": application/json
            Authorization: ${jira.basicAuth}
          method: POST
          parameters:
            - name: newStateID
              label: New State ID
              description: The ID of the state you want to transition the issue to.
          payload: |
            {
              "transition" : {
                "id" : "${parameterValues['newStateID']}"
              }
            }
          waitForCompletion: false
        - label: "Jira: Add Comment"
          description:  Add a comment to an existing Jira Issue
          type: commentJiraIssue
          enabled: ${jira.enabled}
          url: "${execution.stages.?[type == 'createJiraIssue'][0]['context']['buildInfo']['self']}/comment"
          customHeaders:
            "Content-Type": application/json
            Authorization: ${jira.basicAuth}
          method: POST
          parameters:
            - name: body
              label: Comment body
              description: The text body of the component.
          payload: |
            {
              "body" : "${parameterValues['body']}"
            }
          waitForCompletion: false

  orca.yml: |
    server:
        port: ${services.orca.port:8083}
        address: ${services.orca.host:localhost}
    oort:
        baseUrl: ${services.oort.baseUrl:localhost:7002}
    front50:
        baseUrl: ${services.front50.baseUrl:localhost:8080}
    mort:
        baseUrl: ${services.mort.baseUrl:localhost:7002}
    kato:
        baseUrl: ${services.kato.baseUrl:localhost:7002}
    bakery:
        baseUrl: ${services.bakery.baseUrl:localhost:8087}
        extractBuildDetails: ${services.bakery.extractBuildDetails:true}
        allowMissingPackageInstallation: ${services.bakery.allowMissingPackageInstallation:true}
    echo:
        enabled: ${services.echo.enabled:false}
        baseUrl: ${services.echo.baseUrl:8089}
    igor:
        baseUrl: ${services.igor.baseUrl:8088}
    flex:
      baseUrl: http://not-a-host
    default:
      bake:
        account: ${providers.aws.primaryCredentials.name}
      securityGroups:
      vpc:
        securityGroups:
    redis:
      connection: ${REDIS_HOST:redis://localhost:6379}
    tasks:
      executionWindow:
        timezone: ${services.orca.timezone}
    spectator:
      applicationName: ${spring.application.name}
      webEndpoint:
        enabled: ${services.spectator.webEndpoint.enabled:false}
        prototypeFilter:
          path: ${services.spectator.webEndpoint.prototypeFilter.path:}        
      stackdriver:
        enabled: ${services.stackdriver.enabled}
        projectName: ${services.stackdriver.projectName}
        credentialsPath: ${services.stackdriver.credentialsPath}
    stackdriver:
      hints:
        - name: controller.invocations
          labels:
          - application
  rosco-armory.yml: |
    redis:
      timeout: 50000

    rosco:
      jobs:
        local:
          timeoutMinutes: 60
  rosco.yml: |
    server:
      port: ${services.rosco.port:8087}
      address: ${services.rosco.host:localhost}

    redis:
      connection: ${REDIS_HOST:redis://localhost:6379}

    aws:
      enabled: ${providers.aws.enabled:false}

    docker:
      enabled: ${services.docker.enabled:false}
      bakeryDefaults:
        targetRepository: ${services.docker.targetRepository}

    google:
      enabled: ${providers.google.enabled:false}
      accounts:
        - name: ${providers.google.primaryCredentials.name}
          project: ${providers.google.primaryCredentials.project}
          jsonPath: ${providers.google.primaryCredentials.jsonPath}
      gce:
        bakeryDefaults:
          zone: ${providers.google.defaultZone}

    rosco:
      configDir: ${services.rosco.configDir}
      jobs:
        local:
          timeoutMinutes: 30

    spectator:
      applicationName: ${spring.application.name}
      webEndpoint:
        enabled: ${services.spectator.webEndpoint.enabled:false}
        prototypeFilter:
          path: ${services.spectator.webEndpoint.prototypeFilter.path:}
      stackdriver:
        enabled: ${services.stackdriver.enabled}
        projectName: ${services.stackdriver.projectName}
        credentialsPath: ${services.stackdriver.credentialsPath}

    stackdriver:
      hints:
        - name: bakes
          labels:
          - success
  spinnaker-armory.yml: |
    armory:
      architecture: 'k8s'
      
    features:
      artifacts:
        enabled: true
      pipelineTemplates:
        enabled: ${PIPELINE_TEMPLATES_ENABLED:false}
      infrastructureStages:
        enabled: ${INFRA_ENABLED:false}
      certifiedPipelines:
        enabled: ${CERTIFIED_PIPELINES_ENABLED:false}
      configuratorEnabled:
        enabled: true
      configuratorWizard:
        enabled: true
      configuratorCerts:
        enabled: true
      loadtestStage:
        enabled: ${LOADTEST_ENABLED:false}
      jira:
        enabled: ${JIRA_ENABLED:false}
        basicAuthToken: ${JIRA_BASIC_AUTH}
        url: ${JIRA_URL}
        login: ${JIRA_LOGIN}
        password: ${JIRA_PASSWORD}

      slaEnabled:
        enabled: ${SLA_ENABLED:false}
      chaosMonkey:
        enabled: ${CHAOS_ENABLED:false}

      armoryPlatform:
        enabled: ${PLATFORM_ENABLED:false}
        uiEnabled: ${PLATFORM_UI_ENABLED:false}

    services:
      default:
        host: ${DEFAULT_DNS_NAME:localhost}

      clouddriver:
        host: ${DEFAULT_DNS_NAME:armory-clouddriver}
        entityTags:
          enabled: false

      configurator:
        baseUrl: http://${CONFIGURATOR_HOST:armory-configurator}:8069

      echo:
        host: ${DEFAULT_DNS_NAME:armory-echo}

      deck:
        gateUrl: ${API_HOST:service.default.host}
        baseUrl: ${DECK_HOST:armory-deck}

      dinghy:
        enabled: ${DINGHY_ENABLED:false}
        host: ${DEFAULT_DNS_NAME:armory-dinghy}
        baseUrl: ${services.default.protocol}://${services.dinghy.host}:${services.dinghy.port}
        port: 8081

      front50:
        host: ${DEFAULT_DNS_NAME:armory-front50}
        cassandra:
          enabled: false
        redis:
          enabled: true
        gcs:
          enabled: ${ARMORYSPINNAKER_GCS_ENABLED:false}
        s3:
          enabled: ${ARMORYSPINNAKER_S3_ENABLED:false}
        storage_bucket: ${ARMORYSPINNAKER_CONF_STORE_BUCKET}
        rootFolder: ${ARMORYSPINNAKER_CONF_STORE_PREFIX:front50}

      gate:
        host: ${DEFAULT_DNS_NAME:armory-gate}

      igor:
        host: ${DEFAULT_DNS_NAME:armory-igor}


      kayenta:
        enabled: true
        host: ${DEFAULT_DNS_NAME:armory-kayenta}
        canaryConfigStore: true
        port: 8090
        baseUrl: ${services.default.protocol}://${services.kayenta.host}:${services.kayenta.port}
        metricsStore: ${METRICS_STORE:stackdriver}
        metricsAccountName: ${METRICS_ACCOUNT_NAME}
        storageAccountName: ${STORAGE_ACCOUNT_NAME}
        atlasWebComponentsUrl: ${ATLAS_COMPONENTS_URL:}
        
      lighthouse:
        host: ${DEFAULT_DNS_NAME:armory-lighthouse}
        port: 5000
        baseUrl: ${services.default.protocol}://${services.lighthouse.host}:${services.lighthouse.port}

      orca:
        host: ${DEFAULT_DNS_NAME:armory-orca}

      platform:
        enabled: ${PLATFORM_ENABLED:false}
        host: ${DEFAULT_DNS_NAME:armory-platform}
        baseUrl: ${services.default.protocol}://${services.platform.host}:${services.platform.port}
        port: 5001

      rosco:
        host: ${DEFAULT_DNS_NAME:armory-rosco}
        enabled: true
        configDir: /opt/spinnaker/config/packer

      bakery:
        allowMissingPackageInstallation: true

      barometer:
        enabled: ${BAROMETER_ENABLED:false}
        host: ${DEFAULT_DNS_NAME:armory-barometer}
        baseUrl: ${services.default.protocol}://${services.barometer.host}:${services.barometer.port}
        port: 9092
        newRelicEnabled: ${NEW_RELIC_ENABLED:false}

      redis:
        host: redis
        port: 6379
        connection: ${REDIS_HOST:redis://localhost:6379}

      fiat:
        enabled: ${FIAT_ENABLED:false}
        host: ${DEFAULT_DNS_NAME:armory-fiat}
        port: 7003
        baseUrl: ${services.default.protocol}://${services.fiat.host}:${services.fiat.port}

    providers:
      aws:
        enabled: ${SPINNAKER_AWS_ENABLED:true}
        defaultRegion: ${SPINNAKER_AWS_DEFAULT_REGION:us-west-2}
        defaultIAMRole: ${SPINNAKER_AWS_DEFAULT_IAM_ROLE:SpinnakerInstanceProfile}
        defaultAssumeRole: ${SPINNAKER_AWS_DEFAULT_ASSUME_ROLE:SpinnakerManagedProfile}
        primaryCredentials:
          name: ${SPINNAKER_AWS_DEFAULT_ACCOUNT:default-aws-account}

      kubernetes:
        proxy: localhost:8001
        apiPrefix: api/v1/proxy/namespaces/kube-system/services/kubernetes-dashboard/#
  spinnaker.yml: |2
    global:
      spinnaker:
        timezone: 'America/Los_Angeles'
        architecture: ${PLATFORM_ARCHITECTURE}

    services:
      default:
        host: localhost
        protocol: http
      clouddriver:
        host: ${services.default.host}
        port: 7002
        baseUrl: ${services.default.protocol}://${services.clouddriver.host}:${services.clouddriver.port}
        aws:
          udf:
            enabled: true

      echo:
        enabled: true
        host: ${services.default.host}
        port: 8089
        baseUrl: ${services.default.protocol}://${services.echo.host}:${services.echo.port}
        cassandra:
          enabled: false
        inMemory:
          enabled: true

        cron:
          enabled: true
          timezone: ${global.spinnaker.timezone}

        notifications:
          mail:
            enabled: false
            host: # the smtp host
            fromAddress: # the address for which emails are sent from
          hipchat:
            enabled: false
            url: # the hipchat server to connect to
            token: # the hipchat auth token
            botName: # the username of the bot
          sms:
            enabled: false
            account: # twilio account id
            token: # twilio auth token
            from: # phone number by which sms messages are sent
          slack:
            enabled: false
            token: # the API token for the bot
            botName: # the username of the bot

      deck:
        host: ${services.default.host}
        port: 9000
        baseUrl: ${services.default.protocol}://${services.deck.host}:${services.deck.port}
        gateUrl: ${API_HOST:services.gate.baseUrl}
        bakeryUrl: ${services.bakery.baseUrl}
        timezone: ${global.spinnaker.timezone}
        auth:
          enabled: ${AUTH_ENABLED:false}


      fiat:
        enabled: false
        host: ${services.default.host}
        port: 7003
        baseUrl: ${services.default.protocol}://${services.fiat.host}:${services.fiat.port}

      front50:
        host: ${services.default.host}
        port: 8080
        baseUrl: ${services.default.protocol}://${services.front50.host}:${services.front50.port}
        storage_bucket: ${SPINNAKER_DEFAULT_STORAGE_BUCKET:}
        bucket_location:
        bucket_root: front50
        cassandra:
          enabled: false
        redis:
          enabled: false
        gcs:
          enabled: false
        s3:
          enabled: false

      gate:
        host: ${services.default.host}
        port: 8084
        baseUrl: ${services.default.protocol}://${services.gate.host}:${services.gate.port}

      igor:
        enabled: false
        host: ${services.default.host}
        port: 8088
        baseUrl: ${services.default.protocol}://${services.igor.host}:${services.igor.port}

      kato:
        host: ${services.clouddriver.host}
        port: ${services.clouddriver.port}
        baseUrl: ${services.clouddriver.baseUrl}

      mort:
        host: ${services.clouddriver.host}
        port: ${services.clouddriver.port}
        baseUrl: ${services.clouddriver.baseUrl}

      orca:
        host: ${services.default.host}
        port: 8083
        baseUrl: ${services.default.protocol}://${services.orca.host}:${services.orca.port}
        timezone: ${global.spinnaker.timezone}
        enabled: true

      oort:
        host: ${services.clouddriver.host}
        port: ${services.clouddriver.port}
        baseUrl: ${services.clouddriver.baseUrl}

      rosco:
        host: ${services.default.host}
        port: 8087
        baseUrl: ${services.default.protocol}://${services.rosco.host}:${services.rosco.port}
        configDir: /opt/rosco/config/packer

      bakery:
        host: ${services.rosco.host}
        port: ${services.rosco.port}
        baseUrl: ${services.rosco.baseUrl}
        extractBuildDetails: true
        allowMissingPackageInstallation: false

      docker:
        targetRepository: # Optional, but expected in spinnaker-local.yml if specified.

      jenkins:
        enabled: ${services.igor.enabled:false}
        defaultMaster:
          name: Jenkins
          baseUrl:   # Expected in spinnaker-local.yml
          username:  # Expected in spinnaker-local.yml
          password:  # Expected in spinnaker-local.yml

      redis:
        host: redis
        port: 6379
        connection: ${REDIS_HOST:redis://localhost:6379}

      cassandra:
        host: ${services.default.host}
        port: 9042
        embedded: false
        cluster: CASS_SPINNAKER

      travis:
        enabled: false
        defaultMaster:
          name: ci # The display name for this server. Gets prefixed with "travis-"
          baseUrl: https://travis-ci.com
          address: https://api.travis-ci.org
          githubToken: # GitHub scopes currently required by Travis is required.

      spectator:
        webEndpoint:
          enabled: false

      stackdriver:
        enabled: ${SPINNAKER_STACKDRIVER_ENABLED:false}
        projectName: ${SPINNAKER_STACKDRIVER_PROJECT_NAME:${providers.google.primaryCredentials.project}}
        credentialsPath: ${SPINNAKER_STACKDRIVER_CREDENTIALS_PATH:${providers.google.primaryCredentials.jsonPath}}


    providers:
      aws:
        enabled: ${SPINNAKER_AWS_ENABLED:false}
        simpleDBEnabled: false
        defaultRegion: ${SPINNAKER_AWS_DEFAULT_REGION:us-west-2}
        defaultIAMRole: BaseIAMRole
        defaultSimpleDBDomain: CLOUD_APPLICATIONS
        primaryCredentials:
          name: default
        defaultKeyPairTemplate: "{{name}}-keypair"


      google:
        enabled: ${SPINNAKER_GOOGLE_ENABLED:false}
        defaultRegion: ${SPINNAKER_GOOGLE_DEFAULT_REGION:us-central1}
        defaultZone: ${SPINNAKER_GOOGLE_DEFAULT_ZONE:us-central1-f}


        primaryCredentials:
          name: my-account-name
          project: ${SPINNAKER_GOOGLE_PROJECT_ID:}
          jsonPath: ${SPINNAKER_GOOGLE_PROJECT_CREDENTIALS_PATH:}
          consul:
            enabled: ${SPINNAKER_GOOGLE_CONSUL_ENABLED:false}


      cf:
        enabled: false
        defaultOrg: spinnaker-cf-org
        defaultSpace: spinnaker-cf-space
        primaryCredentials:
          name: my-cf-account
          api: my-cf-api-uri
          console: my-cf-console-base-url

      azure:
        enabled: ${SPINNAKER_AZURE_ENABLED:false}
        defaultRegion: ${SPINNAKER_AZURE_DEFAULT_REGION:westus}
        primaryCredentials:
          name: my-azure-account

          clientId:
          appKey:
          tenantId:
          subscriptionId:

      titan:
        enabled: false
        defaultRegion: us-east-1
        primaryCredentials:
          name: my-titan-account

      kubernetes:

        enabled: ${SPINNAKER_KUBERNETES_ENABLED:false}
        primaryCredentials:
          name: my-kubernetes-account
          namespace: default
          dockerRegistryAccount: ${providers.dockerRegistry.primaryCredentials.name}

      dockerRegistry:
        enabled: ${SPINNAKER_KUBERNETES_ENABLED:false}

        primaryCredentials:
          name: my-docker-registry-account
          address: ${SPINNAKER_DOCKER_REGISTRY:https://index.docker.io/ }
          repository: ${SPINNAKER_DOCKER_REPOSITORY:}
          username: ${SPINNAKER_DOCKER_USERNAME:}
          passwordFile: ${SPINNAKER_DOCKER_PASSWORD_FILE:}
          
      openstack:
        enabled: false
        defaultRegion: ${SPINNAKER_OPENSTACK_DEFAULT_REGION:RegionOne}
        primaryCredentials:
          name: my-openstack-account
          authUrl: ${OS_AUTH_URL}
          username: ${OS_USERNAME}
          password: ${OS_PASSWORD}
          projectName: ${OS_PROJECT_NAME}
          domainName: ${OS_USER_DOMAIN_NAME:Default}
          regions: ${OS_REGION_NAME:RegionOne}
          insecure: false
```

dp.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: armory-clouddriver
  name: armory-clouddriver
  namespace: armory
spec:
  replicas: 1
  revisionHistoryLimit: 7
  selector:
    matchLabels:
      app: armory-clouddriver
  template:
    metadata:
      annotations:
        artifact.spinnaker.io/location: '"armory"'
        artifact.spinnaker.io/name: '"armory-clouddriver"'
        artifact.spinnaker.io/type: '"kubernetes/deployment"'
        moniker.spinnaker.io/application: '"armory"'
        moniker.spinnaker.io/cluster: '"clouddriver"'
      labels:
        app: armory-clouddriver
    spec:
      containers:
      - name: armory-clouddriver
        image: harbor.od.com/armory/clouddriver:v1.8.x
        imagePullPolicy: IfNotPresent
        command:
        - bash
        - -c
        args:
        - bash /opt/spinnaker/config/default/fetch.sh && cd /home/spinnaker/config
          && /opt/clouddriver/bin/clouddriver
        ports:
        - containerPort: 7002
          protocol: TCP
        env:
        - name: JAVA_OPTS
          value: -Xmx2048M
        envFrom:
        - configMapRef:
            name: init-env
        livenessProbe:
          failureThreshold: 5
          httpGet:
            path: /health
            port: 7002
            scheme: HTTP
          initialDelaySeconds: 600
          periodSeconds: 3
          successThreshold: 1
          timeoutSeconds: 1
        readinessProbe:
          failureThreshold: 5
          httpGet:
            path: /health
            port: 7002
            scheme: HTTP
          initialDelaySeconds: 180
          periodSeconds: 3
          successThreshold: 5
          timeoutSeconds: 1
        securityContext:
          runAsUser: 0
        volumeMounts:
        - mountPath: /etc/podinfo
          name: podinfo
        - mountPath: /home/spinnaker/.aws
          name: credentials
        - mountPath: /opt/spinnaker/credentials/custom
          name: default-kubeconfig
        - mountPath: /opt/spinnaker/config/default
          name: default-config
        - mountPath: /opt/spinnaker/config/custom
          name: custom-config
      imagePullSecrets:
      - name: harbor
      volumes: 
      - configMap:
          defaultMode: 420
          name: default-kubeconfig
        name: default-kubeconfig
      - configMap:
          defaultMode: 420
          name: custom-config
        name: custom-config
      - configMap:
          defaultMode: 420
          name: default-config
        name: default-config
      - name: credentials
        secret:
          defaultMode: 420
          secretName: credentials
      - downwardAPI:
          defaultMode: 420
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.labels
            path: labels
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.annotations
            path: annotations
        name: podinfo
```

svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: armory-clouddriver
  namespace: armory
spec:
  ports:
  - port: 7002
    protocol: TCP
    targetPort: 7002
  selector:
    app: armory-clouddriver
```

