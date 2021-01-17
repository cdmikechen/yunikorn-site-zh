---
id: scheduler
title: Scheduler
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

## 队列

显示有关队列的常规信息，如名称、状态、容量和属性。
队列的层次结构保存在响应json中。 

**URL** : `/ws/v1/queues`

**方法** : `GET`

**是否需要认证** : NO

### 成功返回

**返回代码** : `200 OK`

**样例内容**

对于默认队列层次结构（仅限于`root.default`的叶队列存在），以下内容的类似响应将发送回客户端：

```json
{
    "partitionName": "[mycluster]default",
    "capacity": {
        "capacity": "map[ephemeral-storage:75850798569 hugepages-1Gi:0 hugepages-2Mi:0 memory:80000 pods:110 vcore:60000]",
        "usedcapacity": "0"
    },
    "nodes": null,
    "queues": {
        "queuename": "root",
        "status": "Active",
        "capacities": {
            "capacity": "[]",
            "maxcapacity": "[ephemeral-storage:75850798569 hugepages-1Gi:0 hugepages-2Mi:0 memory:80000 pods:110 vcore:60000]",
            "usedcapacity": "[memory:8000 vcore:8000]",
            "absusedcapacity": "[memory:54 vcore:80]"
        },
        "queues": [
            {
                "queuename": "default",
                "status": "Active",
                "capacities": {
                    "capacity": "[]",
                    "maxcapacity": "[]",
                    "usedcapacity": "[memory:8000 vcore:8000]",
                    "absusedcapacity": "[]"
                },
                "queues": null,
                "properties": {}
            }
        ],
        "properties": {}
    }
}
```

## 应用

显示有关应用程序的常规信息，如已用资源、队列名称、提交时间和分配等。

**URL** : `/ws/v1/apps`

**方法** : `GET`

**查询参数** : 

1. queue=<fully qualified queue name\>

用于筛选在给定队列中运行的应用程序的队列全称。例如，"/ws/v1/apps?queue=root.default" 返回在 "root.default" 排队中运行的应用程序。

**是否需要认证** : NO

### 成功返回

**返回代码** : `200 OK`

**样例内容**

在下面的示例中，有3个资源分配属于两个应用。

```json
[
    {
        "applicationID": "application-0002",
        "usedResource": "[memory:4000 vcore:4000]",
        "partition": "[mycluster]default",
        "queueName": "root.default",
        "submissionTime": 1595939756253216000,
        "allocations": [
            {
                "allocationKey": "deb12221-6b56-4fe9-87db-ebfadce9aa20",
                "allocationTags": null,
                "uuid": "9af35d44-2d6f-40d1-b51d-758859e6b8a8",
                "resource": "[memory:4000 vcore:4000]",
                "priority": "<nil>",
                "queueName": "root.default",
                "nodeId": "node-0001",
                "applicationId": "application-0002",
                "partition": "default"
            }
        ],
        "applicationState": "Running"
    },
    {
        "applicationID": "application-0001",
        "usedResource": "[memory:4000 vcore:4000]",
        "partition": "[mycluster]default",
        "queueName": "root.default",
        "submissionTime": 1595939756253460000,
        "allocations": [
            {
                "allocationKey": "54e5d77b-f4c3-4607-8038-03c9499dd99d",
                "allocationTags": null,
                "uuid": "08033f9a-4699-403c-9204-6333856b41bd",
                "resource": "[memory:2000 vcore:2000]",
                "priority": "<nil>",
                "queueName": "root.default",
                "nodeId": "node-0001",
                "applicationId": "application-0001",
                "partition": "default"
            },
            {
                "allocationKey": "af3bd2f3-31c5-42dd-8f3f-c2298ebdec81",
                "allocationTags": null,
                "uuid": "96beeb45-5ed2-4c19-9a83-2ac807637b3b",
                "resource": "[memory:2000 vcore:2000]",
                "priority": "<nil>",
                "queueName": "root.default",
                "nodeId": "node-0002",
                "applicationId": "application-0001",
                "partition": "default"
            }
        ],
        "applicationState": "Running"
    }
]
```

## 节点

显示有关YuniKorn所管理的节点的常规信息。
包括主机和机架名称、容量、资源和分配等在内的节点详细信息。

**URL** : `/ws/v1/nodes`

**方法** : `GET`

**是否需要认证** : NO

### 成功返回

**返回代码** : `200 OK`

**样例内容**

这里您可以看到一个来自具有3个资源分配给2节点集群的响应示例。

