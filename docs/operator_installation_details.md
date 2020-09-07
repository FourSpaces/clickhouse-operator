# Install ClickHouse Operator

验证 [clickhouse-operator-install.yaml][clickhouse-operator-install.yaml] 文件的可用性.
In is located in `deploy/operator` folder inside `clickhouse-operator` sources.
位于clickhouse-operator源中的deploy/operator文件夹中。

## Install
Operator installation process is quite straightforward and consists of one main step - deploy **ClickHouse operator**.
We'll apply operator manifest directly from github repo



操作符的安装过程非常简单，只包含一个主要步骤——部署ClickHouse操作符。
我们将直接从github回购应用操作符清单

```bash
kubectl apply -f https://raw.githubusercontent.com/Altinity/clickhouse-operator/master/deploy/operator/clickhouse-operator-install.yaml
```

The following results are expected:

预期结果如下:

```text
customresourcedefinition.apiextensions.k8s.io/clickhouseinstallations.clickhouse.altinity.com created
serviceaccount/clickhouse-operator created
clusterrolebinding.rbac.authorization.k8s.io/clickhouse-operator created
deployment.apps/clickhouse-operator configured
```

## Verify operator is up and running
Operator is deployed in **kube-system** namespace 
```bash
kubectl get pods --namespace kube-system
```

Expected results:
```text
NAME                                   READY   STATUS    RESTARTS   AGE
...
clickhouse-operator-5c46dfc7bd-7cz5l   1/1     Running   0          43m
...
```


## 资源描述

1. 让我们浏览一下使用ClickHouse操作符创建的所有资源，它们是:

   1. 自定义资源定义
   2. 服务帐户
   3. 集群角色绑定
   4. 部署


### 自定义资源定义
```text
customresourcedefinition.apiextensions.k8s.io/clickhouseinstallations.clickhouse.altinity.com created
```
New [Custom Resource Definition][customresourcedefinitions] named **ClickHouseInstallation** is created.
k8s API is extended with new kind `ClickHouseInstallation` and we'll be able to manage k8s resource of `kind: ClickHouseInstallation`

将创建名为ClickHouseInstallation的新自定义资源定义。
k8s API扩展了新的类型ClickHouseInstallation，我们将能够管理k8s的类型资源:ClickHouseInstallation

### 服务帐户
```text
serviceaccount/clickhouse-operator created
```
New [Service Account][configure-service-account] named **clickhouse-operator** is created.
A service account provides an identity used to contact the `apiserver` by the processes that run in a Pod. 
Processes in containers inside pods can contact the `apiserver`, and when they do, they are authenticated as a particular `Service Account` - `clickhouse-operator` in this case.

将创建名为clickhouse-operator的新服务帐户。
服务帐户提供一个标识，用于通过在Pod中运行的进程联系服务器。
pods中的容器中的进程可以与apiserver联系，当它们这样做时，它们将被身份验证为特定的服务帐户(在本例中为clickhouse操作符)。

### 集群角色绑定
```text
clusterrolebinding.rbac.authorization.k8s.io/clickhouse-operator created
```
New [CluserRoleBinding][rolebinding-and-clusterrolebinding] named **clickhouse-operator** is created.
A role binding grants the permissions defined in a role to a set of users. 
It holds a reference to the role being granted to the list of subjects (users, groups, or service accounts).
In this case Role

将创建名为clickhouse-operator的新CluserRoleBinding。
角色绑定将角色中定义的权限授予一组用户。
它持有一个角色引用，该角色被授予主题列表(用户、组或服务帐户)。
在这种情况下，角色

```yaml
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
```
被授予
```yaml
subjects:
  - kind: ServiceAccount
    name: clickhouse-operator
    namespace: kube-system
```
`clickhouse-operator` Service Account created earlier.
Permissions are granted cluster-wide with a `ClusterRoleBinding`.

之前创建的clickhouse-operator服务帐户。
权限是通过ClusterRoleBinding在集群范围内授予的。

### Deployment
```text
deployment.apps/clickhouse-operator configured
```
New [Deployment][deployment] named **clickhouse-operator** is created. 
ClickHouse operator app would be run by this deployment in `kube-system` namespace.

将创建名为clickhouse-operator的新部署。
通过这个部署，ClickHouse操作程序将在kube-system名称空间中运行。

## 验证资源

检查自定义资源定义
```bash
kubectl get customresourcedefinitions
```
Expected result/预期的结果
```text
NAME                                              CREATED AT
...
clickhouseinstallations.clickhouse.altinity.com   2019-01-25T10:17:57Z
...
```

Check Service Account
```bash
kubectl get serviceaccounts -n kube-system
```
Expected result
```text
NAME                                 SECRETS   AGE
...
clickhouse-operator                  1         27h
...
```

Check Cluster Role Binding
```bash
kubectl get clusterrolebinding
```
Expected result
```text
NAME                                                   AGE
...
clickhouse-operator                                    31m
...

```
Check deployment
```bash
kubectl get deployments --namespace kube-system
kubectl get deployments --namespace  test-clickhouse-operator
```
Expected result
```text
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
...
clickhouse-operator    1/1     1            1           31m
...

```

[clickhouse-operator-install.yaml]: ../deploy/operator/clickhouse-operator-install.yaml
[customresourcedefinitions]: https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions
[configure-service-account]: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
[rolebinding-and-clusterrolebinding]: https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding
[deployment]: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
