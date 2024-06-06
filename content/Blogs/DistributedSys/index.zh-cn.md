---
title: "MIT 6.824 Distributed Systems 分布式系统"
date: 2024-05-11
draft: false
description: "Distributed Systems"
summary: "MapReduce, Threads, RPC, GFS, Raft "
tags: ["Distributed System"]
---
## Lab 1: MapReduce
__code:__ https://github.com/CIRCIRCIRCLE/Distributed_System-6.824/tree/main/mapreduce  
__基本概述:__ 该Lab基于C++实现了基本的MapReduce框架，使用条件变量与互斥锁实现了高效的超时监控机制，通过RPC连接实现了Master与Worker之间的通信。
### Map和Reduce的实现机制
<figure style="text-align:center; ">
  <img src="figs/MapReduce.png" alt="MapReduce" width="800">
  <figcaption>Execution of MapReduce (Image source: Hu et al.)</figcaption>
</figure>

1. Map函数:   
目的: 将输入文件内容拆分成键值对。    
实现: 读取输入文件内容，将其拆分成单词，每个单词作为键，值设为"1"。   
过程:
读取文件内容到一个字符串中 ->  将字符串按单词拆分，生成一系列键值对（每个单词对应一个键值对，值为"1"）-> 返回键值对列表。   

2. Reduce函数:   
目的: 对中间结果进行聚合，计算每个键（单词）出现的总次数。  
实现: 读取中间结果文件，聚合相同键的值。  
过程:
    - Shuffle函数: 读取所有中间结果文件，将相同键的值聚合在一起。
    - ReduceF函数: 计算每个键的总值，并输出最终结果。

### 定时与超时重传的机制
1. 任务分配时启动计时器:每次分配任务时启动一个计时器线程，监控任务的完成情况。
2. 计时器线程等待超时: 使用 pthread_cond_timedwait() 等待指定时间，如果在超时前任务未完成，则继续等待。
3. 任务完成前取消计时器: 任务完成时，Worker会通知Master取消计时器。
4. 任务超时重新分配: 如果任务在超时前未完成，Master会将任务标记为超时并重新分配。    
__Notes:__ pthread_cond_timedwait() 使用操作系统提供的同步原语，可以更高效地等待事件发生或超时，不会占用CPU资源。在等待期间，线程会让出CPU，让其他线程运行，提高系统的整体效率。相比使用 sleep() 函数的忙等待，pthread_cond_timedwait() 可以有效避免CPU资源的浪费。

### RPC原理
1. ButtonRPC库:
    - ButtonRPC是一个轻量级的RPC框架，用于实现远程过程调用。
    - 它允许Master和Worker之间进行通信，以分配任务和报告状态。
2. RPC调用过程:
    - Master作为RPC服务器，定义并绑定各类RPC接口，例如分配任务、更新任务状态等。 
    - Worker作为RPC客户端，连接到Master并调用这些接口获取任务或报告任务完成情况。
    - 使用ButtonRPC库的简洁API进行网络通信和数据传输，使得Master和Worker能够高效地协同工作。