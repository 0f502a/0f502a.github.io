---
layout: post
title: Training Operator
date: 2024-05-10 13:48 +0800

categories: [k8s]
tags: [k8s, 学习笔记, HPC]

img_path: /assets/img/2024-05-10-training-operator.assets
---

## 概念

### Operator模式

Kubernetes Operator模式是一种扩展Kubernetes功能的机制，它允许用户定义自己的资源类型来管理应用程序或服务。Operator模式基于Kubernetes控制器概念构建，但它提供了一些额外的功能，使其更适用于管理复杂的应用程序。

Operator模式的核心是自定义资源（CR）和控制器。CR是用户定义的资源类型，用于表示要管理的应用程序或服务。控制器是监视CR并根据其状态采取相应行动的程序。


### Training operator

Kubeflow是一个开源的机器学习平台，用于在Kubernetes上部署和管理机器学习工作流程。Training Operator是Kubeflow的一个组件，用于训练机器学习模型。

Training operator定义了一套模型训练相关的CR，以及用于管理CR的控制器，能够实现在分布式的集群中部署训练任务。

![image](ml-lifecycle-training-operator.drawio.svg)

## 执行流程

1. **基于 Training Operator SDK 编写训练任务**: 开发人员使用 Training Operator SDK 定义训练任务的配置，例如训练使用的镜像、代码、数据、参数等。SDK 提供了各种 API 来简化训练任务的定义，例如定义资源规格、配置环境变量、挂载数据卷等。

2. **提交 CR**: 将训练任务配置转换为 Kubernetes 自定义资源 (CR)，并提交到 Kubernetes API 服务器。CR 的类型通常以 "Job" 结尾，例如 "PyTorchJob" 或 "TFJob"。

3. **Training Operator 控制器监视 CR**: Training Operator 控制器会持续监视 Kubernetes API 服务器上的 CR 状态。当发现新的 CR 或 CR 状态发生变化时，控制器会根据 CR 的配置采取相应的操作。

4. **调度 Job 到工作节点**: 控制器会根据 CR 的资源需求，从 Kubernetes 集群中选择合适的节点来运行训练任务。节点的选择需要考虑多种因素，例如节点的硬件资源、GPU 数量、网络性能等。

5. **在节点上执行训练任务**: 在选定的工作节点上，控制器会启动训练任务的 Pod。Pod 负责运行训练任务的容器。容器内包含训练所需的镜像、代码、数据和环境。控制器会监控 Pod 的运行状态，并根据需要进行调整或重启。

6. **训练任务完成**: 当训练任务完成时，Pod 会正常退出。控制器会更新 CR 的状态，并进行后续清理工作，例如释放资源等。

补充：

- Training operator可以和其他k8s工具一起使用：例如结合Volcano调度器，扩展调度能力。

## 参考文档

- [Training Operator官方文档](https://www.kubeflow.org/docs/components/training/overview/)
