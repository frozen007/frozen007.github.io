---
layout: post
title:  "DIY一个没有OGNL漏洞的Struts2"
date:   2014-06-03 16:55:09 +0800
blurb:  "自从struts2引入OGNL用于解析请求参数以后，参数注入漏洞的安全问题越来越突出"
og_image:
categories: tech
---

最开始接触的MVC框架就是struts，那时候的版本是1.x，它配置清晰，简单而且灵活，上手很快，所以被用于大多数网站的服务器框架。可自从struts2引入OGNL用于解析请求参数以后，参数注入漏洞的安全问题越来越突出，虽然后续的struts2升级中封堵了大部分漏洞，但是老是这样亡羊补牢也不是长久之计。

在移动应用服务器的架构选择中，我还是选用了struts2。因为我发现移动应用大部分都是直接通过HTTP接口提交请求数据，请求参数都是简单的key-value，所以struts2本来具有的强大的参数解析功能，完全没有用武之地，如果能把OGNL从struts2中完全移除，那么这个服务器架构就可以经受住大部分已知的参数注入攻击。

可以简单概括，都是OGNL惹的祸。那我接下来的主要任务就是去除OGNL解析参数的功能。

首先必须搞清楚struts2（v2.3.1.2）接受请求并进行分发的基本原理和流程。通过检查源码，我更倾向于用org.apache.struts2.dispatcher.ng.servlet.StrutsServlet而不是org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter，因为StrutsServlet里的流程更加清晰，比较适合进行改造，servlet主方法如下：

```java
public void service(HttpServletRequest request, HttpServletResponse response) 
                throws IOException, ServletException {
   try {
       prepare.createActionContext(request, response);
       prepare.assignDispatcherToThread();
       prepare.setEncodingAndLocale(request, response);
       request = prepare.wrapRequest(request);
       ActionMapping mapping = prepare.findActionMapping(request, response);
       if (mapping == null) {
           boolean handled = execute.executeStaticResourceRequest(request, response);
           if (!handled)
               throw new ServletException("......");
       } else {
           execute.executeAction(request, response, mapping);
       }
   } finally {
       prepare.cleanupRequest(request);
   }
}
```

熟悉struts2用法的人都知道，它可以将请求参数中的key-value直接映射到Action类的对应成员变量上，而且支持Map，List等，那么这就需要OGNL对请求参数按照其特定的语言进行参数解析，将解析后的对象放入到ActionContext中，然后再注入到对应的Action实例上。而大部分漏洞就发生在OGNL解析参数的时候，所以应该可以在红色的方法createActionContext找到ONGL解析参数的身影（具体代码原理不再列出）：

```java
ValueStack stack = dispatcher.getContainer().getInstance(ValueStackFactory.class).createValueStack();
stack.getContext().putAll(dispatcher.createContextMap(request, response, null, servletContext));
ctx = new ActionContext(stack.getContext());
```

再看一下struts2的标准xml文件，里面会有一行xwork2的注入定义

因此，可以将这段定义的实现类改为一个我们自己的"OGNL"实现，也就是一个空的实现，就可以避免struts2调用OGNL引擎去解析参数。先看了一下OgnlValueStackFactory创建的com.opensymphony.xwork2.ognl.OgnlValueStack实现类的代码，里面核心方法是findValue、findString这些方法，最终会调用tryFindValue去走OGNL引擎，所以只需做一些技巧，重新定义它们，让其直接返回即可。

幸运的是，在struts2的源代码中，xwork-core的test包中免费赠送了一个这种东西，直接拿过来改改就行了：struts-2.3.1.2\src\xwork-core\src\test\java\com\opensymphony\xwork2\StubValueStack.java，看了代码后发现，这个实现原理就是往里面放了什么值取出来以后还是什么值，符合我们的要求。

经过上面的改造后的struts2，可以抵御目前已知的大部分远程代码执行攻击，但唯一的缺陷是不再支持OGNL，表现就是无法适用于WEB开发，特别是无法实现请求参数与Action类中成员变量的自动绑定。但由于是开发移动应用接口服务器，通讯的参数值比较简单，所以完全不受影响，并且由于上了一层参数解析，执行速度会比之前略有提高。
