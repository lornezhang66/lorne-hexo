---
layout: file
title: Mave 国内镜像
date: 2018-02-10 23:24:46
tags:
 - maven
 - 工具类
---

@[maven, 国内镜像]

国内比较不错的maven 镜像
国内连接maven官方的仓库更新依赖库，网速一般很慢，收集一些国内快速的maven仓库镜像以备用。

```xml:n
<!-- 资金问题已关闭
<mirror>

      <id>CN</id>
      <name>OSChina Central</name>                                                                                                                       
      <url>http://maven.oschina.net/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>

    </mirror>
    -->
<!-- 目前可用 -->
  <mirrors>
    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>        
    </mirror>
  </mirrors>    
    
```
==================其他maven仓库镜像=========================
```xml:n
<mirror>  
      <id>repo2</id>  
      <mirrorOf>central</mirrorOf>  
      <name>Human Readable Name for this Mirror.</name>  
      <url>http://repo2.maven.org/maven2/</url>  
</mirror>  
<mirror>  
      <id>net-cn</id>  
      <mirrorOf>central</mirrorOf>  
      <name>Human Readable Name for this Mirror.</name>  
      <url>http://maven.net.cn/content/groups/public/</url>   
</mirror>  
<mirror>  
      <id>ui</id>  
      <mirrorOf>central</mirrorOf>  
      <name>Human Readable Name for this Mirror.</name>  
     <url>http://uk.maven.org/maven2/</url>  
</mirror>  
<mirror>  
      <id>ibiblio</id>  
      <mirrorOf>central</mirrorOf>  
      <name>Human Readable Name for this Mirror.</name>  
     <url>http://mirrors.ibiblio.org/pub/mirrors/maven2/</url>  
</mirror>  
<mirror>  
      <id>jboss-public-repository-group</id>  
      <mirrorOf>central</mirrorOf>  
      <name>JBoss Public Repository Group</name>  
     <url>http://repository.jboss.org/nexus/content/groups/public</url>  
</mirror>


<mirror>  
      <id>JBossJBPM</id> 
　　　　<mirrorOf>central</mirrorOf> 
　　　　<name>JBossJBPM Repository</name> 
　　　　<url>https://repository.jboss.org/nexus/content/repositories/releases/</url>
</mirror>
```
>   
>   -------------------------spring maven--------------------------------
>   http://maven.springframework.org/release/
>   ---------------------------------------------------------------------

>   http://maven.antelink.com/content/repositories/central/
>   http://nexus.openkoala.org/nexus/content/groups/Koala-release/
>   http://maven.tmatesoft.com/content/groups/public/
>   http://mavensync.zkoss.org/maven2/