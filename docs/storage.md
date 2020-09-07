# Storage

Examples are available in [examples][chi-examples] folder:

示例可在示例文件夹:

1. [Simple Default Persistent Volume][03-persistent-volume-01-default-volume.yaml]
1. [Pod Template with Persistent Volume][03-persistent-volume-02-pod-template.yaml]
1. AWS-based cluster with data replication and Persistent Volumes [minimal][04-replication-zookeeper-03-minimal-AWS-persistent-volume.yaml] 
and [medium][04-replication-zookeeper-04-medium-AWS-persistent-volume.yaml] Zookeeper installations



简单默认持久卷

带有持久卷的Pod模板

基于aws的集群，包含数据复制和持久卷，最小和中等的Zookeeper安装



## Persistent Volumes
k8s cluster administrator provision storage to applications (users) via `PersistentVolume` objects. 
Applications (users) claim storage with `PersistentVolumeClaim` objects and then mount claimed `PersistentVolume`s into filesystem via `volumeMounts`+`volumes`.



k8s集群管理员通过PersistentVolume对象为应用程序(用户)提供存储空间。
应用程序(用户)使用PersistentVolumeClaim对象声明存储，然后通过volumeMounts+卷将声明的persistent卷挂载到文件系统中。

`PersistentVolume` can be created as:

PersistentVolume可以创建为:

1. **Manual volume provisioning**. Cluster administrator manually make calls to storage (cloud) provider to provision new storage volumes, and then create `PersistentVolume` objects to represent those volumes in Kubernetes.
Users claim those `PersistentVolume`s later via `PersistentVolumeClaim`s
1. **Dynamic volume provisioning**. No need for cluster administrators to pre-provision storage manually.
Storage resources are dynamically provisioned by special software module, called provisioner, which is specified by the `StorageClass` object.
`StorageClass`es abstract the underlying storage provider with all parameters (such as disk type or location).
`StorageClass`es use software modules - provisioners that are specific to the storage platform or cloud provider to give Kubernetes access to the physical media being used.



1、手动卷配置。集群管理员手动调用存储(云)提供者来提供新的存储卷，然后创建PersistentVolume对象来在Kubernetes中表示这些卷。
用户稍后通过PersistentVolumeClaims声明这些persistentvolume



2、动态卷配置。集群管理员不需要手动预置存储空间。
存储资源是由特殊的软件模块动态提供的，称为供应器，由StorageClass对象指定。
storagecl类使用所有参数(如磁盘类型或位置)抽象底层存储提供程序。

storagecl类使用软件模块——特定于存储平台或云提供商的供应器，使Kubernetes能够访问所使用的物理介质。



### What it is and how to use `StorageClass`

Applications (users) refer `StorageClass` by name in the `PersistentVolumeClaim` with `storageClassName` parameter.

应用程序(用户)使用storageClassName参数在PersistentVolumeClaim中通过名称引用StorageClass。

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
  namespace: mytestns
