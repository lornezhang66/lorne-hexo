---
layout: file
title: elastic-job 简单实践
date: 2018-02-10 23:09:18
tags: 
 - 定时任务
 - elastic-job
 - 工具类
---

## elastic-job 简单实践


## 简介
>   elastic-job 是轻量级的分布式任务治理框架，出自当当网dd-framework 与 dubbox 都属于dd-framework下的子模块。

## 实践
>   maven   引入elastic-job包 和 elastic-spring 包

```xml:n
<properties>
    <!-- elastic.job 版本号 -->
    <elastic.job.version>1.1.1-SNAPSHOT</elastic.job.version>
</properties>
```

```xml:n
<!-- 引入elastic-job核心模块 -->
<dependency>
    <groupId>com.dangdang</groupId>
    <artifactId>elastic-job-core</artifactId>
    <version>${elastic.job.version}</version>
</dependency>

<!-- 使用springframework自定义命名空间时引入 -->
<dependency>
    <groupId>com.dangdang</groupId>
    <artifactId>elastic-job-spring</artifactId>
    <version>${elastic.job.version}</version>
</dependency>
```

>   新建spring-elastic.xml文件 elsetic spring 的配置文件

```xml:n
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   	xmlns:context="http://www.springframework.org/schema/context"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:reg="http://www.dangdang.com/schema/ddframe/reg"
    xmlns:job="http://www.dangdang.com/schema/ddframe/job"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
	http://www.springframework.org/schema/jee 
	http://www.springframework.org/schema/jee/spring-jee-4.0.xsd
	http://www.springframework.org/schema/tx 
	http://www.springframework.org/schema/tx/spring-tx.xsd
	http://www.dangdang.com/schema/ddframe/reg 
	http://www.dangdang.com/schema/ddframe/reg/reg.xsd 
	http://www.dangdang.com/schema/ddframe/job 
	http://www.dangdang.com/schema/ddframe/job/job.xsd ">
    <!--配置作业注册中心 -->
    <reg:zookeeper id="regCenter" server-lists="192.168.1.3:2181" namespace="logistics-job" base-sleep-time-milliseconds="1000" max-sleep-time-milliseconds="3000" max-retries="3" />
    
    <!-- 配置简单作业  -->
    <job:simple id="simpleElasticJob" class="com.logistics.edi.job.MyElasticJob" registry-center-ref="regCenter" cron="0/10 * * * * ?"   sharding-total-count="3" sharding-item-parameters="0=A,1=B,2=C" />

    <!-- 配置数据流作业
    <job:dataflow id="throughputDataFlow" class="xxx.MyThroughputDataFlowElasticJob" registry-center-ref="regCenter" cron="0/10 * * * * ?" sharding-total-count="3" sharding-item-parameters="0=A,1=B,2=C" process-count-interval-seconds="10" concurrent-data-process-thread-count="10" />
    -->
    <!-- 配置脚本作业
    <job:script id="scriptElasticJob" registry-center-ref="regCenter" cron="0/10 * * * * ?" sharding-total-count="3" sharding-item-parameters="0=A,1=B,2=C" script-command-line="/your/file/path/demo.sh" />
    -->
    <!-- 配置带监听的简单作业
    <job:simple id="listenerElasticJob" class="xxx.MySimpleListenerElasticJob" registry-center-ref="regCenter" cron="0/10 * * * * ?"   sharding-total-count="3" sharding-item-parameters="0=A,1=B,2=C">
        <job:listener class="xx.MySimpleJobListener"/>
        <job:listener class="xx.MyOnceSimpleJobListener" started-timeout-milliseconds="1000" completed-timeout-milliseconds="2000" />
    </job:simple>
    -->
</beans>
<!-- 192.168.1.3:2181 zookeeper.url -->
```

>   在 spring 配置文件里引入spring-elastic.xml

```xml
<import resource="spring-elastic.xml" />
```

>   编写Java文件 MyElasticJob.java

```java
package com.logistics.edi.job;

import org.apache.log4j.Logger;

import com.dangdang.ddframe.job.api.JobExecutionMultipleShardingContext;
import com.dangdang.ddframe.job.plugin.job.type.simple.AbstractSimpleElasticJob;

/** 
 * @Description: 定时任务
 * @author zhangxl
 * @date 2016年8月1日 下午3:07:08 
 * @version Ver 1.0
 */
public class MyElasticJob extends AbstractSimpleElasticJob {

	private Logger logger = Logger.getLogger(getClass());
	
	/** 
	 * @Description: 执行定时任务
	 * @param arg0
	 * @author zhangxl
	 * @date 2016年8月1日 下午3:07:08
	 */
	@Override
	public void process(JobExecutionMultipleShardingContext arg0) {
		logger.info("执行定时任务");
	}

}

```

## 总结
```
何为分布式作业？
分片概念
任务的分布式执行，需要将一个任务拆分为n个独立的任务项，然后由分布式的服务器分别执行某一个或几个分片项。

例如：有一个遍历数据库某张表的作业，现有2台服务器。为了快速的执行作业，那么每台服务器应执行作业的50%。
为满足此需求，可将作业分成2片，每台服务器执行1片。作业遍历数据的逻辑应为：服务器A遍历ID以奇数结尾的数据；服务器B遍历ID以偶数结尾的数据。
如果分成10片，则作业遍历数据的逻辑应为：每片分到的分片项应为ID%10，而服务器A被分配到分片项0,1,2,3,4
；服务器B被分配到分片项5,6,7,8,9，直接的结果就是服务器A遍历ID以0-4结尾的数据；服务器B遍历ID以5-9结尾的数据。

分片项与业务处理解耦
Elastic-job并不直接提供数据处理的功能，框架只会将分片项分配至各个运行中的作业服务器，开发者需要自行处理分片项与真实数据的对应关系。

分布式作业的执行
Elastic-job并无作业调度中心节点，而是基于部署作业框架的程序在到达相应时间点时各自触发调度。

注册中心仅用于作业注册和监控信息存储。而主作业节点仅用于处理分片和清理等功能。

个性化参数的适用场景
个性化参数即shardingItemParameters，可以和分片项匹配对应关系，用于将分片项的数字转换为更加可读的业务代码。

例如：按照地区水平拆分数据库，数据库A是北京的数据；数据库B是上海的数据；数据库C是广州的数据。 如果仅按照分片项配置，开发者需要了解0表示北京；1表示上海；2表示广州。 合理使用个性化参数可以让代码更可读，如果配置为0=北京,1=上海,2=广州，那么代码中直接使用北京，上海，广州的枚举值即可完成分片项和业务逻辑的对应关系。

作业高可用
Elastic-job提供最安全的方式执行作业。将分片项设置为1，并使用多于1台的服务器执行作业，作业将会以1主n从的方式执行。

一旦执行作业的服务器崩溃，等待执行的服务器将会在下次作业启动时替补执行。 开启失效转移功能效果更好，可以保证在本次作业执行时崩溃，备机立即启动替补执行。

最大限度利用资源
Elastic-job也提供最灵活的方式，最大限度的提高执行作业的吞吐量。将分片项设置为大于服务器的数量，最好是大于服务器倍数的数量，作业将会合理的利用分布式资源，动态的分配分片项。

例如：3台服务器，分成10片，则分片项分配结果为服务器A=0,1,2;服务器B=3,4,5;服务器C=6,7,8,9。 如果服务器C崩溃，则分片项分配结果为服务器A=0,1,2,3,4;服务器B=5,6,7,8,9。在不丢失分片项的情况下，最大限度的利用现有资源提高吞吐量。
```