```json
[
    {
        "partitionName": "[mycluster]default",
        "nodesInfo": [
            {
                "nodeID": "node-0001",
                "hostName": "",
                "rackName": "",
                "capacity": "[ephemeral-storage:75850798569 hugepages-1Gi:0 hugepages-2Mi:0 memory:14577 pods:110 vcore:10000]",
                "allocated": "[memory:6000 vcore:6000]",
                "occupied": "[memory:154 vcore:750]",
                "available": "[ephemeral-storage:75850798569 hugepages-1Gi:0 hugepages-2Mi:0 memory:6423 pods:110 vcore:1250]",
                "allocations": [
                    {
                        "allocationKey": "54e5d77b-f4c3-4607-8038-03c9499dd99d",
                        "allocationTags": null,
                        "uuid": "08033f9a-4699-403c-9204-6333856b41bd",
                        "resource": "[memory:2000 vcore:2000]",
                        "priority": "<nil>",
                        "queueName": "root.default",
                        "nodeId": "node-0001",
                        "applicationId": "application-0001",
                        "partition": "default"
                    },
                    {
                        "allocationKey": "deb12221-6b56-4fe9-87db-ebfadce9aa20",
                        "allocationTags": null,
                        "uuid": "9af35d44-2d6f-40d1-b51d-758859e6b8a8",
                        "resource": "[memory:4000 vcore:4000]",
                        "priority": "<nil>",
                        "queueName": "root.default",
                        "nodeId": "node-0001",
                        "applicationId": "application-0002",
                        "partition": "default"
                    }
                ],
                "schedulable": true
            },
            {
                "nodeID": "node-0002",
                "hostName": "",
                "rackName": "",
                "capacity": "[ephemeral-storage:75850798569 hugepages-1Gi:0 hugepages-2Mi:0 memory:14577 pods:110 vcore:10000]",
                "allocated": "[memory:2000 vcore:2000]",
                "occupied": "[memory:154 vcore:750]",
                "available": "[ephemeral-storage:75850798569 hugepages-1Gi:0 hugepages-2Mi:0 memory:6423 pods:110 vcore:1250]",
                "allocations": [
                    {
                        "allocationKey": "af3bd2f3-31c5-42dd-8f3f-c2298ebdec81",
                        "allocationTags": null,
                        "uuid": "96beeb45-5ed2-4c19-9a83-2ac807637b3b",
                        "resource": "[memory:2000 vcore:2000]",
                        "priority": "<nil>",
                        "queueName": "root.default",
                        "nodeId": "node-0002",
                        "applicationId": "application-0001",
                        "partition": "default"
                    }
                ],
                "schedulable": true
            }
        ]
    }
]
```

## Goroutines信息

转储当前运行的goroutine的堆栈跟踪。

**URL** : `/ws/v1/stack`

**方法** : `GET`

**是否需要认证** : NO

### 成功返回

**返回代码** : `200 OK`

**样例内容**

```text
goroutine 356 [running
]:
github.com/apache/incubator-yunikorn-core/pkg/webservice.getStackInfo.func1(0x30a0060,
0xc003e900e0,
0x2)
	/yunikorn/go/pkg/mod/github.com/apache/incubator-yunikorn-core@v0.0.0-20200717041747-f3e1c760c714/pkg/webservice/handlers.go: 41 +0xab
github.com/apache/incubator-yunikorn-core/pkg/webservice.getStackInfo(0x30a0060,
0xc003e900e0,
0xc00029ba00)
	/yunikorn/go/pkg/mod/github.com/apache/incubator-yunikorn-core@v0.0.0-20200717041747-f3e1c760c714/pkg/webservice/handlers.go: 48 +0x71
net/http.HandlerFunc.ServeHTTP(0x2df0e10,
0x30a0060,
0xc003e900e0,
0xc00029ba00)
	/usr/local/go/src/net/http/server.go: 1995 +0x52
github.com/apache/incubator-yunikorn-core/pkg/webservice.Logger.func1(0x30a0060,
0xc003e900e0,
0xc00029ba00)
	/yunikorn/go/pkg/mod/github.com/apache/incubator-yunikorn-core@v0.0.0-20200717041747-f3e1c760c714/pkg/webservice/webservice.go: 65 +0xd4
net/http.HandlerFunc.ServeHTTP(0xc00003a570,
0x30a0060,
0xc003e900e0,
0xc00029ba00)
	/usr/local/go/src/net/http/server.go: 1995 +0x52
github.com/gorilla/mux.(*Router).ServeHTTP(0xc00029cb40,
0x30a0060,
0xc003e900e0,
0xc0063fee00)
	/yunikorn/go/pkg/mod/github.com/gorilla/mux@v1.7.3/mux.go: 212 +0x140
net/http.serverHandler.ServeHTTP(0xc0000df520,
0x30a0060,
0xc003e900e0,
0xc0063fee00)
	/usr/local/go/src/net/http/server.go: 2774 +0xcf
net/http.(*conn).serve(0xc0000eab40,
0x30a61a0,
0xc003b74000)
	/usr/local/go/src/net/http/server.go: 1878 +0x812
created by net/http.(*Server).Serve
	/usr/local/go/src/net/http/server.go: 2884 +0x4c5

goroutine 1 [chan receive,
	26 minutes
]:
main.main()
	/yunikorn/pkg/shim/main.go: 52 +0x67a

goroutine 19 [syscall,
	26 minutes
]:
os/signal.signal_recv(0x1096f91)
	/usr/local/go/src/runtime/sigqueue.go: 139 +0x9f
os/signal.loop()
	/usr/local/go/src/os/signal/signal_unix.go: 23 +0x30
created by os/signal.init.0
	/usr/local/go/src/os/signal/signal_unix.go: 29 +0x4f

...
```

