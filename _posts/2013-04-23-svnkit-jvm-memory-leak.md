---
layout: post
title:  "一次使用第三方SVN开源API引起的JVM内存泄漏问题"
date:   2014-06-03 16:55:09 +0800
blurb:  "前段时间发现我负责的版本发布系统出现了一次OutOfMemoryError，很奇怪这个系统日常访问量不大"
og_image:
categories: 
---

前段时间发现我负责的版本发布系统出现了一次OutOfMemoryError，很奇怪这个系统日常访问量不大，而且最大heap设置为1G以上，当时没有重视，只是简单重启了应用，一切又正常了。

过了27天后，偶然发现JVM的old gen已经占用了700MB，当时系统压力很低，顿时觉得事情不妙，一定有内存泄漏了。立即用jmap导出了当时Heap的快照，用mat分析后发现确实有明显的内存泄漏。发现导致内存占用过多的对象是TimerTask的一个子类TimeoutTask，一共有14013个实例，占用了650MB的heap空间，这个类来自SVNKit。由于版本发布系统需要自动从SVN上checkout代码，然后编译打包，所以使用了SVNKit的API(svnkit-1.3.5)。后来下载了源代码发现TimeoutTask应该是负责检测与SVN链接是否timeout的一个task，但很悲催的发现它竟然会不停的把自己加入到java.util.Timer的调度队列中：

```java
private class TimeoutTask extends TimerTask {
    public void run() {
        boolean scheduled = false;
        try {
            long currentTime = System.currentTimeMillis();
            synchronized (myInactiveRepositories) {
                for (Iterator repositories = myInactiveRepositories.keySet().iterator(); repositories.hasNext();) {
                    SVNRepository repos = (SVNRepository) repositories.next();
                    long time = ((Long) myInactiveRepositories.get(repos)).longValue();
                    if (currentTime - time >= getTimeout()) {
                        repositories.remove();
                        repos.closeSession();
                    }
                }
                if (myTimer != null) {
                    myTimer.schedule(new TimeoutTask(), 10000);
                    scheduled = true;
                }
            }
        } catch (Throwable th) {
            SVNDebugLog.getDefaultLog().logSevere(SVNLogType.WC, th);
            if (!scheduled && myTimer != null) {
                myTimer.schedule(new TimeoutTask(), 10000);
                scheduled = true;
            }
        }
    }
}
```

SVNKit为什么要这么设计呢？实际上是为了保持客户端与SVN服务器的链接打开，每10秒一次检测。貌似没有问题，但我们创建SVN客户端时用的是下面的方法：
```java
SVNClientManager client = SVNClientManager.newInstance()
```

这个模式会每次都创建一个TimeoutTask实例，而TimeoutTask的执行结束前每次又都会创建一个TimeoutTask实例，10秒后新创建的TimeoutTask会再次执行相同的动作。也就是说我创建过多少SVNClientManager实例就会有多少个TimeoutTask实例存在，而且永远不会被垃圾回收掉，因为这些TimeoutTask都被加入到了一个全局的Timer(DefaultSVNRepositoryPool中定义的一个私有静态Timer)对象的调度队列中，从而导致JVM的Old Gen会缓慢增长。

有两种解决方法，一个是SVNClientManager只创建一次；另一个是SVNClientManager使用完了以后马上执行dispose方法就会释放掉那些TimeoutTask。

从这个问题我也有一些自己的感受：
- 1.第三方API有的虽然功能很强大，但在使用时务必要仔细参考其说明文档，甚至是源代码，否则出了问题会比较棘手。
- 2.遇到一个问题一定要有解决到底的精神，如果只是一知半解不想把问题搞清楚，那么永远只能知道些皮毛。

我有时经常听到有人在讨论C++快还是Java快这种无聊的问题，我想说的是作为合格的程序员，你所关心的不应当是如何使用那些外表华丽的框架，你应当能了解他们的实现原理，并从原理中学习涉及的基础知识，比如HTTP协议，Socket通讯，J2EE的Servlet标准，多线程技术等，知道这些基础知识远比知道怎么简单实用一个框架要有用得多。















