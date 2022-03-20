---
id: events
title: 事件
---

<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

即将举行的会议
---

加入我们 **4:30pm - 5:30pm，太平洋标准时间，2022 年 2 月 16 日**

We are hosting an Apache YuniKorn (Incubating) meetup on Wednesday February 16. Come join us! Since our first meetup held in November last year, lots of new development has happened in the community. We are excited to have three talks lined up! Add this event to your calendar: [:calendar:](https://calendar.google.com/calendar/u/0/r/eventedit/copy/N21jYmxiYWx0c211M2pvMTIydDZxZ2s5ajAgYXBhY2hlLnl1bmlrb3JuQG0/Y2hlbnlhemhhbmdjaGVueWFAZ21haWwuY29t?cid=YXBhY2hlLnl1bmlrb3JuQGdtYWlsLmNvbQ)

Erica Lin, Luna Xu, and Sean Gorsky from Sync Computing will present their work on Spark autotuner and orchestrator.
Description:
Efficiently managing the infrastructure and schedules of thousands of data pipelines in multi-tenant and heterogeneous environments is a daunting task. Poor application tuning, resource allocation, and scheduling can lead to exorbitant costs on the cloud, sluggish performance, and failed jobs due to the intractable infrastructure search space. Addressing each of these codependent issues separately also often doesn't lead to the best results overall. Sync Computing will share how their autotuner and orchestrator addresses these issues jointly as a single optimization problem. As a result, the solution offers globally optimized Spark configurations, resource provisioning, and job scheduling with configurable optimization goals, enabling a seamless user experience of running DAG workflows in the cloud. We experimentally demonstrate on AWS up to 77% cost savings or 45% performance acceleration of a Spark and Airflow DAG. Simulations of our solution on a multi-day Alibaba cloud trace resulted in a 57% reduction in total DAG completion time for their batch workloads. This work could be complementary to Yunikorn and we would like to discuss potential integration strategies with the community.

Craig Condit from Cloudera will talk about the latest YuniKorn release v0.12.2 and offer a glimpse of what the upcoming v1.0 release has in store.
Description:
v0.12.2 release offers a host of new features and bug fixes. Specifically, the admission controller implementation has been refactored to enable the YuniKorn scheduler to be seamless maintained and upgraded. In the upcoming v1.0 release, we will witness a major milestone of the YuniKorn project. An exciting feature is a new mode of deployment based on the Kubernetes Scheduling Framework, which offers better compatibility with the default scheduler and makes it easier for new users to try out YuniKorn.

Tingyao Huang and Yuteng Chen are both YuniKorn committers. They have done extensive performance benchmarking using the latest YuniKorn codebase. Their results showed convincing throughput advantages of YuniKorn compared to the default scheduler.

----

Past Meetups
---

**4:30pm - 6:00pm, PST, Nov 18, 2021**

**Wilfred Spiegelenburg** presented a session to introduce the latest status in the YuniKorn community.

_Abstract_: Apache YuniKorn (Incubating) 今年早些时候发布了 v0.11 版本，其中包含了许多新功能和改进，比如 Gang scheduling、REST API 增强和 Kubernetes 1.19 支持。
在一个月内，我们计划发布主要的 v1.0.0 版本，它将支持 Kubernetes 1.20 和 1.21、改进的节点排序以及大量的小修复和增强功能！
在本次会议中，我们将深入探讨在 Kubernetes 上使用临时占位 pod 背后的 Gang scheduling 实现、通过简化的调度器核心代码和新的节点排序算法显著提高性能。
请在 [此处](https://docs.google.com/document/d/1-NP0J22-Gp3cZ_hfKyA9htXJw7tlk-BmljF-7CBJg44) 找到本次会议的所有主题。

----

过去的会议
---

- ApacheCon 2021: [Next Level Spark on Kubernetes with Apache YuniKorn (Incubating)](https://youtu.be/gOST-iT-hj8) :busts_in_silhouette: Weiwei Yang, Chaoran Yu
- ApacheCon Asia 2021: [State Of The Union With Apache YuniKorn (Incubating) - Cloud Native Scheduler For Big Data Usecases](https://www.youtube.com/watch?v=c9UYxzqVMeg)  :busts_in_silhouette: Julia Kinga Marton, Sunil Govindan
- Future of Data Meetup: [Apache YuniKorn: Cloud-native Resource Scheduling](https://www.youtube.com/watch?v=j-6ehu6GrwE) :busts_in_silhouette: Wangda Tan, Wilfred Spiegelenburg
- ApacheCon 2020: [Integrate Apache Flink with Cloud Native Ecosystem](https://youtu.be/4hghJCuZk5M) :busts_in_silhouette: Yang Wang, Tao Yang
- Spark+AI Summit 2020: [Cloud-Native Spark Scheduling with YuniKorn Scheduler](https://www.youtube.com/embed/ZA6aPZ9r9wA) :busts_in_silhouette: Gao Li, Weiwei Yang
- Flink Forward 2020: [Energize multi-tenant Flink on K8s with YuniKorn](https://www.youtube.com/embed/NemFKL0kK9U) :busts_in_silhouette: Weiwei Yang, Wilfred Spiegelenburg


演示视频
---

请订阅 [YuniKorn Youtube Channel](https://www.youtube.com/channel/UCDSJ2z-lEZcjdK27tTj_hGw) 获取有关新演示的通知！
- [Running YuniKorn on Kubernetes - a 12 minutes Hello-world demo](https://www.youtube.com/watch?v=cCHVFkbHIzo)
- [YuniKorn configuration hot-refresh introduction](https://www.youtube.com/watch?v=3WOaxoPogDY)
- [YuniKorn scheduling and volumes on Kubernetes](https://www.youtube.com/watch?v=XDrjOkMp3k4)
- [YuniKorn placement rules for applications](https://www.youtube.com/watch?v=DfhJLMjaFH0)
- [Federated K8s compute pools with YuniKorn](https://www.youtube.com/watch?v=l7Ydg_ZGZw0&t)

如果您是 YuniKorn 的布道者，并且您有公开的会议演讲、与 YuniKorn 相关的演示录音，请提交 PR 以扩展此列表！
