---
id: cluster
title: Cluster
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

## 集群

返回由YuniKorn调度程序管理的群集常规信息，信息内容包括应用程序和容器的数量（包括总计、失败、挂起、正在运行和已完成的数量）。

**URL** : `/ws/v1/clusters`

**方法** : `GET`

**是否需要认证** : NO

### 成功返回

**返回代码** : `200 OK`

**样例内容**

例如，下面是一个2节点集群的响应，其中包含3个应用程序和4个正在运行的容器。

```json
[
    {
        "clusterName": "kubernetes",
        "totalApplications": "3",
        "failedApplications": "1",
        "pendingApplications": "",
        "runningApplications": "3",
        "completedApplications": "",
        "totalContainers": "4",
        "failedContainers": "",
        "pendingContainers": "",
        "runningContainers": "4",
        "activeNodes": "2",
        "totalNodes": "2",
        "failedNodes": ""
    }
]
```
		