spec:
  storageClassName: my-storage-class
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
```
Storage class name - `my-storage-class` in this example - is specific for each k8s installation and has to be provided (announced to applications(users)) by cluster administrator. 

存储类名——本例中为my-storage-class——特定于每个k8s安装，必须由集群管理员提供(通知应用程序(用户))。

However, this is not convenient and sometimes we'd like to just use **any** available storage, without bothering to know what storage classes are available in this k8s installation.

然而，这并不方便，有时我们只想使用任何可用的存储，而不费心去知道这个k8s安装中有哪些存储类可用。

The cluster administrator have an option to specify a **default `StorageClass`**. 

集群管理员可以选择指定默认的StorageClass。

When present, the user can create a `PersistentVolumeClaim` having no `storageClassName` specified, simplifying the process and reducing required knowledge of the underlying storage provider.

当存在时，用户可以创建不指定storageClassName的PersistentVolumeClaim，从而简化过程并减少所需的底层存储提供程序知识。

Important notes on `PersistentVolumeClaim` 

`PersistentVolumeClaim`的重要的笔记 

1. if `storageClassName` is not specified, **default `StorageClass`** (must be specified by cluster administrator) would be used for provisioning
1. if `storageClassName` is set to an empty string (""), no **`StorageClass`** will be used, and thus, dynamic provisioning is efficiently disabled for this `PersistentVolumeClaim`. 
Available PVs that do not have any `storageClassName` specified  will be considered for binding to this PVC
1. if `storageClassName` is set, then the matching `StorageClass` will be used for provisioning



1、如果没有指定storageClassName，则将使用缺省的StorageClass(必须由集群管理员指定)进行配置

2、如果storageClassName设置为空字符串(“”)，则不会使用StorageClass，因此，对于这个PersistentVolumeClaim，动态配置被有效地禁用了。
将考虑将没有指定任何storageClassName的可用pv绑定到此PVC

3、如果设置了storageClassName，那么将使用匹配的StorageClass进行供应






## AWS-specific
We can use `kubectl` to check for `StorageClass` objects. Here we use cluster created with `kops`

我们可以使用kubectl检查StorageClass对象。这里我们使用kops创建的集群

```bash
kubectl get storageclasses.storage.k8s.io 
```
```text
NAME            PROVISIONER             AGE
default         kubernetes.io/aws-ebs   1d
gp2 (default)   kubernetes.io/aws-ebs   1d
```
We can see two storage classes available:

我们可以看到两个存储类可用:

1. named as **default**
1. named as **gp2** which is the **default `StorageClass`**

We can take a look inside them as: 

我们可以看看它们的内部:

```bash
kubectl get storageclasses.storage.k8s.io default -o yaml
kubectl get storageclasses.storage.k8s.io gp2 -o yaml
```
What we can see, that, actually, those `StorageClass`es are equal:

我们可以看到，实际上，这些storagecl类是相等的:

```text
metadata:
  labels:
    k8s-addon: storage-aws.addons.k8s.io
  name: gp2
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

```text
metadata:
  labels:
    k8s-addon: storage-aws.addons.k8s.io
  name: default
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

What does this mean - we can specify our `PersistentVolumeClaim` object with either: 

这意味着什么-我们可以指定我们的PersistentVolumeClaim对象:

1. no `storageClassName` specified (just omit this field) - and in this case `StorageClass` named **gp2** would be used (because it is the **default** one) or 
1. specify



1、没有指定storageClassName(忽略此字段)——在本例中，将使用名为gp2的StorageClass(因为它是默认值)或

2、指定

```yaml
storageClassName: default
```
and in this case `StorageClass` named **default** would be used. The result would be the same as when `StorageClass` named **gp2** used (which is actually the **default `StorageClass`** in the system)

在本例中，将使用名为default的StorageClass。结果将与使用名为gp2的StorageClass时相同(这实际上是系统中的默认StorageClass)


## Pods

Pods use `PersistentVolumeClaim` as **volume**.

Pods使用持久的容积作为容积。

`PersistentVolumeClaim` must exist in the same namespace as the pod using the claim.

PersistentVolumeClaim必须与使用声明的pod存在在同一个名称空间中。

The k8s inspects the `PersistentVolumeClaim` to find appropriate `PersistentVolume` and mounts that `PersistentVolume` into pod's filesystem via `volumeMounts`.

k8s检查PersistentVolumeClaim以找到适当的持久化卷，并通过volumeMounts将这个持久化卷挂载到pod的文件系统中。

A Pod refers "volumes: name" via "volumeMounts: name" in Pod or Pod Template as:

在Pod或Pod模板中，通过“volumeMounts: name”表示“卷:名称”为:

```yaml
# ...
# excerpt from Pod or Pod Template manifest
# ...
containers:
  - name: myclickhouse
    image: clickhouse
    volumeMounts:
      - mountPath: "/var/lib/clickhouse"
        name: my-volume
