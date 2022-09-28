---
layout: post
title:  "用实际代码来学习java线程的状态"
date:   2022-09-10 15:28:23 +0800
blurb:  "用实际代码来学习java线程的状态"
og_image:
categories: diy
---

Java中的线程状态主要有以下几个：

| 状态           | 说明           |
| ------------- | ------------- |
| NEW           |               | 
| RUNNABLE      |               |
| BLOCKED       |               |
| TIMED_WAITING |               |
| WAITING       |               |
| TERMINATED    |               |




```mermaid
stateDiagram
    NEW
    RUNNABLE
    WAITING
    BLOCKED
    TIMED_WAITING
    WAITING
    TERMINATED

    [*] --> NEW: new Thread

    NEW --> RUNNABLE: start()

    RUNNABLE --> WAITING: Object.wait()\nThread.join()\nLockSupport.park()

    RUNNABLE --> TIMED_WAITING: Thread.sleep\nObject.wait(long millis)\nThread.join(long millis)\nLockSupport.parkNanos\parkUntil()

    WAITING --> RUNNABLE: Object.notify()\nObject.notifyAll()\nLockSupport.unpark\ninterrupt（）

    TIMED_WAITING --> RUNNABLE: 同WAITING --> RUNNABLE

    RUNNABLE --> BLOCKED: synchronized block/method

    RUNNABLE --> TERMINATED: 执行完run()

```