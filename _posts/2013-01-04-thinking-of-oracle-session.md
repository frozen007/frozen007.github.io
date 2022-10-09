---
layout: post
title:  "由Oracle的V$SESSION中的MACHINE字段想到的"
date:   2013-01-04 17:43:56 +0800
blurb:  "今天部门的一位同事发现oracle的连接数被占用了很多，于是想通过V$SESSION视图找出占用最多连接的用户"
og_image:
categories: tech
---

今天部门的一位同事发现oracle的连接数被占用了很多，于是想通过V$SESSION视图找出占用最多连接的用户，于是写了下面的sql：

```sql
SELECT USERNAME, MACHINE, COUNT(*) 
FROM V$SESSION 
GROUP BY USERNAME, MACHINE;
```

结果中有一个USERNAME=yhstat，MACHINE=jdbcclient的记录，显示有80多个连接。由此他觉得是机器名为jdbcclient的那台服务器占用了过多的连接。

从名字上看MACHINE的确是机器名，但我却觉得其实不是，jdbcclient只是表明这些连接都是由java的jdbc驱动程序创建的。但为什么有的MACHINE字段里的确写的是机器名，而有的不是呢？我事后特意查了一下JDBC的api，发现java.sql.Connection接口有几个方法：

```java
void setClientInfo(Properties properties)
Properties getClientInfo()
void setClientInfo(Properties properties)
String getClientInfo(String name)
```

这几个方法的用意很有可能就是为了设置一些额外连接信息的，比如V$SESSION里的MACHINE。那么jdbcclient是怎么来的呢？由于java.sql.Connection只是接口，具体实现要看每个数据库厂商的驱动程序，所以想搞明白来由还得查一下Oracle JDBC的API文档。果然，在oracle.jdbc.OracleConnection中可以发现一个常量定义：

```java
static final java.lang.String CONNECTION_PROPERTY_THIN_VSESSION_MACHINE
```

而它有以下描述：
```
Use this connection property to change the value that will show up in the "machine" column of the "v$session" table for this connection. 
Note that this setting only applies to the thin driver.
If you don't specify a value, by default, the driver will attempt to retrieve your host name. 
If the attempt fails, it will use "jdbcclient".
```

从上面描述中可以得知，这个常量就是上面提到过的方法 setClientInfo和 getClientInfo 所使用的key，对应的值是所在机器的host名，但如果获取host名失败，则使用jdbcclient作为值。那么什么情况下会获取host名失败呢？我想到的一个场景是启动Java应用的时候用的不是root用户，而是单独创建的一个linux用户，而且这个用户权限很小，没有获取host的权限，获取host时就会失败。而那些用root用户启动的Java应用，则可以获取host名。这样就可以解释为什么MACHINE有时可以显示准确的机器名，而有时只显示jdbcclient了。

对于Oracle来说，由于MACHINE字段的值完全可以由程序自己说了算，所以参考的意义其实并不大，况且实际的生产环境中很少有Java应用能用root来启动，从而导致可能无法获取host，MACHINE的值就更没意义了。如果想跟踪一个Connection，看来还是要靠ip更准确。