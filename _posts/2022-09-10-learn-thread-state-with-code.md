---
layout: post
title:  "通过实际代码理解Java线程的状态"
date:   2022-09-10 15:28:23 +0800
blurb:  "通过实际代码理解Java线程的状态"
og_image:
mermaid: true
categories: diy
---

### 线程状态介绍

Java中的线程状态主要有以下几个：
- NEW
- RUNNABLE
- BLOCKED
- TIMED_WAITING
- WAITING
- TERMINATED


### 状态转移图

按照jdk-docs上不同状态的说明，可以总结为下面的状态转移图：

```mermaid
%%{init: { "theme": "default", "fontFamily": "Trebuchet MS, Verdana, Arial, Sans-Serif" } }%%
stateDiagram
    direction LR
    NEW
    RUNNABLE
    WAITING
    BLOCKED
    TIMED_WAITING
    WAITING
    TERMINATED

    [*] --> NEW: new Thread
    NEW --> RUNNABLE: start()
    
    RUNNABLE --> WAITING: Object.wait(),Thread.join()\nLockSupport.park()

    RUNNABLE --> BLOCKED: synchronized block/method

    BLOCKED --> RUNNABLE: 获取到资源锁    

    RUNNABLE --> TERMINATED: 执行完run()
    
    RUNNABLE --> TIMED_WAITING: Thread.sleep(),Object.wait(long millis),Thread.join(long millis)\nLockSupport.parkNanos,parkUntil()

    WAITING --> RUNNABLE: Object.notify(),Object.notifyAll()\nLockSupport.unpark,interrupt()

    TIMED_WAITING --> RUNNABLE: 同WAITING --> RUNNABLE
```

### 状态转移demo

demo方案
- 1.在主线程中启动子线程，子线程用全局锁lck来与主线程交互
- 2.使用BufferedReader使线程挂起，用于观察线程状态
- 3.使用JDK自带的jconsole来查看线程当前的状态

验证代码

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;

public class ThreadDemo1 {
    
    public static Object lck = new Object();

    public static void main(String[] args) throws Exception {
        Thread t1 = new Thread(new Runnable(){
            @Override
            public void run() {
                synchronized(lck) {
                    System.out.println("t1 thread got lck");
                    try { new BufferedReader(new InputStreamReader(System.in)).readLine();} catch (Exception e) {}
                }

                System.out.println("t1 thread ready to notifyAll");
                try { new BufferedReader(new InputStreamReader(System.in)).readLine();} catch (Exception e) {}
 
                synchronized(lck) {
                    lck.notifyAll();
                }
                
                System.out.println("t1 thread release lck");
            }
        });
        t1.start();

        Thread.sleep(2000L);

        System.out.println("main thread try to get lck");
        synchronized(lck) {
            System.out.println("main thread ready to wait lck");
            lck.wait();
            System.out.println("main thread got lck");
            try { new BufferedReader(new InputStreamReader(System.in)).readLine();} catch (Exception e) {}
        }

        System.out.println("end");
    }
}
```