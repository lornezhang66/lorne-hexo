---
layout: file
title: linux 下安装svn 配置submin
date: 2018-02-10 23:20:21
tags:
 - svn
 - submin
 - 工具类
---

## linux 下安装svn 配置submin

@[svn, svn管理界面]


> linux 下安装配置svn 并配置submin 实现可视化权限管理

1.  安装svn 
    -   yum -y install subversion
    -   查询svn安装目录
        find / -name svnserver.conf
    -   创建svn版本库目录
        mkdir /home/work/svndata
    -   创建版本库
        svnadmin create /home/work/svndata
    -   配置svn
    
    >   vim /home/work/svndata/conf/svnserver.conf
        ## 打开下面几个注释
        anon-access = read #匿名用户可读
        auth-access = write #授权用户可写
        password-db = passwd #使用哪个文件作为账号文件
        authz-db = authz #使用哪个文件作为权限文件
        realm = /var/svn/svnrepos # 认证空间名，版本库所在目录 
        ---------------------------------------
        vim /home/work/svndata/conf/passwd
        ## 添加账号 = 密码
        admin = 123456
        -------------------------------------
        vim /home/work/svndata/config/authz
        ## 设置用户权限
        [/]
        admin = rw
        * = r
                
        -   启动svn版本库
        svnserve --listen-port 3690 -d -r /home/work/svndata
        -   停止svn服务
        killall svnserver
        
        
        
2.  开放3690 端口
    -   查询3690 端口是否启动
    
    ```sh:n
        netstat -an | grep  3690           
    ```
    -   开启3690 端口
    
    ```sh:n
        /sbin/iptables -I INPUT -p tcp --dport 3690 -j ACCEPT #开启3690 svn
        
    ```
    -   保存配置
    
    ```sh:n    
    /etc/rc.d/init.d/iptables save
    ```
    -   重启配置
    
    ```sh:n
     /etc/rc.d/init.d/iptables restart
     ```    
    -   查询端口是否开放
    
    ```sh:n
    /etc/init.d/iptables status    
    ```