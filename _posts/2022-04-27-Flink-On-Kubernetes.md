---
layout: post
date:   2022-04-27 18:20:00
title:  "Flink-on-Kubernetes"
categories: Flink
tags:  Flink Kubernetes flink-on-kubernetes
mathjax: true
---

* content
{:toc}

Flink on Native Kubernetes.

2022-04-27-Flink-On-Kubernetes.md

# 1. 背景
Kubernetes 由于出色的容器编排能力及扩展性,在众多容器化项目厮杀中胜出,已经成为企业一项重要的基础设施;但由于其上运行的多为服务类应用,资源使用存峰谷,资源利用率较低。相反基于Hadoop大数据计算密集型集群存在计算资源不足的情况, Flink on K8s 与服务类应用混部解决大数据计算资源不足、资源利用率的需求油然而生;另外AI 类任务复杂的环境基于镜像的部署有着天然的优势,无Hadoop基础设施使用 Flink 的场景越来越多。Flink on K8s 的价值是明显的, 也为计算引擎平台化带来了更多的扩展性。但最有价值的是Kubernetes作为可获取的云可伸缩性资源运行Flink计算按需计费是经济的。
# 2. Flink On Native Kubernetes 架构

# 3. Flink On Native Kubernetes 部署

# 4. Session & Application 模式作业提交
# 5. 集成 Hive 
# 6. 总结
1. Client 一次性提交 ConfigMap， Job Manager, Job Manager Deployment 等资源描述符以及授权信息。
2. 创建 k8s Service -> 将 flink k8s Deployment