## 指标

从 Prometheus 服务器检索指标的接口。
这些指标是用帮助消息和类型信息进行转储的。

**URL** : `/ws/v1/metrics`

**方法** : `GET`

**是否需要认证** : NO

### 成功返回

**返回代码** : `200 OK`

**样例内容**

```text
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 2.567e-05
go_gc_duration_seconds{quantile="0.25"} 3.5727e-05
go_gc_duration_seconds{quantile="0.5"} 4.5144e-05
go_gc_duration_seconds{quantile="0.75"} 6.0024e-05
go_gc_duration_seconds{quantile="1"} 0.00022528
go_gc_duration_seconds_sum 0.021561648
go_gc_duration_seconds_count 436
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 82
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.12.17"} 1
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 9.6866248e+07

...

# HELP yunikorn_scheduler_vcore_nodes_usage Nodes resource usage, by resource name.
# TYPE yunikorn_scheduler_vcore_nodes_usage gauge
yunikorn_scheduler_vcore_nodes_usage{range="(10%, 20%]"} 0
yunikorn_scheduler_vcore_nodes_usage{range="(20%,30%]"} 0
yunikorn_scheduler_vcore_nodes_usage{range="(30%,40%]"} 0
yunikorn_scheduler_vcore_nodes_usage{range="(40%,50%]"} 0
yunikorn_scheduler_vcore_nodes_usage{range="(50%,60%]"} 0
yunikorn_scheduler_vcore_nodes_usage{range="(60%,70%]"} 0
yunikorn_scheduler_vcore_nodes_usage{range="(70%,80%]"} 1
yunikorn_scheduler_vcore_nodes_usage{range="(80%,90%]"} 0
yunikorn_scheduler_vcore_nodes_usage{range="(90%,100%]"} 0
yunikorn_scheduler_vcore_nodes_usage{range="[0,10%]"} 0
```

## 配置验证

**URL** : `/ws/v1/validate-conf`

**方法** : `POST`

**是否需要认证** : NO

### 成功返回

不管配置是否允许，如果服务器能够处理请求，它都将产生一个 200 HTTP 的状态码。

**返回代码** : `200 OK`

#### 被允许的配置方法

发送以下简单的配置会产生一个返回


```yaml
partitions:
  - name: default
    queues:
      - name: root
        queues:
          - name: test
```

返回

```json
{
    "allowed": true,
    "reason": ""
}
```

#### 不被允许的配置方法

由于yaml文件中输入了 "wrong_text" 这个错误的字段，因此不允许进行以下配置。

```yaml
partitions:
  - name: default
    queues:
      - name: root
        queues:
          - name: test
  - wrong_text
```

返回

```json
{
    "allowed": false,
    "reason": "yaml: unmarshal errors:\n  line 7: cannot unmarshal !!str `wrong_text` into configs.PartitionConfig"
}
```

## 应用历史

通过时间戳检索关于总应用程序的数量的历史数据的接口。

**URL** : `/ws/v1/history/apps`

**方法** : `GET`

**是否需要认证** : NO

### 成功返回

**返回代码** : `200 OK`

**样例内容**

```json
[
    {
        "timestamp": 1595939966153460000,
        "totalApplications": "1"
    },
    {
        "timestamp": 1595940026152892000,
        "totalApplications": "1"
    },
    {
        "timestamp": 1595940086153799000,
        "totalApplications": "2"
    },
    {
        "timestamp": 1595940146154497000,
        "totalApplications": "2"
    },
    {
        "timestamp": 1595940206155187000,
        "totalApplications": "2"
    }
]
```

## 容器历史

Endpoint to retrieve historical data about the number of total containers by timestamp.
通过时间戳检索有关容器总数的历史数据的接口。

**URL** : `/ws/v1/history/containers`

**方法** : `GET`

**是否需要认证** : NO

### 成功返回

**返回代码** : `200 OK`

**样例内容**

```json
[
    {
        "timestamp": 1595939966153460000,
        "totalContainers": "1"
    },
    {
        "timestamp": 1595940026152892000,
        "totalContainers": "1"
    },
    {
        "timestamp": 1595940086153799000,
        "totalContainers": "3"
    },
    {
        "timestamp": 1595940146154497000,
        "totalContainers": "3"
    },
    {
        "timestamp": 1595940206155187000,
        "totalContainers": "3"
    }
]
```
