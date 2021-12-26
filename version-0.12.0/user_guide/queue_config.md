---
id: queue_config
title: 分区和队列配置
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

队列配置的基础在 [配置设计文档](design/scheduler_configuration.md) 中给出。

本文档提供了通用队列配置。
它引用了 [访问控制列表](user_guide/acls.md) 和 [放置规则](user_guide/placement_rules.md) 文档。

本文档通过示例说明了如何为调度器创建分区和队列配置。

作为应用程序提交的一部分，调度器依赖于 shim 来可靠地提供用户信息。
当前 shim 使用 [用户和组解析](usergroup_resolution) 中提供的方法识别用户和用户所属的组。

## 配置
这里描述的调度器的配置文件只提供了分区和队列的配置。

默认情况下，我们在部署中使用名为 `queues.yaml` 的文件。
可以通过调度器的命令行标志 `policyGroup` 更改文件名。
更改文件名后必须在 deployment 详细信息中进行相应的更改，无论是 `configmap` 还是包含在 docker 容器中的文件。

配置的示例文件位于调度器核心内 [queues.yaml](https://github.com/apache/incubator-yunikorn-core/blob/master/config/queues.yaml)。

## 分区
分区是调度器配置的顶层。
配置中可以定义多个分区。

配置中分区定义的基本结构：
```yaml
partitions:
  - name: <name of the 1st partition>
  - name: <name of the 2nd partition>
```
分区的默认名称是 `default`。
分区定义包含特定 shim 的调度程器的完整配置。
每个 shim 使用自己唯一的分区。

分区必须至少定义以下键：
* 名称
* [队列](#queues)

队列配置释义如下。

可以选择为分区定义以下键：
* [布置规则](#placement-rules)
* [限制](#limits)
* 节点排序策略
* 抢占

布置规则和限制在它们自己的章节中进行了详解

`nodesortpolicy` 定义了节点为分区排序的方式。
可以使用的值的详细信息在 [排序策略](sorting_policies.md#node-sorting) 文档中可以参考。

抢占键目前只能有一个子键：_enabled_ 。
这个布尔值定义了整个分区的抢占行为。

_enabled_ 的默认值为 _false_ 。
允许值：_true_ 或 _false_ ，任何其他值都会导致解析错误。

设置了 _preemption_ 标志和 `nodesortpolicy` 为 _fair_ 的示例 `partition` yaml 描述：
```yaml
partitions:
  - name: <分区名称>
    nodesortpolicy: fair
    preemption:
      enabled: true
```
注意:
目前，Kubernetes 唯一的 shim 不支持除 `default` 分区之外的任何其他分区。
这已为 shim 在 [jira](https://issues.apache.org/jira/browse/YUNIKORN-22) 上进行了记录。

### 队列

YuniKorn 通过利用资源队列来管理资源。资源队列具有以下特征：
- 队列可以有 **等级制的** 结构
- 每个队列都可以预设 **最小/最大容量** ，其中最小容量定义了保证资源，最大容量定义了资源限制（又名资源配额）
- 任务必须在某个子队列下运行
- 队列可以是 **静态的**（从配置文件加载）或 **动态的**（由 YuniKorn 内部管理）
- 队列级别 **资源公平** 由调度器强制执行
- 作业只能在特定队列下运行

:::info
YuniKorn 队列与 [Kubernetes 命名空间](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) 的区别：
Kubernetes 命名空间提供了 Kubernetes 资源的范围，包括安全上下文（即谁可以访问对象）和当 [资源配额](https://kubernetes.io/docs/concepts/policy/resource-quotas/) 被定义（即对象可以使用多少资源）时的资源边界。
另一方面，YuniKorn 队列仅用于一组作业可以使用多少资源，以及按什么顺序使用。考虑到资源公平性、作业排序等，它对跨租户的资源共享会提供更细粒度的控制。在大多数情况下，YuniKorn 队列可以用来代替命名空间资源配额，以提供更多的调度功能 .
:::

_queues_ 内容是主要的配置元素。
它定义了队列的层次结构。

It can have a `root` queue defined but it is not a required element.
If the `root` queue is not defined the configuration parsing will insert the root queue for consistency.
The insertion of the root queue is triggered by:
* If the configuration has more than one queue defined at the top level a root queue is inserted.
* If there is only one queue defined at the top level and it is not called `root` a root queue is inserted.  
我们可以定义一个 `root` 队列，但它不是必需的元素。
如果没有定义 `root` 队列，配置解析将插入根队列以保持一致性。
根队列的插入由以下情况触发：
* 如果配置在顶层定义了多个队列，一个根队列会被插入。
* 如果在顶层只定义了一个队列并且它不被称为 `root`，一个根队列会被插入。

定义的队列将成为插入的 `root` 队列的子队列。

带有子队列的基本 `queues` yaml 内容：
```yaml
queues:
- name: <队列名>
  queues:
  - name: <队列名>
```

队列支持的参数：
* name
* parent
* queues
* properties
* adminacl
* submitacl
* [resources](#resources)
* [limits](#limits)

每个队列都必须有一个 _名称_。
队列的名称在定义队列的级别上必须是唯一的。
由于队列结构是完全分层的，层次结构中不同点的队列可能具有相同的名称。
例如：队列结构 `root.testqueue` 和 `root.parent.testqueue` 是一个有效的结构。
队列不能包含点 “.” 字符，因为该字符用于分隔层次结构中的队列。
如果配置中队列的名称不是唯一的或包含点，则会生成解析错误并拒绝配置。

* parent
* leaf
在结构中的队列将自动获得分配的类型。
队列的类型基于队列具有子队列或次级队列的这个事实。
队列的两个类型是：
* parent
* leaf

应用程序只能分配到 _leaf_ 队列。
在配置中具有子队列或次级队列的队列自动成为 _parent_ 队列。
如果队列在配置中没有子队列，则它是 _leaf_ 队列，除非 `parent` 参数设置为 _true_。
尝试覆盖配置中的 _parent_ 队列类型会导致配置解析错误。

父队列的次级队列在 `queues` 层级下定义。
`queues` 层级是队列级别的递归条目，并使用完全相同的参数集。

`properties` 参数是一个简单的键值对列表。
该列表为队列提供了一组简单的参数属性。
键或值的值没有限制，任何配置都是允许的。
目前，属性列表仅用于调度器中，用于为 leaf 队列定义 [排序顺序](sorting_policies.md#application-sorting)。
未来的扩展，比如在队列或其他排序策略上打开或关闭抢占的选项，将使用相同的属性构造，而无需更改配置。

对队列的访问是通过 `adminacl` 设置的，用于管理操作和通过 `submitacl` 条目提交申请。
ACL 记录在 [访问控制列表](acls.md) 文档中。

队列资源限制是通过 `resources` 参数设置的。
用户和组限制是通过 `limits` 参数设置的。
由于这两个条目都是复杂的配置条目，它们在下面的 [资源](#resources) 和 [限制](#limits) 中进行了说明。

下面列举一个具有限制的 _parent_ 队列 `root.namespaces` 的示例配置：
```yaml
partitions:
  - name: default
    queues:
      - name: namespaces
        parent: true
        resources:
          guaranteed:
            {memory: 1000, vcore: 10}
          max:
            {memory: 10000, vcore: 100}
```

### 放置规则

The placement rules are defined and documented in the [placement rule](placement_rules.md) document.

Each partition can have only one set of placement rules defined. 
If no rules are defined the placement manager is not started and each application *must* have a queue set on submit.  

### Limits
Limits define a set of limit objects for a partition or queue.
It can be set on either the partition or on a queue at any level.
```yaml
limits:
  - limit: <description>
  - limit: <description>
```

A limit object is a complex configuration object.
It defines one limit for a set of users and or groups.
Multiple independent limits can be set as part of one `limits` entry on a queue or partition.
Users and or groups that are not part of the limit setting will not be limited.

A sample limits entry:
```yaml
limits:
  - limit: <description>
    users:
    - <user name or "*"">
    - <user name>
    groups:
    - <group name or "*"">
    - <group name>
    maxapplications: <1..maxint>
    maxresources:
      <resource name 1>: <0..maxint>
      <resource name 2>: <0..maxint>
```

Limits are applied recursively in the case of a queue limit.
This means that a limit on the `root` queue is an overall limit in the cluster for the user or group.
A `root` queue limit is thus also equivalent with the `partition` limit.

The limit object parameters:
* limit
* users
* groups
* maxapplications
* maxresources

The _limit_ parameter is an optional description of the limit entry.
It is not used for anything but making the configuration understandable and readable. 

The _users_ and _groups_ that can be configured can be one of two types:
* a star "*" 
* a list of users or groups.

If the entry for users or groups contains more than one (1) entry it is always considered a list of either users or groups.
The star "*" is the wildcard character and matches all users or groups.
Duplicate entries in the lists are ignored and do not cause a parsing error.
Specifying a star beside other list elements is not allowed.

_maxapplications_ is an unsigned integer value, larger than 1, which allows you to limit the number of running applications for the configured user or group.
Specifying a zero maximum applications limit is not allowed as it would implicitly deny access.
Denying access must be handled via the ACL entries.  

The _maxresources_ parameter can be used to specify a limit for one or more resources.
The _maxresources_ uses the same syntax as the [resources](#resources) parameter for the queue. 
Resources that are not specified in the list are not limited.
A resource limit can be set to 0.
This prevents the user or group from requesting the specified resource even though the queue or partition has that specific resource available.  
Specifying an overall resource limit of zero is not allowed.
This means that at least one of the resources specified in the limit must be greater than zero.

If a resource is not available on a queue the maximum resources on a queue definition should be used.
Specifying a limit that is effectively zero, _maxapplications_ is zero and all resource limits are zero, is not allowed and will cause a parsing error.
 
A limit is per user or group. 
It is *not* a combined limit for all the users or groups together.

As an example: 
```yaml
limit: "example entry"
maxapplications: 10
users:
- sue
- bob
```
In this case both the users `sue` and `bob` are allowed to run 10 applications.

### Resources
The resources entry for the queue can set the _guaranteed_ and or _maximum_ resources for a queue.
Resource limits are checked recursively.
The usage of a leaf queue is the sum of all assigned resources for that queue.
The usage of a parent queue is the sum of the usage of all queues, leafs and parents, below the parent queue. 

The root queue, when defined, cannot have any resource limit set.
If the root queue has any limit set a parsing error will occur.
The maximum resource limit for the root queue is automatically equivalent to the cluster size.
There is no guaranteed resource setting for the root queue.

Maximum resources when configured place a hard limit on the size of all allocations that can be assigned to a queue at any point in time.
A maximum resource can be set to 0 which makes the resource not available to the queue.
Guaranteed resources are used in calculating the share of the queue and during allocation. 
It is used as one of the inputs for deciding which queue to give the allocation to.
Preemption uses the _guaranteed_ resource of a queue as a base which a queue cannot go below.

Basic `resources` yaml entry:
```yaml
resources:
  guaranteed:
    <resource name 1>: <0..maxint>
    <resource name 2>: <0..maxint>
  max:
    <resource name 1>: <0..maxint>
    <resource name 2>: <0..maxint>
```
Resources that are not specified in the list are not limited, for max resources, or guaranteed in the case of guaranteed resources. 

### Child Template

The parent queue can provide a template to define the behaviour of dynamic leaf queues below it. A parent queue having no child template inherits the child template
from its parent if that parent has one defined. A child template can be defined at any level in the queue hierarchy on a queue that is of the type parent.

The supported configuration in template are shown below.
1. application sort policy
2. max resources
3. guaranteed resources
4. max applications

As an example:
```yaml
 partitions:
   - name: default
     placementrules:
       - name: provided
         create: true
     queues:
       - name: root
         submitacl: '*'
         childtemplate:
           maxapplications: 10
           properties:
             application.sort.policy: stateaware
           resources:
             guaranteed:
               vcore: 1000
               memory: 1000
             max:
               vcore: 20000
               memory: 600000
         queues:
           - name: parent
             parent: true
             childtemplate:
               resources:
                 max:
                   vcore: 21000
                   memory: 610000
           - name: notemplate
             parent: true
```
In this case, `root.parent.sales` will directly use the child template of parent queue `root.parent`. By contrast,
`root.notemplate.sales` will use the child template set on the queue `root` since its parent queue `root.notemplate` inherits the child template from the queue `root`.

[DEPRECATED] Please migrate to template if your cluster is relying on old behavior that dynamic leaf queue can inherit
`application.sort.policy` from parent (introduced by [YUNIKORN-195](https://issues.apache.org/jira/browse/YUNIKORN-195)). The old behavior will get removed in the future release.
