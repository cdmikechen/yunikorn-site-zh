---
id: sorting_policies
title: 排序策略
---

<!--
 * Licensed to the Apache Software Foundation (ASF) under one
 * or more contributor license agreements.  See the NOTICE file
 * distributed with this work for additional information
 * regarding copyright ownership.  The ASF licenses this file
 * to you under the Apache License, Version 2.0 (the
 * "License"); you may not use this file except in compliance
 * with the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 -->

调度器使用多种策略来允许在不更改代码的情况下更改调度行为。
可以为以下各项设置策略：
* [应用程序](#应用排序)
* [节点](#节点排序)
* [请求](#请求排序)

## 应用排序
通过配置为每个队列设置应用程序的排序策略。
排序策略设置仅对 `叶子` 队列有效。
每个 `叶子` 队列可以使用不同的策略。

排序策略仅指定应用程序在队列中的排序顺序。
该顺序对于指定在分配资源时首先考虑哪个应用程序至关重要。
排序策略 _不_ 影响队列中同时调度或活动的应用程序的数量。
除非指定需要过滤掉，否则所有包含待处理资源请求的应用程序都可以并且将被安排在一个队列中。
即使使用先进先出策略对应用程序进行排序，多个应用程序也会在队列中并行运行。

`父` 队列将始终使用公平策略对子队列进行排序。

以下配置信息将队列 `root.sandbox` 的应用程序排序策略设置为 `fifo` ：
```yaml
partitions:
  - name: default
    queues:
    - name: root
      queues:
      - name: sandbox
        properties:
          application.sort.policy: fifo
```

在调度期间唯一考虑的是应用程序必须有未完成的请求。
应用过滤器 _同时_ 对应用程序进行排序，用以删除所有清空未完成请求的应用程序。

### Fifo 排序策略
简短述：first in first out（fifo），先进先出，基于应用创建时间
配置值：fifo（默认）
行为：
在排序之前，应用程序会进行过滤并且必须有待处理的资源请求。

过滤后剩下的应用程序仅根据应用程序创建时间戳进行排序，不应用其他过滤。
由于只能在系统锁定时添加应用程序，因此不可能有两个应用程序具有完全相同的时间戳。

执行的结果是请求资源的最老的应用程序获得资源。
当老的应用程序的所有当前请求都得到满足时，新应用程序将获得资源。

### Fair 排序策略
简述：基于使用的公平策略
配置值：fair
行为：
在排序之前，应用程序会进行过滤并且必须有待处理的资源请求。

过滤后留下的应用程序将根据应用程序使用情况进行排序。
应用程序的使用被定义为应用程序的所有已确认和未确认的资源分配。
在计算使用量时，将考虑应用程序上定义的所有资源。

执行的结果是可用资源平均分布在所有请求资源的应用程序中。

### StateAware 策略
简述：最多一个应用程序处于启动或接受状态的限制
配置值：stateaware
行为：
此排序策略需要了解应用程序状态。
应用程序状态在 [应用状态](design/scheduler_object_states.md#应用状态) 文档中进行了描述。

在对应用程序进行排序之前，以下过滤器将应用于队列中的所有应用程序：
第一个过滤器基于应用程序状态。
以下应用程序会通过过滤器的过滤并生成第一个中间列表：
* 处于 _running_ 状态的所有应用程序
* _一个_ 处于 _starting_ 状态的应用程序
* 如果有 _没有_ 个应用程序处于 _starting_ 状态，则添加 _一个_ 处于 _accepted_ 状态的应用程序

第二个过滤器将第一个过滤器的结果作为输入。
再次过滤初步列表：删除所有 _没有_ 待处理请求的应用程序。

在基于状态和 pending 请求进行过滤后，剩余的应用程序将被排序。
因此，最终列表被过滤两次，其余应用程序在创建时排序。

回顾 _staring_ 和 _accepted_ 状态交互：
只有在没有处于 _starting_ 状态的应用程序时，才会添加处于 _accepted_ 状态的应用程序。
处于 _starting_ 状态的应用程序不必须有待处理的请求。
任何处于 _starting_ 状态的应用程序都将阻止 _accepted_ 应用程序被添加到过滤列表中。

相关的关详细信息，请参阅设计文档中的 [示例运行](design/state_aware_scheduling.md#示例运行)。

执行的结果是已经运行资源请求的应用程序将首先获得资源。
一个新应用程序会被添加到正在运行的应用程序列表中，以便在所有正在运行的应用程序之后分配。  

## 节点排序
通过配置为分区设置节点排序策略。
每个分区可以使用不同的策略。

以下配置信息将分区 `default` 的节点排序策略设置为 `fair` ：
```yaml
partitions:
  - name: default
    nodesortpolicy:
        type: fair
```

### Fairness 策略
简述：可用资源，降序
配置值：fair（默认）
行为：
按可用资源量对节点列表进行排序，以使具有 _最高_ 可用资源量的节点成为列表中的第一个。
在计算使用量时，将考虑节点上定义的所有资源。
为节点比较相同类型的资源。

这导致会在利用率最低的节点首先考虑用使用新的资源分配。
资源分配会分布在所有可用节点上。
除非整个环境的利用率很高，否则会导致单个可用节点的整体利用率较低。
将所有节点上的负载保持在相似水平确实有帮助。
在通过添加新节点自动扩展的环境中，这可能会触发意外的自动扩展请求。

### BinPacking 策略
简述：可用资源，升序
配置值：binpacking
行为：
按可用资源量对节点列表进行排序，以便具有 _最小_ 可用资源量的节点是列表中的第一个。
在计算使用量时，将考虑节点上定义的所有资源。
为节点比较相同类型的资源。

这导致进行新的资源分配时首先考虑具有最高利用率的节点。
从而实现（更）少量节点的（更）高利用率，更适合于云部署。

## 资源权重
节点排序策略可以使用节点的利用率来确定排序。因为节点可以有几种独特的资源类型，所以节点的利用率由其各个资源类型的加权平均值决定。可以使用 `nodesortpolicy` 的 `resourceweights` 部分自定义资源权重。如果 `resourceweights` 不存在或为空，默认配置将 `vcore` 和 `memory` 的权重设置为 `1.0`。所有其他资源类型都将被忽略。只有明确提到的资源类型才会有权重。

YuniKorn 在内部以 `vcore` 资源类型跟踪 CPU 资源。这映射到 Kubernetes 的 `cpu` 资源类型 。
所有其他资源类型在 YuniKorn 和 Kubernetes 之间具有一致的命名。

例如，在默认配置中，如果一个节点分配了 90% 的 CPU 和 50% 的内存，则该节点将被视为 `70%` 的利用率。

以下配置信息将分区 `default` 的 `vcore` 权重设置为 `4.0`，将 `memory` 设置为 `1.0`。
这将使 CPU 使用率比内存使用率高四倍：
```yaml
partitions:
  - name: default
    nodesortpolicy:
      type: fair
      resourceweights:
        vcore: 4.0
        memory: 1.0
```

使用此配置，在此示例中，如果一个节点分配了 `90%` 的 CPU 和 `50%` 的内存，则该节点将被认为是 `82%` 的利用率。

请注意，权重是相互关联的，因此指定 `{ 4.0, 1.0 }` 的权重等效于 `{ 1.0, 0.25 }`。 不允许负权重。

## 请求排序
当前有一种用于对应用程序中的请求进行排序的策略。
但是此策略不可配置。只能根据请求的优先级对请求进行排序。
如果应用程序中有多个具有相同优先级的请求，则请求的顺序是不确定的。
这意味着具有相同优先级的请求的顺序可以并且很可能会在运行之间发生变化。
