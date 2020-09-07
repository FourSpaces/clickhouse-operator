# Setting up Zookeeper

本文档描述了如何在k8s环境中设置ZooKeeper.

Zookeeper安装有两种选择:
1. [Quick start](#quick-start) - 只要快速运行，不要问任何问题
1. [Advanced setup](#advanced-setup) - 设置内部细节，如存储类、副本编号等

在ZooKeeper安装过程中，会创建/配置以下项目:
1. [OPTIONAL] 创建单独的命名空间来运行Zookeeper
1. 创建k8s资源(可选，在名称空间内):
  * [Service][k8sdoc_service_main] - 用于为动物园管理员提供中央访问点
  * [Headless Service][k8sdoc_service_headless] - 用于提供DNS名称
  * [Disruption Balance][k8sdoc_disruption_balance] - 用于指定脱机pods的最大数量
  * [OPTIONAL] [Storage Class][k8sdoc_storage_class] - 用于指定Zookeeper用于数据存储的存储类
  * [Stateful Set][k8sdoc_statefulset] - 用于管理和扩展pods的集合

## 快速启动
快速启动有两种方式:
1. With persistent volume - good for AWS. File are located in [deploy/zookeeper/quick-start-persistent-volume][quickstart_persistent] 

   ```
   具有持久卷—适用于AWS。文件位于部署/zookeeper/快速启动持久卷中
   ```

   

1. With local [`emptyDir`][k8sdoc_emptydir] storage - good for standalone local run, however has to true persistence. \
Files are located in [deploy/zookeeper/quick-start-volume-emptyDir][quickstart_emptydir] 

       使用本地emptyDir存储——适合独立本地运行，但是必须具有真正的持久性。\

文件位于deploy/zookeeper/快速启动卷- emptydir中



每种快速启动风格都提供了以下安装选项:
1. 1-node Zookeeper cluster (**zookeeper-1-** files). 没有提供故障转移.
1. 3-node Zookeeper cluster (**zookeeper-3-** files). 提供故障转移.

In case you'd like to test with AWS or any other cloud provider, we recommend to go with [deploy/zookeeper/quick-start-persistent-volume][quickstart_persistent] persistent storage.

如果您想在AWS或任何其他云提供商进行测试，我们建议使用deploy/zookeeper/快速启动的持久卷持久存储。

In case of local test, you'd may prefer to go with [deploy/zookeeper/quick-start-volume-emptyDir][quickstart_emptydir] `emptyDir`.

如果是本地测试，您可能更喜欢使用deploy/zookeeper/快速启动卷-emptyDir emptyDir。



### 基于脚本的安装
In this example we'll go with simple 1-node Zookeeper cluster on AWS and pick [deploy/zookeeper/quick-start-persistent-volume][quickstart_persistent].

在本例中，我们将在AWS上使用简单的1节点Zookeeper集群，并选择[deploy/zookeeper/quick-start-persistent-volume][quickstart_persistent].

Both [create][zookeeper-1-node-create.sh] and [delete][zookeeper-1-node-delete.sh]
shell scripts are available for simplification.  

创建和删除
可以使用shell脚本进行简化。



### 手动安装
如果您想手动部署Zookeeper，需要执行以下步骤:

### Namespace
Create **namespace**
```bash
kubectl create namespace zoo1ns
```

### Zookeeper
将Zookeeper部署到这个名称空间
```bash
kubectl apply -f zookeeper-1-node.yaml -n zoo1ns
```

Now Zookeeper should be up and running. Let's [explore Zookeeper cluster](#explore-zookeeper-cluster).

现在Zookeeper应该可以运行了。让我们来探索Zookeeper集群。



**IMPORTANT** 快速启动zookeeper安装主要用于测试目的。

For fine-tuned Zookeeper setup please refer to [advanced setup](#advanced-setup) options.  

有关微调的Zookeeper设置，请参考高级设置选项。





## Advanced setup
Advanced files are are located in [deploy/zookeeper/advanced][zookeeper-advanced] folder. 

高级文件位于deploy/zookeeper/ Advanced文件夹中。

All resources are separated into different files so it is easy to modify them and setup required options.  

所有资源都被分割到不同的文件中，所以很容易修改它们和设置所需的选项。

Advanced setup is available in two options:

高级设置是可在两个选项:

1. With [persistent volume][k8sdoc_persistent_volume]
1. With [emptyDir volume][k8sdoc_emptydir]

Each of these options have both `create` and `delete` scripts provided

每个选项都提供了创建和删除脚本

1. Persistent volume  [create][zookeeper-persistent-volume-create.sh] and [delete][zookeeper-persistent-volume-delete.sh] scripts
1. EmptyDir volume  [create][zookeeper-volume-emptyDir-create.sh] and [delete][zookeeper-volume-emptyDir-delete.sh] scripts

Step-by-step explanations:





一步一步执行的例子：

### Namespace
Create **namespace** in which all the rest resources would be created
```bash
kubectl create namespace zoons
```



### Zookeeper Service

Create service. This service provides DNS name for client access to all Zookeeper nodes.

创建服务。此服务为客户端访问所有Zookeeper节点提供DNS名称。

```bash
kubectl apply -f 01-service-client-access.yaml -n zoons
```
Should have as a result（结果应该是这样的）
```text
service/zookeeper created
```

### Zookeeper Headless Service / Zookeeper无头服务
Create headless service. This service provides DNS names for all Zookeeper nodes

创建无头服务。该服务为所有Zookeeper节点提供DNS名称

```bash
kubectl apply -f 02-headless-service.yaml -n zoons
```
Should have as a result
```text
service/zookeeper-nodes created
```



### Disruption Budget

Create budget. Disruption Budget instructs k8s on how many offline Zookeeper nodes can be at any time

创建预算。中断预算指示k8s在任何时候可以有多少离线Zookeeper节点

```bash
kubectl apply -f 03-pod-disruption-budget.yaml -n zoons
```
Should have as a result
```text
poddisruptionbudget.policy/zookeeper-pod-distribution-budget created
```



### Storage Class

This part is not that straightforward and may require communication with k8s instance administrator.

这一部分并不简单，可能需要与k8s实例管理员进行通信。

First of all, we need to decide, whether Zookeeper would use [Persistent Volume][k8sdoc_persistent_volume] 
as a storage or just stick to more simple [Volume][k8sdoc_volume] (In doc [emptyDir][k8sdoc_emptydir] type is used)

In case we'd prefer to stick with simpler solution and go with [Volume of type emptyDir][k8sdoc_emptydir], 
we need to go with **emptyDir StatefulSet config** [05-stateful-set-volume-emptyDir.yaml][05-stateful-set-volume-emptyDir.yaml] 
as described in next [Stateful Set unit](#stateful-set). Just move to [it](#stateful-set).

首先，我们需要决定，Zookeeper是否会使用持久卷
作为存储或只是坚持更简单的卷(在doc中使用emptyDir类型)

如果我们想坚持使用更简单的解决方案，使用类型为emptyDir的卷，
我们需要使用emptyDir StatefulSet config 05-stateful-set-volume-emptyDir.yaml
如在下一个有状态集单元中所述。只要动起来。



In case we'd prefer to go with [Persistent Volume][k8sdoc_persistent_volume] storage, we need to go 
with **Persistent Volume StatefulSet config** [05-stateful-set-persistent-volume.yaml][05-stateful-set-persistent-volume.yaml]

如果我们更喜欢使用持久卷存储，那么我们需要这样做
与持久卷状态设置配置05- status -set- persistence - Volume .yaml



Shortly, [Storage Class][k8sdoc_storage_class] is used to bind together [Persistent Volumes][k8sdoc_persistent_volume],
which are created either by k8s admin manually or automatically by [Provisioner][k8sdocs_dynamic_provisioning]. In any case, Persistent Volumes are provided externally to an application to be deployed into k8s. 

稍后，存储类用于将持久卷绑定在一起，
由k8s管理员手动或由供应者自动创建。在任何情况下，持久性卷都是从外部提供给要部署到k8s中的应用程序。



So, this application has to know **Storage Class Name** to ask for from the k8s in application's claim for new persistent volume - [Persistent Volume Claim][k8sdoc_persistent_volume_claim].

因此，这个应用程序必须知道“存储类名”，以便向应用程序claim中的k8s请求新的持久卷—持久卷claim。



This **Storage Class Name** should be asked from k8s admin and written as application's **Persistent Volume Claim** `.spec.volumeClaimTemplates.storageClassName` parameter in `StatefulSet` configuration. **StatefulSet manifest with emptyDir** [05-stateful-set-volume-emptyDir.yaml](../deploy/zookeeper/advanced/05-stateful-set-volume-emptyDir.yaml) and/or **StatefulSet manifest with Persistent Volume** [05-stateful-set-persistent-volume.yaml](../deploy/zookeeper/advanced/05-stateful-set-persistent-volume.yaml). 

应该向k8s管理员询问这个存储类名，并将其编写为应用程序的持久卷声明。spec. volumeclaimtemplates。有状态集配置中的storageClassName参数。带有emptyDir 05-stateful-set-volume-emptyDir的状态集清单。带有持久卷05- status -set- persistence - Volume .yaml的yaml和/或状态集清单。



### Stateful Set
Edit **StatefulSet manifest with emptyDir** [05-stateful-set-volume-emptyDir.yaml][05-stateful-set-volume-emptyDir.yaml] 
and/or **StatefulSet manifest with Persistent Volume** [05-stateful-set-persistent-volume.yaml][05-stateful-set-persistent-volume.yaml] 
according to your Storage Preferences.

In case we'd go with [Volume of type emptyDir][k8sdoc_emptydir], ensure `.spec.template.spec.containers.volumes` is in place 
and look like the following:

如果我们使用类型为emptyDir的卷，请确保.spec.template.spec.containers。卷已经到位
看起来像这样:

```yaml
      volumes:
      - name: datadir-volume
        emptyDir:
          medium: "" #accepted values:  empty str (means node's default medium) or Memory
          sizeLimit: 1Gi
```
and ensure `.spec.volumeClaimTemplates` is commented.

In case we'd go with **Persistent Volume** storage, ensure `.spec.template.spec.containers.volumes` is commented 
and ensure `.spec.volumeClaimTemplates` is uncommented.

如果我们使用持久卷存储，请确保.spec.template.spec.containers。卷是评论
并确保.spec。volumeClaimTemplates注释。

```yaml
  volumeClaimTemplates:
  - metadata:
      name: datadir-volume
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
## storageClassName has to be coordinated with k8s admin and has to be created as a `kind: StorageClass` resource
      storageClassName: storageclass-zookeeper
```
and ensure **storageClassName** (`storageclass-zookeeper` in this example) is specified correctly, as described 
in [Storage Class](#storage-class) section

As `.yaml` file is ready, just apply it with `kubectl`

并确保正确指定storageClassName(本例中为storageclas -zookeeper)，如所述
存储类节

当.yaml文件准备好后，只需将其应用到kubectl中

```bash
kubectl apply -f 05-stateful-set.yaml -n zoons
```
Should have as a result/结果应该有什么
```text
statefulset.apps/zookeeper-node created
```

Now we can take a look into Zookeeper cluster deployed in k8s:

现在我们可以看看在k8s中部署的Zookeeper集群:

## Explore Zookeeper cluster

### DNS names
We are expecting to have ZooKeeper cluster of 3 pods inside `zoons` namespace, named as:

我们希望在zoons命名空间中有3个pods的ZooKeeper集群，命名为:

```text
zookeeper-0
zookeeper-1
zookeeper-2
```
Those pods are expected to have short DNS names as:

这些pods将会有简短的DNS名称:

```text
zookeeper-0.zookeepers.zoons
zookeeper-1.zookeepers.zoons
zookeeper-2.zookeepers.zoons
```

where `zookeepers` is name of [Zookeeper headless service](#zookeeper-headless-service) and `zoons` is name of [Zookeeper namespace](#namespace).

and full DNS names (FQDN) as:

其中zookeepers是Zookeeper无头服务的名称，zoons是Zookeeper命名空间的名称。

和完整的DNS名称(FQDN)为:

```text
zookeeper-0.zookeepers.zoons.svc.cluster.local
zookeeper-1.zookeepers.zoons.svc.cluster.local
zookeeper-2.zookeepers.zoons.svc.cluster.local
```

### Resources/资源

在Zookeeper的名称空间中列出pods
```bash
kubectl get pod -n zoons
```

预期输出如下所示
```text
NAME             READY   STATUS    RESTARTS   AGE
zookeeper-0      1/1     Running   0          9m2s
zookeeper-1      1/1     Running   0          9m2s
zookeeper-2      1/1     Running   0          9m2s
```

List services
```bash
kubectl get service -n zoons
```

Expected output is like the following
```text
NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
zookeeper              ClusterIP   10.108.36.44   <none>        2181/TCP                     168m
zookeepers             ClusterIP   None           <none>        2888/TCP,3888/TCP            31m
```

List statefulsets
```bash
kubectl get statefulset -n zoons
```

Expected output is like the following
```text
NAME            READY   AGE
zookeepers      3/3     10m
```

如果所有看起来很好，Zookeeper集群已经启动并运行



[k8sdoc_service_main]: https://kubernetes.io/docs/concepts/services-networking/service/
[k8sdoc_service_headless]: https://kubernetes.io/docs/concepts/services-networking/service/#headless-services
[k8sdoc_disruption_balance]: https://kubernetes.io/docs/concepts/workloads/pods/disruptions/
[k8sdoc_storage_class]: https://kubernetes.io/docs/concepts/storage/storage-classes/
[k8sdoc_statefulset]: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
[k8sdoc_volume]: https://kubernetes.io/docs/concepts/storage/volumes
[k8sdoc_emptydir]: https://kubernetes.io/docs/concepts/storage/volumes/#emptydir
[k8sdoc_persistent_volume]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/
[k8sdoc_persistent_volume_claim]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims
[k8sdocs_dynamic_provisioning]: https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/

[quickstart_persistent]: ../deploy/zookeeper/quick-start-persistent-volume
[quickstart_emptydir]: ../deploy/zookeeper/quick-start-volume-emptyDir

[zookeeper-1-node-create.sh]: ../deploy/zookeeper/quick-start-persistent-volume/zookeeper-1-node-create.sh
[zookeeper-1-node-delete.sh]: ../deploy/zookeeper/quick-start-persistent-volume/zookeeper-1-node-delete.sh
[zookeeper-advanced]: ../deploy/zookeeper/advanced
[zookeeper-persistent-volume-create.sh]: ../deploy/zookeeper/advanced/zookeeper-persistent-volume-create.sh
[zookeeper-persistent-volume-delete.sh]: ../deploy/zookeeper/advanced/zookeeper-persistent-volume-delete.sh
[zookeeper-volume-emptyDir-create.sh]: ../deploy/zookeeper/advanced/zookeeper-volume-emptyDir-create.sh
[zookeeper-volume-emptyDir-delete.sh]: ../deploy/zookeeper/advanced/zookeeper-volume-emptyDir-delete.sh
[05-stateful-set-volume-emptyDir.yaml]: ../deploy/zookeeper/advanced/05-stateful-set-volume-emptyDir.yaml
[05-stateful-set-persistent-volume.yaml]: ../deploy/zookeeper/advanced/05-stateful-set-persistent-volume.yaml
