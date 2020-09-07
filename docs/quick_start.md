# 快速入门指南

# 目录
* [ClickHouse Operator Installation](#clickhouse-operator-installation)
* [构建 ClickHouse Operator from 源码](#building-clickhouse-operator-from-sources)
* [Examples/例子](#examples)
  * [琐碎的 Example](#trivial-example)
  * [Connect to ClickHouse Database](#connect-to-clickhouse-database)
  * [简单的持久卷例子](#simple-persistent-volume-example)
  * [使用Pod和VolumeClaim模板进行自定义部署](#custom-deployment-with-pod-and-volumeclaim-templates)
  * [使用特定的ClickHouse配置进行自定义部署](#custom-deployment-with-specific-clickhouse-configuration)

# Prerequisites / 先决条件
1. Operational Kubernetes instance
1. 正确配置的`kubectl`
1. `curl`

# ClickHouse Operator Installation

Apply `clickhouse-operator` installation manifest. The simplest way - directly from `github`.

## **如果您方便，请将 operator  安装到`kube-system`命名空间中**


just run:
```bash
kubectl apply -f https://raw.githubusercontent.com/Altinity/clickhouse-operator/master/deploy/operator/clickhouse-operator-install.yaml
```

## **如果您想自定义安装参数**,

such as namespace where to install operator or operator's image, use the special installer script.
如果要在其他命名空间安装 operator 或者 operator的镜像，需要使用定制的安装脚本
```bash
curl -s https://raw.githubusercontent.com/Altinity/clickhouse-operator/master/deploy/operator-web-installer/clickhouse-operator-install.sh | OPERATOR_NAMESPACE=test-clickhouse-operator bash

cat 
```

Take into account explicitly specified namespace
考虑明确指定的名称空间
```bash
OPERATOR_NAMESPACE=test-clickhouse-operator
```
This namespace would be created and used to install `clickhouse-operator` into.
Install script would download some `.yaml` and `.xml` files and install `clickhouse-operator` into specified namespace.
该名称空间将被创建并用于将clickhouse-operator安装到其中。
安装脚本会下载一些.yaml和.xml文件，并将clickhouse-operator安装到指定的命名空间中。

如果未指定“ OPERATOR_NAMESPACE”，则为：
```bash
cd ~
curl -s https://raw.githubusercontent.com/Altinity/clickhouse-operator/master/deploy/operator-web-installer/clickhouse-operator-install.sh | bash
```
installer will create namespace `clickhouse-operator` and install **clickhouse-operator** into it.
安装程序将创建命名空间“ clickhouse-operator”，并将** clickhouse-operator **安装到其中。

## **如果您无法在受保护的环境中从Internet运行脚本**, 

you can download manually [this template file][clickhouse-operator-install-template.yaml]
and edit it according to your choice. After that apply it with `kubectl`. Or you can use this snippet instead:
你可以下载 [这个模版][clickhouse-operator-install-template.yaml]并根据您的选择进行编辑。 之后，将其与`kubectl`一起应用。 或者，您可以改用以下代码段：

```bash
# Namespace to install operator into
OPERATOR_NAMESPACE="${OPERATOR_NAMESPACE:-clickhouse-operator}"
# Namespace to install metrics-exporter into
METRICS_EXPORTER_NAMESPACE="${OPERATOR_NAMESPACE}"

# Operator's docker image
OPERATOR_IMAGE="${OPERATOR_IMAGE:-altinity/clickhouse-operator:latest}"
# Metrics exporter's docker image
METRICS_EXPORTER_IMAGE="${METRICS_EXPORTER_IMAGE:-altinity/metrics-exporter:latest}"

# 建立 clickhouse-operator into 指定 namespace
kubectl apply --namespace="${OPERATOR_NAMESPACE}" -f <( \
    curl -s https://raw.githubusercontent.com/Altinity/clickhouse-operator/master/deploy/operator/clickhouse-operator-install-template.yaml | \
        OPERATOR_IMAGE="${OPERATOR_IMAGE}" \
        OPERATOR_NAMESPACE="${OPERATOR_NAMESPACE}" \
        METRICS_EXPORTER_IMAGE="${METRICS_EXPORTER_IMAGE}" \
        METRICS_EXPORTER_NAMESPACE="${METRICS_EXPORTER_NAMESPACE}" \
        envsubst \
)
```

## Operator installation 过程
```text
Setup ClickHouse Operator into test-clickhouse-operator namespace
namespace/test-clickhouse-operator created
customresourcedefinition.apiextensions.k8s.io/clickhouseinstallations.clickhouse.altinity.com configured
serviceaccount/clickhouse-operator created
clusterrolebinding.rbac.authorization.k8s.io/clickhouse-operator configured
service/clickhouse-operator-metrics created
configmap/etc-clickhouse-operator-files created
configmap/etc-clickhouse-operator-confd-files created
configmap/etc-clickhouse-operator-configd-files created
configmap/etc-clickhouse-operator-templatesd-files created
configmap/etc-clickhouse-operator-usersd-files created
deployment.apps/clickhouse-operator created
```

检查 `clickhouse-operator` 是否在运行:
```bash
kubectl get pods -n test-clickhouse-operator
```
```text
NAME                                 READY   STATUS    RESTARTS   AGE
clickhouse-operator-5ddc6d858f-drppt 1/1     Running   0          1m
```

## 从源头构建ClickHouse运营商

Complete instructions on how to build ClickHouse operator from sources as well as how to build a docker image and use it inside `kubernetes` described [here][build_from_sources].
有关如何从源代码构建ClickHouse运算符以及如何构建docker映像并在[此处] [build_from_sources]中所述的`kubernetes中使用它的完整说明

# 例子

有几个现成的[ClickHouseInstallation示例] [chi-examples]。 以下是一些入门。

## 创建自定义命名空间
将所有组件都运行在专用名称空间中是一个好习惯。 让我们在“ test”命名空间中运行示例
```bash
kubectl create namespace test
```
```text
namespace/test created
```

## 简单的例子

这是个简单的 [1 shard 1 replica][01-simple-layout-01-1shard-1repl.yaml] 例子.

**WARNING**: 除了“ Hello，world！”以外，请勿将其用于任何其他用途，因为它没有持久性存储！

```
cd clickhouse-operator-master/docs/chi-examples
```

```bash
kubectl apply -n test-clickhouse-operator -f https://github.com/Altinity/clickhouse-operator/tree/master/docs/chi-examples/01-simple-layout-01-1shard-1repl.yaml

kubectl apply -n test-clickhouse-operator -f 01-simple-layout-01-1shard-1repl.yaml
```
```text
clickhouseinstallation.clickhouse.altinity.com/simple-01 created
```

安装规范很简单，定义了一个副本集群：
```yaml
apiVersion: "clickhouse.altinity.com/v1"
kind: "ClickHouseInstallation"
metadata:
  name: "simple-01"
```

创建集群后，需要进行两项检查

```bash
kubectl get pods -n test-clickhouse-operator
```
```text
NAME                    READY   STATUS    RESTARTS   AGE
chi-b3d29f-a242-0-0-0   1/1     Running   0          10m
```

注意“正在运行”状态。 还检查运营商创建的服务：

```bash
kubectl get service -n test-clickhouse-operator
```
```text
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP                          PORT(S)                         AGE
chi-b3d29f-a242-0-0     ClusterIP      None             <none>                               8123/TCP,9000/TCP,9009/TCP      11m
clickhouse-example-01   LoadBalancer   100.64.167.170   abc-123.us-east-1.elb.amazonaws.com   8123:30954/TCP,9000:32697/TCP   11m
```

ClickHouse已启动并正在运行！

## Connect to ClickHouse Database 

有两种方法可以连接到ClickHouse数据库

```
[root@logt222 ~]# kubectl get service -n test-clickhouse-operator
NAME                          TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                         AGE
chi-simple-01-cluster-0-0     ClusterIP      None            <none>        8123/TCP,9000/TCP,9009/TCP      13h
clickhouse-operator-metrics   ClusterIP      10.43.125.221   <none>        8888/TCP                        17h
clickhouse-simple-01          LoadBalancer   10.43.20.142    <pending>     8123:32339/TCP,9000:30207/TCP   13h
```

1. 如果先前的命令“ kubectl get service -n test”报告为** EXTERNAL-IP **（在我们的案例中为abc-123.us-east-1.elb.amazonaws.com），我们可以使用以下命令直接访问ClickHouse：
```bash
clickhouse-client -h abc-123.us-east-1.elb.amazonaws.com
```
```text
ClickHouse client version 18.14.12.
Connecting to abc-123.us-east-1.elb.amazonaws.com:9000.
Connected to ClickHouse server version 19.4.3 revision 54416.
```
2. 如果没有** EXTERNAL-IP **，我们可以从Kubernetes集群内部访问ClickHouse
```bash
kubectl -n test-clickhouse-operator exec -it chi-b3d29f-a242-0-0-0 -- clickhouse-client

kubectl -n test-clickhouse-operator exec -it chi-simple-01-cluster-0-0-0 -- clickhouse-client
```
```text
ClickHouse client version 19.4.3.11.
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 19.4.3 revision 54416.
```

## 简单的持久卷例子

如果有可用的动态卷配置-ex.: 
在AWS上运行-我们能够使用PersistentVolumeClaims
清单是 [available in examples][03-persistent-volume-01-default-volume.yaml]

```yaml
apiVersion: "clickhouse.altinity.com/v1"
kind: "ClickHouseInstallation"
metadata:
  name: "pv-simple"
spec:
  defaults:
    templates:
      dataVolumeClaimTemplate: volume-template
      logVolumeClaimTemplate: volume-template
  configuration:
    clusters:
      - name: "simple"
        layout:
          shardsCount: 1
          replicasCount: 1
      - name: "replicas"
        layout:
          shardsCount: 1
          replicasCount: 2
      - name: "shards"
        layout:
          shardsCount: 2
  templates:
    volumeClaimTemplates:
      - name: volume-template
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 123Mi
```



```
kubectl apply -n test-clickhouse-operator -f 03-persistent-volume-02-pod-template.yaml
```





## 使用Pod和VolumeClaim模板进行自定义部署

让我们安装更复杂的示例:
1. 指定的部署
1. Pod template
1. VolumeClaim template（卷目录模版）

Manifest is [available in examples][03-persistent-volume-02-pod-template.yaml]

```yaml
apiVersion: "clickhouse.altinity.com/v1"
kind: "ClickHouseInstallation"
metadata:
  name: "pv-log"
spec:
  configuration:
    clusters:
      - name: "deployment-pv"
        # Templates are specified for this cluster explicitly
        # 为此群集显式指定模板
        templates:
          podTemplate: pod-template-with-volumes
        layout:
          shardsCount: 1
          replicasCount: 1

  templates:
    podTemplates:
      - name: pod-template-with-volumes
        spec:
          containers:
            - name: clickhouse
              image: yandex/clickhouse-server:19.3.7
              ports:
                - name: http
                  containerPort: 8123
                - name: client
                  containerPort: 9000
                - name: interserver
                  containerPort: 9009
              volumeMounts:
                - name: data-storage-vc-template
                  mountPath: /var/lib/clickhouse
                - name: log-storage-vc-template
                  mountPath: /var/log/clickhouse-server

    volumeClaimTemplates:
      - name: data-storage-vc-template
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 3Gi
      - name: log-storage-vc-template
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 2Gi
```

## 使用特定的ClickHouse配置进行自定义部署

您可以告诉 operator 配置ClickHouse，如下例所示 ([link to the manifest][05-settings-01-overview.yaml]):

```yaml
apiVersion: "clickhouse.altinity.com/v1"
kind: "ClickHouseInstallation"
metadata:
  name: "settings-01"
spec:
  configuration:
    users:
      # test user has 'password' specified, while admin user has 'password_sha256_hex' specified
      test/password: qwerty
      test/networks/ip:
        - "127.0.0.1/32"
        - "192.168.74.1/24"
      test/profile: test_profile
      test/quota: test_quota
      test/allow_databases/database:
        - "dbname1"
        - "dbname2"
        - "dbname3"
      # admin use has 'password_sha256_hex' so actual password value is not published
      admin/password_sha256_hex: 8bd66e4932b4968ec111da24d7e42d399a05cb90bf96f587c3fa191c56c401f8
      admin/networks/ip: "127.0.0.1/32"
      admin/profile: default
      admin/quota: default
      # readonly user has 'password' field specified, not 'password_sha256_hex' as admin user above
      readonly/password: readonly_password
      readonly/profile: readonly
      readonly/quota: default
    profiles:
      test_profile/max_memory_usage: "1000000000"
      test_profile/readonly: "1"
      readonly/readonly: "1"
    quotas:
      test_quota/interval/duration: "3600"
    settings:
      compression/case/method: zstd
      disable_internal_dns_cache: 1
    files:
      dict1.xml: |
        <yandex>
            <!-- ref to file /etc/clickhouse-data/config.d/source1.csv -->
        </yandex>
      source1.csv: |
        a1,b1,c1,d1
        a2,b2,c2,d2
    clusters:
      - name: "standard"
        layout:
          shardsCount: 1
          replicasCount: 1
```

[build_from_sources]: ./operator_build_from_sources.md
[clickhouse-operator-install-template.yaml]: ../deploy/operator/clickhouse-operator-install-template.yaml
[chi-examples]: ./chi-examples/
[01-simple-layout-01-1shard-1repl.yaml]: ./chi-examples/01-simple-layout-01-1shard-1repl.yaml
[03-persistent-volume-01-default-volume.yaml]: ./chi-examples/03-persistent-volume-01-default-volume.yaml
[03-persistent-volume-02-pod-template.yaml]: ./chi-examples/03-persistent-volume-02-pod-template.yaml
[05-settings-01-overview.yaml]: ./chi-examples/05-settings-01-overview.yaml
