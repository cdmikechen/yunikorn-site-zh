---
id: cache_removal
title: 调度器缓存删除设计
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

# 在内核中结合缓存和调度器实现的建议
本文描述调度器和缓存实现的当前状态。
它描述了基于对当前行为所做的分析而计划的更改。

## 目标
本目标是在变更前后提供相同的功能。
- 合并前后的单元测试必须全部通过。
- 在内核定义的冒烟测试应全部通过并没有重大变化 <sup id="s1">[定义](#f1)</sup>.
- 作为shim代码一部分的端到端测试必须全部通过而不发生更改。

## 背景 
当前的调度器核心是围绕用于存储数据的两个主要组件构建的：缓存和调度器对象。
缓存对象构成了要跟踪的大多数数据的基础。
调度器对象跟踪特定于飞行(flight)细节并在缓存对象之上构建。

两层之间的通信使用一个同步事件，在某些情况下还使用直接更新。
调度器和缓存之间的同步更新确实意味着调度器与缓存之间有一段短时间的“不同步”。
这个短周期可对调度决策产生影响。
其中一个影响被记录在 [YUNIKORN-169](https://issues.apache.org/jira/browse/YUNIKORN-169).

另外一点是这两种结构给代码带来的复杂性。
在调度器和缓存之间包含一组不同的通信消息。
调度器和缓存对象之间的一对一映射表明，这种区别可能更需要人为控制这种依赖。
---
<b id="f1"></b>定义：冒烟测试的主要变化被定义为，在测试上改变用例和因而改变的变化。由于所做的检查可能依赖于已删除的缓存对象，因此需要进行一些更改。[↩](#s1)
## 结构分析
### 对象
按照代码分析现有的对象。
调度器和缓存对象之间的重叠部分通过在同一行显示它们来展现。
N/A 表示调度器或缓存中没有等效对象。

| 缓存对象                        | 调度器对象                       |
| ------------------------------ | ------------------------------ |
| ClusterInfo                    | ClusterSchedulingContext       |
| PartitionInfo                  | partitionSchedulingContext     |
| AllocationInfo                 | schedulingAllocation           |
| N/A                            | schedulingAllocationAsk        |
| N/A                            | reservation                    |
| ApplicationInfo                | SchedulingApplication          |
| applicationState               | N/A                            |
| NodeInfo                       | SchedulingNode                 |
| QueueInfo                      | SchedulingQueue                |
| SchedulingObjectState          | N/A                            |

作为缓存的一部分的 `initializer` 代码不定义特定的对象。
它含有一个包级别定义的混合体代码和作为 `ClusterInfo` 对象一部分的代码。

### 事件
在内核定义的事件里有多个来源和目标。
某些事件只在缓存和调度器之间的核心里。
这些事件将被删除。

| 事件                                       | 流程                  | 建议      |
| ----------------------------------------- | --------------------- | -------- |
| AllocationProposalBundleEvent             | Scheduler -> Cache    | Remove   |
| RejectedNewApplicationEvent               | Scheduler -> Cache    | Remove   |
| ReleaseAllocationsEvent                   | Scheduler -> Cache    | Remove   |
| RemoveRMPartitionsEvent                   | Scheduler -> Cache    | Remove   |
| RemovedApplicationEvent                   | Scheduler -> Cache    | Remove   |
| SchedulerNodeEvent                        | Cache -> Scheduler    | Remove   |
| SchedulerAllocationUpdatesEvent           | Cache -> Scheduler    | Remove   |
| SchedulerApplicationsUpdateEvent          | Cache -> Scheduler    | Remove   |
| SchedulerUpdatePartitionsConfigEvent      | Cache -> Scheduler    | Remove   |
| SchedulerDeletePartitionsConfigEvent      | Cache -> Scheduler    | Remove   |
| RMApplicationUpdateEvent (add/remove app) | Cache/Scheduler -> RM | Modify   |
| RMRejectedAllocationAskEvent              | Cache/Scheduler -> RM | Modify   |
| RemoveRMPartitionsEvent                   | RM -> Scheduler       |          |
| RMUpdateRequestEvent                      | RM -> Cache           | Modify   |
| RegisterRMEvent                           | RM -> Cache           | Modify   |
| ConfigUpdateRMEvent                       | RM -> Cache           | Modify   |
| RMNewAllocationsEvent                     | Cache -> RM           | Modify   |
| RMReleaseAllocationEvent                  | Cache -> RM           | Modify   |
| RMNodeUpdateEvent                         | Cache -> RM           | Modify   |
|                                           |                       |          |

移除缓存后，由缓存处理的事件将需要由核心代码处理。
两个事件由缓存和调度器处理。

## 详细流程分析
### 缓存和调度程序中同时存在的对象
当前的设计基于这样一个事实：缓存对象是所有数据存储的基础。
每个缓存对象必须有一个对应的调度器对象。
核心程序内围绕缓存和调度程序对象的协议很简单。
如果该对象同时存在于调度器和缓存中，则该对象将被添加到缓存中，从而触发相应调度器对象的创建。
删除对象总是以相反的方式处理：首先从调度器中删除，这将触发从缓存中删除的方法。
例如，由 `RMUpdateRequestEvent` 触发的应用程序的创建将由缓存处理。
Creating a `SchedulerApplicationsUpdateEvent` to create the corresponding application in the scheduler.
创建 `SchedulerApplicationsUpdateEvent` 会在调度器中创建相应的应用。

当添加应用程序和对象状态时，它们也会被添加到缓存对象中。
缓存对象被认为是数据存储，因此也包含了状态。
调度器中没有相应的状态对象。
不可能为同一对象维护两种状态。

该规则的其他例外是被认为遗失的对象和只有调度器的对象。
`schedulingAllocationAsk` 跟踪调度器中应用未完成的请求。
`reservation` 跟踪一个节点对应用和请求组合的临时保留。


### 添加/删除应用的操作
RM（shim）发送在调度器接口中定义的复合 `更新请求` 。
此消息由RM代理封装并转发到缓存进行处理。
RM可以请求添加或删除应用。

**应用添加或删除**
```
1. RM代理将 cacheevent.RMUpdateRequestEvent 发送到缓存
2. cluster_info.processApplicationUpdateFromRMUpdate
   2.1: 将新应用添加到分区。
   2.2: 将删除的应用发送到调度器（但不从缓存中删除任何内容）
3. scheduler.processApplicationUpdateEvent
   3.1: 将新应用添加到调度器
        (当失败时候, 发送 RejectedNewApplicationEvent 到缓存)
        无论是否失败，都将R MApplicationUpdateEvent 发送到 RM。
   3.2: 从调度器删除应用
        发送 RemovedApplicationEvent 到缓存
```

### 删除应用和添加或删除请求的操作
RM（shim）发送在调度器接口中定义的复合 `更新请求` 。
此消息由RM代理封装并转发到缓存进行处理。
RM可以请求删除应用。
RM可以请求添加或删除请求。

**allocation delete**
This describes the allocation delete initiated by the RM only
````
1. RMProxy sends cacheevent.RMUpdateRequestEvent to cache
2. cluster_info.processNewAndReleaseAllocationRequests
   2.1: (by-pass): Send to scheduler via event SchedulerAllocationUpdatesEvent
3. scheduler.processAllocationUpdateEvent 
   3.1: Update ReconcilePlugin
   3.2: Send confirmation of the releases back to Cache via event ReleaseAllocationsEvent
4. cluster_info.processAllocationReleases to process the confirmed release
````

**ask add**
If the ask already exists this add is automatically converted into an update.
```
1. RMProxy sends cacheevent.RMUpdateRequestEvent to cache
2. cluster_info.processNewAndReleaseAllocationRequests
   2.1: Ask sanity check (such as existence of partition/app), rejections are send back to the RM via RMRejectedAllocationAskEvent
   2.2: pass checked asks to scheduler via SchedulerAllocationUpdatesEvent
3. scheduler.processAllocationUpdateEvent
   3.1: Update scheduling application with the new or updated ask. 
   3.2: rejections are send back to the RM via RMRejectedAllocationAskEvent 
   3.3: accepted asks are not confirmed to RM or cache
```

**ask delete**
```
1. RMProxy sends cacheevent.RMUpdateRequestEvent to cache
2. cluster_info.processNewAndReleaseAllocationRequests
   2.1: (by-pass): Send to scheduler via event SchedulerAllocationUpdatesEvent
3. scheduler.processAllocationReleaseByAllocationKey
   3.1: Update scheduling application and remove the ask. 
```

### Operations to add, update or remove nodes
The RM (shim) sends a complex `UpdateRequest` as defined in the scheduler interface.
This message is wrapped by the RM proxy and forwarded to the cache for processing.
The RM can request a node to be added, updated or removed.

**node add** 
```
1. RMProxy sends cacheevent.RMUpdateRequestEvent to cache
2. cluster_info.processNewSchedulableNodes
   2.1: node sanity check (such as existence of partition/node)
   2.2: Add new nodes to the partition.
   2.3: notify scheduler of new node via SchedulerNodeEvent
3. notify RM of node additions and rejections via RMNodeUpdateEvent
   3.1: notify the scheduler of allocations to recover via SchedulerAllocationUpdatesEvent
4. scheduler.processAllocationUpdateEvent
   4.1: scheduler creates a new ask based on the Allocation to recover 
   4.2: recover the allocation on the new node using a special process
   4.3: confirm the allocation in the scheduler, on failure update the cache with a ReleaseAllocationsEvent
```

**node update and removal**
```
1. RMProxy sends cacheevent.RMUpdateRequestEvent to cache
2. cluster_info.processNodeActions
   2.1: node sanity check (such as existence of partition/node)
   2.2: Node info update (resource change)
        2.2.1: update node in cache
        2.2.2: notify scheduler of the node update via SchedulerNodeEvent
   2.3: Node status update (not removal), update node status in cache only
   2.4: Node removal
        2.4.1: update node status and remove node from the cache
        2.4.2: remove alloations and inform RM via RMReleaseAllocationEvent
        2.4.3: notify scheduler of the node removal via SchedulerNodeEvent
3. scheduler.processNodeEvent add/remove/update the node  
```

### Operations to add, update or remove partitions
**Add RM**
```
1. RMProxy sends commonevents.RemoveRMPartitionsEvent
   if RM is already registered
   1.1: scheduler.removePartitionsBelongToRM
        1.1.1: scheduler cleans up
        1.1.2: scheduler sends commonevents.RemoveRMPartitionsEvent
   1.2: cluster_info.processRemoveRMPartitionsEvent
        1.2.1: cache cleans up
2. RMProxy sends commonevents.RegisterRMEvent
3. cluster_info.processRMRegistrationEvent
   2.1: cache update internal partitions/queues accordingly.
   2.2: cache sends to scheduler SchedulerUpdatePartitionsConfigEvent.
3. scheduler.processUpdatePartitionConfigsEvent
   3.1: Scheduler update partition/queue info accordingly.
```

**Update and Remove partition**
Triggered by a configuration file update.
```
1. RMProxy sends commonevents.ConfigUpdateRMEvent
2. cluster_info.processRMConfigUpdateEvent
   2.1: cache update internal partitions/queues accordingly.
   2.2: cache sends to scheduler SchedulerUpdatePartitionsConfigEvent.
   2.3: cache marks partitions for deletion (not removed yet).
   2.4: cache sends to scheduler SchedulerDeletePartitionsConfigEvent
3. scheduler.processUpdatePartitionConfigsEvent
   3.1: scheduler updates internal partitions/queues accordingly.
4. scheduler.processDeletePartitionConfigsEvent
   4.1: Scheduler set partitionManager.stop = true.
   4.2: PartitionManager removes queues, applications, nodes async.
        This is the REAL CLEANUP including the cache
```

### Allocations
Allocations are initiated by the scheduling process.
The scheduler creates a SchedulingAllocation on the scheduler side which then gets wrapped in an AllocationProposal.
The scheduler has checked resources etc already and marked the allocation as inflight.
This description picks up at the point the allocation will be confirmed and finalised.

**New allocation**
```
1. Scheduler wraps an SchedulingAllocation in an AllocationProposalBundleEvent 
2. cluster_info.processAllocationProposalEvent
   preemption case: release preempted allocations
   2.1: release the allocation in the cache
   2.2: inform the scheduler the allocation is released via SchedulerNodeEvent
   2.3: inform the RM the allocation is released via RMReleaseAllocationEvent
   all cases: add the new allocation
   2.4: add the new allocation to the cache
   2.5: rejections are send back to the scheduler via SchedulerAllocationUpdatesEvent 
   2.6: inform the scheduler the allocation is added via SchedulerAllocationUpdatesEvent
   2.7: inform the RM the allocation is added via RMNewAllocationsEvent
3. scheduler.processAllocationUpdateEvent
   3.1: confirmations are added to the scheduler and change from inflight to confirmed.
        On failure of processing a ReleaseAllocationsEvent is send to the cache *again* to clean up.
        This is part of the issue in [YUNIKORN-169]
        cluster_info.processAllocationReleases
   3.2: rejections remove the inflight allocation from the scheduler. 
```

## Current locking
**Cluster Lock:**  
A cluster contains one or more Partition objects. A partition is a sub object of Cluster.  
Adding or Removing ANY Partition requires a write-lock of the cluster.
Retrieving any object within the cluster will require iterating over the Partition list and thus a read-lock of the cluster

**Partition Lock:**  
The partition object contains all links to Queue, Application or Node objects.
Adding or Removing ANY Queue, Application or Node needs a write-lock of the partition.
Retrieving any object within the partition will require a read-lock of the partition to prevent data races

Examples of operation needing a write-lock
- Allocation processing after scheduling, will change application, queue and node objects.
  Partition lock is required due to possible updates to reservations.
- Update of Node Resource 
  It not only affect node's available resource, it also affects the Partition's total allocatable Resource 

Example of operations that need a read-lock:
- Retrieving any Queue, Application or Node needs a read-lock
  The object itself is not locked as part of the retrieval
- Confirming an allocation after processing in the cache
  The partition is only locked for reading to allow retrieval of the objects that will be changed.
  The changes are made on the underlying objects.

Example of operations that do not need any lock: 
- Scheduling  
  Locks are taken on the specific objects when needed, no direct updates to the partition until the allocation is confirmed. 

**Queue lock:**  
A queue can track either applications (leaf type) or other queues (parent type).
Resources are tracked for both types in the same way.

Adding or removing an Application (leaf type), or a direct child queue (parent type) requires a write-lock of the queue.  
Updating tracked resources requires a write-lock.
Changes are made recursively never locking more than 1 queue at a time.  
Updating any configuration property on the queue requires a write-lock.
Retrieving any configuration value, or tracked resource, application or queue requires a read-lock.  

Examples of operation needing a write-lock
- Adding an application to a leaf queue
- Updating the reservations

Examples of operation needing a read-lock
- Retrieving an application from a leaf type queue
- Retrieving the pending resources 

**Application lock:**  
An application tracks resources of different types, the allocations and outstanding requests.  
Updating any tracked resources, allocations or requests requires a write-lock.
Retrieving any of those values requires a read-lock.

Scheduling also requires a write-lock of the application.
During scheduling the write-lock is held for the application.
Locks will be taken on the node or queue that need to be accessed or updated.  
Examples of the locks taken on other objects are:
- a read lock to access queue tracked resources
- a write-lock to update the in progress allocations on the node 

Examples of operation needing a write-lock
- Adding a new ask
- Trying to schedule a pending request 

Examples of operation needing a read-lock
- Retrieving the allocated resources
- Retrieving the pending requests

**Node lock:**  
An node tracks resources of different types and allocations.
Updating any tracked resources or allocations requires a write-lock.
Retrieving any of those values requires a read-lock.

Checks run during the allocation phases take locks as required.
Read-locks when checking write-locks when updating.
A node is not locked for the whole allocation cycle.

Examples of operation needing a write-lock
- Adding a new allocation
- updating the node resources

Examples of operation needing a read-lock
- Retrieving the allocated resources
- Retrieving the reservation status

## How to merge Cache and scheduler objects
Since there is no longer the requirement to distinguish the objects in the cache and scheduler the `scheduling` and `info` parts of the name will be dropped.

Overview of the main moves and merges:
1. `application_info` & `scheduling_application`: **merge** to `scheduler.object.application`
2. `allocation_info` & `scheduling_allocation`: **merge** to `scheduler.object.allocation`
3. `node_info` & `scheduling_node`: **merge** to `scheduler.object.node`
4. `queue_info` & `scheduling_queue`: **merge** to `scheduler.object.queue`
5. `partition_info` & `scheduling_partition`: **merge** to `scheduler.PartitionContext`
6. `cluster_info` & `scheduling_context`: **merge** to `scheduler.ClusterContext`
7. `application_state`: **move** to `scheduler.object.applicationState`
8. `object_state`: **move** to `scheduler.object.objectState`
9. `initializer`: **merge** into `scheduler.ClusterContext`

This move and merge of code includes a refactor of the objects into their own package.
That thus affects the two scheduler only objects, reservations and schedulingAsk, that are already defined.
Both will be moved into the objects package.

The top level scheduler package remains for the contexts and scheduling code.

## Code merges
The first change is the event processing.
All RM events will now directly be handled in the scheduler.
Event handling will undergo a major change, far more than a simple merge.
Only the RM generated events will be left after the merge.
As described in the analysis above the scheduler is, in almost all cases, notified of changes from RM events.

Broadly speaking there are only three types of changes triggered by the event removal: 
- configuration changes: new scheduler code required as the cache handling is not transferable to the scheduler
- node, ask and application changes: merge of the cache code into the scheduler
- allocation changes: removal of confirmation cycle and simplification of the scheduler code

Part of the event handling is the processing of the configuration changes.
All configuration changes will now update the scheduler objects directly.
The way the scheduler works is slightly different from the cache which means the code is not transferable. 

Nodes and applications are really split between the cache and scheduler.
Anything that is tracked in the cache object that does not have an equivalent value in the scheduler object will be moved into the scheduler object.
All references to scheduler objects will be removed.
With the code merges existing scheduler code that calls out directly into the cache objects will return the newly tracked value in the scheduler object.
These calls will thus become locked calls in the scheduler.

The concept of an in flight allocation will be removed.
Allocation will be made in the same scheduling iteration without events or creation of a proposal.
Removing the need for tracking of allocating resources on the scheduler objects.
In flight resource tracking was required to make sure that an allocation while not confirmed by the cache would being taken into account while making scheduling decisions.

The application and object state will be an integrated part of the scheduler object.
A state change is thus immediate and this should prevent an issue like [YUNIKORN-169](https://issues.apache.org/jira/browse/YUNIKORN-169) from occuring.

## Locking after merge

### Direction of lock 
It is possible to acquire another lock while holding a lock, but we need to make sure that we do not allow: 
- Holding A.lock and acquire B's lock. 
- Holding B.lock and acquire B's lock. 

The current code in the scheduler takes a lock as late as possible and only for the time period needed.
Some actions are not locked on the scheduler side just on the cache side as each object has its own lock.
This means that a read of a value from the cache would not lock the scheduling object.

With the integration of the cache into the scheduler the number of locks will decrease as the number of objects decreases.
Each equivalent object, cache and scheduler, which used to have their own lock will now have just one.
After the merge of the code is performed one lock will be left.
Locking will occur more frequently as the number of fields in the scheduler objects has increased.

Calls that did not lock the scheduler object before the merge will become locked.
Lock contention could lead to performance degradation.
The reduced overhead in objects and event handling can hopefully compensate for this.
One point to keep track of is the change in locking behaviour.
New behaviour could lead to new deadlock situations when code is simply merged without looking at the order.

### Mitigations for deadlocks
The locking inside the scheduler will be left as is.
This means that the main scheduling logic will be taking and releasing locks as required on the objects.
There are no long held read-locks or write-locks until the application is locked to schedule it.

A major point of attention will need to be that no iterations of objects should be performed while holding on to a lock.
For instance during scheduling while iterating over a queue's application we should not lock the queue.

Another example would be that event processing in the partition should not lock the partition unneeded.
The partition should be locked while retrieving for instance the node that needs updating and release the lock before it tries to lock the node itself.

This approach fits in with the current locking approach and will keep the locking changes to a minimum.
Testing, specifically end-to-end testing, should catch these deadlocks. 
There are no known tools that could be used to detect or describe lock order.
