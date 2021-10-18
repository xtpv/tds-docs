---
id: features
title: 云引擎功能介绍
sidebar_label: 功能介绍
---


云引擎是基于 Docker 的容器云计算平台。云引擎既可以被简单地用来托管静态网站，又可以接受任意程序语言的定制开发来动态处理外来请求，满足业务定制化需求。

## 实用功能

你只需要提供服务端的业务逻辑（网站或云函数等），而服务端的多实例负载均衡，不中断服务的平滑升级等都由云引擎提供支持。用最少代码，提供最大价值。

### Hook 函数

类似于数据库 trigger，可以在特定操作或事件（如数据更新、用户登录）发生前后，自动触发，完成额外的业务处理。

### 云函数

定义自己的 HTTP Endpoint，可以将复杂的客户端逻辑独立成一个公共的「云端函数」，让业务灵活可控。

### 定时任务

周期性执行云函数，比如定时清理过期数据、每周一向所有用户发送推送消息等等，让业务自动化运行。

### LeanCache

基于分布式 Redis 集群技术，可承载高达 70,000 QPS 的峰值流量，轻松应对抢红包、秒杀购物等高并发场景。

### 实例分组管理

将云引擎实例按照用途分组，实现在访问同一数据源的情况下，部署多份云引擎代码，满足不同的业务需求。

### 灵活的部署方式

支持在线编写代码、通过命令行工具部署本地项目、以及部署远程 Git 源码等方式，让开发流程更简单。

## 优势和特色

- 基于 Docker 的容器云。项目代码运行在独立的 Docker 容器中，资源隔离，可以平滑部署、弹性扩容。

- 自动负载均衡。基于自动服务发现技术，多个云引擎实例之间实时进行负载均衡，在系统意外发生时会自动进行故障转移，提供超过 99.9% 的高可用性。

- 支持主流开发语言。支持 Node.js、Python、Java、PHP 等主流后端语言和运行时，并提供多种类型项目模版，10 分钟内即可完成项目发布。
