持久化存储

## 基本概念

- PVC

  PVC：PersistentVolumeClaim，是Pod希望使用的持久化存储的属性。比如，Volume的大小、可读写权限等。

- PV

  PV：PersistentVolume，持久化存储数据卷，这个 API 对象主要定义的是一个持久化存储在宿主机上的目录，比如一个 NFS 的挂载目录。

PVC类似于一个接口，PV类似于一个实现。两者依靠PersistentVolumeController来绑定（Bound）起来。当Pod声明的PVC（在Pod的spec.volumes字段声明要使用的PVC名字以实现关联）已经绑定了PV时，Pod就能正常启动并访问PV中的数据。

PVC一般由开发人员定义所需的Volume的规格，而PV由运维人员来创建。

- StorageClass

  StorageClass是PV的模板，也由运维人员创建。它定义了PV的**规格、存储类型**以及创建这种PV所需的**存储插件（CSI）**等配置。

  - **存储插件**实现了PV的动态创建、删除、扩容等操作。接着由k8s的**Dynamic Provisioner**根据StorageClass自动创建PV。

  - 在PVC的`spec.storageClassName`字段声明要使用的StorageClass名字。

  - 但SC并不是专为Dynamic Provisioner服务的，也可以用于静态PV的绑定。例如设置`storageClassName=manual`，此时集群里并没有一个名为`manual`的StorageClass，但是可以手动创建一个PV，然后将PVC的`storageClassName`设置为`manual`，这样PVC就会绑定到这个PV上。这属于**Static Provisioning**。但注意的是，在k8s继续PV和PVC绑定时，仍要满足`storageClassName`字段是一致的才能绑定成功。

![image](k8s-storage.png)

- CSI 

  Container Storage Interface，容器存储接口，是一个标准化的存储插件接口，允许存储供应商为容器化应用程序提供存储解决方案。

## 应用场景

### 1. 使用CSI存储插件的Volume持久化创建、挂载流程

1. 简要流程：

> 用户提交请求创建pod，Kubernetes发现这个pod声明使用了PVC，那就靠PersistentVolumeController帮它找一个PV配对。
>
> 没有现成的PV，就去找对应的StorageClass，帮它新创建一个PV，然后和PVC完成绑定。
>
> 新创建的PV，还只是一个API 对象，需要经过“两阶段处理”变成宿主机上的“持久化Volume”才真正有用：
> 第一阶段由运行在master上的AttachDetachController负责，力这个PV完成 Attach 操作，为宿主机挂载远程磁盘；
> 第二阶段是运行在每个节点上kubelet组件的内部，把第一步attach的远程磁盘 mount 到宿主机目录。这个控制循环叫VolumeManagerReconciler，运行在独立的Goroutine，不会阻塞kubelet主循环。
>
> 完成这两步，PV对应的“持久化 Volume”就准备好了，POD可以正常启动，将“持久化Volume”挂载在容器内指定的路径。


2. Csi存储插件的Volume持久化创建、挂载的详细流程
   ![image](csi-storage-workflow.png)

<details>
  <summary>PlantUML源码</summary>

  ```plantuml
  @startuml
  title CSI Storage Plugin PV Creation and Mounting Workflow

  participant "User" as User

  box "master"
  participant "Kubernetes API Server" as APIServer
  participant "Volume Controller" as VolCtrl
  end box

  box "Node B with CSI Controller Service"
  participant "External Provisioner" as Prov
  participant "External Attacher" as Attacher
  participant "CSI Plugin b" as CSIPlugin_b
  end box

  box "Node A with CSI Node Service"
  participant "Kubelet" as Kubelet
  participant "CSI Plugin a" as CSIPlugin_a
  end box


  == Provision 阶段 ==
  note over User: Create PVC and PV
  User -> APIServer: Create PVC

  APIServer -> Prov: [监听]External Provisioner 监听到 PVC 创建事件
  Prov -> CSIPlugin_b: 调用 `CreateVolume` 方法
  CSIPlugin_b -> CSIPlugin_b: 创建PV
  CSIPlugin_b -> APIServer: PV created

  APIServer -> VolCtrl: [监听]`PersistentVolumeController`控制循环监听到一组\n相同 StorageClass 的 PV 和 PVC
  VolCtrl -> VolCtrl: 将 PV 和 PVC 绑定在一起
  APIServer --> User: PVC enters Bound state


  == Attach 阶段 ==
  note over User: Create Pod
  User -> APIServer: Create Pod with PVC
  APIServer -> APIServer: Schedule Pod to Node A

  APIServer -> VolCtrl: [监听]`AttachDetachController`控制循环监听到有Volume\n需要被Attach到Node A上
  VolCtrl -> VolCtrl: 创建`VolumeAttachment`对象
  APIServer -> Attacher: [监听]External Attacher 监听到 VolumeAttachment event
  Attacher -> CSIPlugin_b: 调用 `ControllerPublishVolume` 方法，将存储卷挂载到宿主机Node A上
  CSIPlugin_b -> APIServer: Attach completed


  == Mount 阶段 ==
  Kubelet -> Kubelet: [监听]`VolumeManagerReconciler`循环发现节点上有Volume设备Attach了

  Kubelet -> CSIPlugin_a: 调用 `NodeStageVolume` 方法
  CSIPlugin_a -> CSIPlugin_a: 格式化Volume\n并挂载到临时目录
  CSIPlugin_a -> Kubelet: Volume staged

  Kubelet -> CSIPlugin_a: 调用 `NodePublishVolume` 方法
  CSIPlugin_a -> CSIPlugin_a: 将临时目录挂载到\nVolume对应的宿主机目录
  CSIPlugin_a -> Kubelet: Volume mounted

  Kubelet -> APIServer: Volume status update

  @enduml
  ```
</details>

