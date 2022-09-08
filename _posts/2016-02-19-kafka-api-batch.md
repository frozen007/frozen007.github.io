---
layout: post
title:  "kafka高级API扩展-实现批量处理消息"
date:   2016-02-19 13:48:01 +0800
blurb:  "实现kafka批量提交的一种思路"
og_image:
categories: tech
tags:
- kafka
---

转自我之前写的一篇文章，虽然现在已经用不到kafka了。。。

之前在创业公司做一个社区app时，曾使用kafka来传递feed数据，但使用的是原生的kafka consumer api，很难做到真正的批量，提交offset也不是很方便，重要的是每次都有配置一大堆辅助的类。

为了解决这些我封装了一个比较容易使用的consumer和produer库，完全基于原生的kafka高级api，实现了批量处理批量提交offset，并且对于新配置一个topic consumer也非常方便。使用这个库开发的项目已经在实际的线上环境运行几个月了，表现良好，每日负责处理大量的feed和用户行为流数据。

配置方式：

    <!--初始化BatchConsumer，会自动启动一个线程-->
    <bean id="kafkaBatchConsumer" class="com.myz.base.kafka.KafkaBatchConsumer" init-method="start">
        <property name="zkConnect" value="${kafka.queue.zookeeper_connect}"/>
        <property name="topic" value="${topicName}"/>
        <property name="groupId" value="${group_id}"/>
        <property name="consumerId" value="${consumer_id}"/>
        <property name="autoCommit" value="false"/>
        <property name="consumerTimeout" value="10"/>
        <property name="messageConsumer" ref="messageConsumer"/>
    </bean>

    <!--实现一个具体的消息处理类，里面可以加入自己的处理逻辑-->
    <bean id="messageConsumer" class="xxxx.queue.FeedMessageConsumer">
    </bean>


[github主页](https://github.com/frozen007/kafka-effective)
