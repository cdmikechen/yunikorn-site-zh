---
id: cross_queue_preemption
title: Cross Queue Preemption
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

## 问题:

根据我们从 YARN Scheduler 抢占中吸取的教训。

**以下是最糟糕的事情:** 

- 抢占是一把猎枪，而不是狙击手，当抢占决定作出时，没有人知道抢占的资源是否会进入请求队列/应用程序/用户。
- 抢占逻辑和分配是不同的，我们必须实现（并模仿）我们在调度器分配逻辑中所做的事情。

**以下是最好的事情:**

- 抢占很快（多亏了猎枪），回收数千个 contaier 并只需要1秒。
- 我们已经了解处理DRF、多个抢占策略（队列间/队列内、散弹枪/外科抢占等）是多么痛苦，并且我们已经开发了一些不错的逻辑来确保更好的模块化和插件能力。

## 回答一些有关设计/实现选择的问题

**1\. 我们真的想要抢占延迟吗？（或者我们只是想控制节奏）**

在CS架构中，我们有抢占延迟，即在抢占候选中选择需要杀死的进程，等待一定时间后再杀死它。

The purposes of preemption delay are: a. give heads-up time to apps so 
they can prepare bad things happen (unfortunately no app do anything for these heads up, at least from what I knew). b. control preemption pace.   

And in practice, I found it causes a lot of issues, for example when a 
cluster state keep changing, it is very hard to ensure accurate preemption. 

**Proposal:**

Remove the preemption-delay, keep the logics of controlling preemption pace. (such as ```yarn.resourcemanager.monitor.capacity.preemption
.total_preemption_per_round```). And we can do allocation together with preemption.
This don't mean containers will be stopped immediately after preemption issued. Instead, RM can control delays between signal a container and kill a container. Such as grace 
termination of POD in K8s: https://kubernetes.io/docs/concepts/workloads/pods/pod/#termination-of-pods   

**2\. Do we want to do preemption for every scheduling logic, or we can do periodically?**

In CS, we have preemption logic runs periodically, like every 1 sec or 3 sec. 

Since preemption logic involves some heavy logics, like calculating shares of queues/apps. And when doing accurate preemption, we may need to scan nodes for preemption candidate. 
Considering this, I propose to have preemption runs periodically. But it is important to note that, we need to try to use as much code as possible for 
allocation-inside-preemption, otherwise there will be too much duplicated logic and very hard to be maintained in the future.

**3\. Preemption cost and function**

We found it is helpful to add cost for preemption, such as container live time, priority, type of container. It could be a cost function (Which returns a numeric value) or it 
could be a comparator (which compare two allocations for preemption ask).

## Pseudo code

Logic of allocation (invoked every allocation cycle)

```
input:
  - nAlloc, allocate N allocations for this allocation cycle.

for partition: 
  askCandidates := findAskCandidates(nAlloc, preemption=false)
  
  allocated, failed_to_allocated := tryAllocate(askCandidates);
  
  send-allocated-to-cache-to-commit;
  
  update-missed-opportunity (allocated, failed_to_allocated);
  
  nAlloc -= len(allocated)   
```

Logic of preemption (invoked every preemption cycle)

```
// It has to be done for every preemption-policy because calculation is different.
for preemption-policy: 
  preempt_results := policy.preempt()
  for preempt_results: 
     send-preempt-result-to-cache-to-commit;
     updated-missed-opportunity (allocated)
```

Inside preemption policy

```
inter-queue-preempt-policy:
  calculate-preemption-quotas;
  
  for partitions:
    total_preempted := resource(0);
    
    while total_preempted < partition-limited:
      // queues will be sorted by allocating - preempting
      // And ignore any key in preemption_mask
      askCandidates := findAskCandidates(N, preemption=true)
      
      preempt_results := tryAllocate(askCandidates, preemption=true);
      
      total_preempted += sigma(preempt_result.allocResource)
      
      send-allocated-to-cache-to-commit;
      
      update-missed-opportunity (allocated, failed_to_allocated);
      
      update-preemption-mask(askCandidates.allocKeys - preempt_results.allocKeys)
```