```
This "volume" definition can either be the final object description of different types, such as:

这个“卷”定义可以是不同类型的最终对象描述，例如

Volume of type `emptyDir`

类型为emptyDir的卷

```yaml
# ...
# excerpt from manifest
# ...
volumes:
  - name: my-volume
    emptyDir: {}
```
Volume of type `hostPath`

类型为`hostPath`的卷

```yaml      
# ...
# excerpt from StatefulSet manifest
# ...
volumes:
  - name: my-volume
    hostPath:
      path: /local/path/
```
or can refer to `PersistentVolumeClaim` as:

或者可以指代PersistentVolumeClaim:

```yaml
# ...
# excerpt from manifest
# ...
volumes:
  - name: my-volume
    persistentVolumeClaim:
      claimName: my-claim
```
where minimal `PersistentVolumeClaim` can be specified as following:

其中最小持续容量可指定如下:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```
Pay attention, that there is no `storageClassName` specified - meaning this `PersistentVolumeClaim` will claim `PersistentVolume` of explicitly specified **default** `StorageClass`.

注意，这里没有指定storageClassName——这意味着这个PersistentVolumeClaim将声明显式指定的默认StorageClass的PersistentVolume。



More details on [storageClassName][persistent-volumes-class-1]

关于storageClassName的更多细节

More details on [PersistentVolumeClaim][persistentvolumeclaims]

关于持久容量的更多细节

Example on how this `persistentVolumeClaim` named `my-pvc` can be used in Pod spec:

例子如何这个持久性volumeclaim命名为my-pvc可用于Pod规范:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  volumes:
  - name: www
    persistentVolumeClaim:
      claimName: my-pvc
  containers:
  - name: nginx
    image: k8s.gcr.io/nginx-slim:0.8
    ports:
    - containerPort: 80
      name: web
    volumeMounts:
    - name: www
      mountPath: /usr/share/nginx/html
```



## StatefulSet

`StatefulSet` shortcuts the way, jumping from `volumeMounts` directly to `volumeClaimTemplates`, skipping `volume`.

StatefulSet快捷方式，从volumeMounts直接跳到volumeClaimTemplates，跳过volume。

More details in [StatefulSet description][creating-a-statefulset]

更多细节见状态集描述

StatefulSet example:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```
Pay attention to `.spec.template.spec.containers.volumeMounts`:

注意

```yaml
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
```
refers directly to:

直接引用:

```yaml
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```



## AWS encrypted volumes/AWS加密卷

As we have discussed in [AWS-specific](#AWS-specific) section, AWS provides **gp2** volumes as default media.

正如我们在特定于AWS的一节中所讨论的，AWS提供gp2卷作为默认媒体。

Let's create **encrypted** volume based on the same **gp2** volume.

让我们基于相同的gp2卷创建加密的卷。

Specify special `StorageClass`

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: encrypted-gp2
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
  encrypted: "true"
reclaimPolicy: Delete
volumeBindingMode: Immediate
```
and use it with `PersistentVolumeClaim`:

并与PersistentVolumeClaim一起使用:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: encrypted-pvc
spec:
  storageClassName: encrypted-gp2
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  resources:
    requests:
      storage: 1Gi
```

[chi-examples]: ./chi-examples
[03-persistent-volume-01-default-volume.yaml]: ./chi-examples/03-persistent-volume-01-default-volume.yaml
[03-persistent-volume-02-pod-template.yaml]: ./chi-examples/03-persistent-volume-02-pod-template.yaml
[04-replication-zookeeper-03-minimal-AWS-persistent-volume.yaml]: ./chi-examples/04-replication-zookeeper-03-minimal-AWS-persistent-volume.yaml
[04-replication-zookeeper-04-medium-AWS-persistent-volume.yaml]: ./chi-examples/04-replication-zookeeper-04-medium-AWS-persistent-volume.yaml
[persistentvolumeclaims]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims
[persistent-volumes-class-1]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#class-1
[creating-a-statefulset]: https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/#creating-a-statefulset
