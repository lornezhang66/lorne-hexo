---
layout: file
title: Maven Nexus安装使用
date: 2018-02-10 23:22:20
tags:
 - maven
 - nexus
 - 工具类
---

@[maven, Nexus]

私服不是Maven的核心概念，它仅仅是一种衍生出来的特殊的Maven仓库。

 如果没有私服，我们所需的所有构件都需要通过maven的中央仓库和第三方的Maven仓库下载到本地，而一个团队中的所有人都重复的从maven仓库下载构件无疑加大了仓库的负载和浪费了外网带宽，如果网速慢的话，还会影响项目的进程。很多情况下项目的开发都是在内网进行的，连接不到maven仓库怎么办呢？开发的公共构件怎么让其它项目使用？这个时候我们不得不为自己的团队搭建属于自己的maven私服，这样既节省了网络带宽也会加速项目搭建的进程，当然前提条件就是你的私服中拥有项目所需的所有构件。

 
1. 安装Nexus

Nexus是典型的JavaWeb应用，它有两种安装包，一种是包含Jetty容器的Bundle包，另一种是不包含Web容器的war包。

1）下载Nexus

官网http://www.sonatype.org/nexus/ 下载最新的Nexus。

2）Bundle方式安装Nexus

a.首先看下解压后的目录，结构：

解压后存在两个文件夹：nexus-2.4.0-01（不同版本版本号不同）和sonatype-work。

nexus-2.4.0-01: 该目录包含了Nexus运行所需要的文件，如启动脚本、依赖jar包等。

sonatype-work：该目录包含Nenus生成的配置文件、日志文件、仓库文件等。

第一个目录是运行Nexus必须的，而第二个不是必须的，Nexus会在运行的时候动态创建该目录。

b.配置Path，启动Nexus（在windows操作系统上）

首先在环境变量path下加入如下地址：C:\nexus\nexus-2.6.2-01-bundle\nexus-2.6.2-01\bin；之后在cmd下启动Nexus服务：

 
启动成功后，可以打开打开浏览器访问：http://localhost:8081/nexus 就可以看到Nexus的界面了，如下图：

8081为默认的端口号，要修改端口号可进入nexus-2.1.2-bundle\nexus-2.1.2\conf\打开nexus.properties文件，修改application-port属性值就可以了
 
 


这时你可以单击界面右上角的Login进行登录，Nexus默认管理用户名和密码为admin/admin123。

b.1在Linux上切换到/opt/nexus/nexus-2.14.0-01/bin目录下

存在nexus及nexus.bat文件，可以使用./nexus start 启动nexus，这时候可能会报错。

这时候提示：
****************************************
WARNING – NOTRECOMMENDED TO RUN AS ROOT
****************************************
If you insist running as root, then set theenvironment variable RUN_AS_USER=root before running this script.
大概意思就是要在环境配置export RUN_AS_USER=root，临时配置
在命令行下输入：
export RUN_AS_USER=root
然后执行，就不会再提示了
./nexus start
 
也可以在系统里面永久配置
vi /etc/profile  加入export RUN_AS_USER=root
 

2. Nexus的索引

这时你使用Nexus搜索插件得不到任何结果，为了能够搜索Maven中央库，首先需要设置Nexus中的MavenCentral仓库下载远程索引。如下图：


单击左边导航栏的Repositories，可以link到这个页面，选择Central，点击Configuration，里面有一个DownloadRemote Indexes配置，默认状态是false，将其改为true，‘Save’后，单击Administration==> ScheduledTasks,就有一条更新Index的任务，这个是Nexus在后天运行了一个任务来下载中央仓库的索引。由于中央仓库的内容比较多，因此其索引文件比较大，Nexus下载该文件也需要比较长的时间。请读者耐心等待把。如果网速不好的话，可以使用其他人搭建好的的Nexus私服。后面会介绍。下图为Nexus后台运行的task图：


 

3.配置Maven从Nexus下载构件

 

1）在POM中配置Nexus私服，这样的配置只对当前的Maven项目有效。

<repositories>
      <repository>
          <id>nexus</id>
          <name>Nexus Repository</name>
          <url>http://localhost:8081/nexus/content/groups/public/</url>
          <releases>
              <enabled>true</enabled>
          </releases>
          <snapshots>
              <enabled>true</enabled>
          </snapshots>
      </repository>
  </repositories>
2）在settings.xml中配置profile元素，这样就能让本机所有的Maven项目都使用自己的Maven私服。 
<mirrors>
    <mirror>
      <id>central</id>
      <mirrorOf>*</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://localhost:8081/nexus/content/groups/public/</url>
    </mirror>
  </mirrors>
  <profiles>
    <profile>
      <id>nexus</id>
      <repositories>
        <repository>
          <id>nexus</id>
          <name>Nexus</name>
          <url>http://localhost:8081/nexus/content/groups/public/</url>
            <releases>
                        <enabled>true</enabled>
                  </releases>
          <snapshots>
                        <enabled>true</enabled>
                  </snapshots>
        </repository>
      </repositories>
    </profile>
</profiles>
以上配置所有Maven下载请求都仅仅通过Nexus，以全面发挥私服的作用。

4. 部署构件到Nexus

1）在POM中配置

<project>
  ... 
  <distributionManagement>
<snapshotRepository>
        <id>user-snapshots</id>
        <name>User Project SNAPSHOTS</name>
        <url>http://localhost:8081/nexus/content/repositories/MyUserReposSnapshots/</url>
    </snapshotRepository>
    
      <repository>
          <id>user-releases</id>
          <name>User Project Release</name>
          <url>http://localhost:8081/nexus/content/repositories/MyUserReposRelease/</url>
      </repository>
      
  </distributionManagement>
   ...
</project>
2）settings.xml中配置认证信息，Nexus的仓库对于匿名用户是只读的。 
<servers>
  
    <server>
      <id>user-snapshots</id>
      <username>lb</username>
      <password>123456</password>
    </server>
        
    <server>
      <id>user-releases</id>
      <username>lb</username>
      <password>123456</password>
    </server>
        
  </servers>
