---
layout: file
title: memcache 安装 使用
date: 2018-02-10 23:18:53
tags: 
 - 工具类
---

转载一篇在CentOS上安装memcache的方法。

所有操作都在SSH下，以根帐号登录。

我的版本为Centos Release 5.3 (Final)
使用这个命令可以知道你的Linux版本
cat /etc/redhat-release
首先要安装libevent库。
cd /usr/local/src
curl -O http://monkey.org/~provos/libevent-1.4.10-stable.tar.gz
tar xzvf libevent-1.4.10-stable.tar.gz
cd libevent-1.4.10-stable
./configure –prefix=/usr/local
make
make install

接下来就是安装memcached

cd /usr/local/src
curl -O http://www.danga.com/memcached/dist/memcached-1.2.8.tar.gz
tar xzvf memcached-1.2.8.tar.gz
cd memcached-1.2.8
LDFLAGS=’-Wl,–rpath /usr/local/lib’ ./configure –prefix=/usr/local
make
make install

安装完毕后，用下面这个命令以用户root来运行memcache
memcached -u root -d -m 64 -l 192.168.0.101 -p 11211
root 为所执行的用户
64 为缓存大小64M
192.168.0.101 为所在的服务器IP地址
11211 是所在端口

要关闭memcache
pkill memcached

接下来是安装php-pecl-memcache
一个命令就可以。
yum install php-pecl-memcache

还是需要php扩展，就用下面这个命令
pecl install memcache

接下来重启apache，用phpinfo()查看，应该可以看到memcache的部分，如果没有的话，检查这里的设置：
/etc/php.ini加上了 extension=memcache.so
当然也要确认memcache.so是否存在，是否在/usr/lib/php/modules/下，如果不是，那么找到它，并用完整路径表示。

查看memcache的运行情况，可以用memcache.php来查看。
当让也要有web 程序支持才有用，比如我用的phpbb 3就可以使用memcache，具体方法参考这里


原作者: David Yin

Update

不过我今天实战中并没有使用上面的方法：

介绍一下yum的方法：

yum install libevent

这个是第一步，

第二步是安装memcache，但是标准的CentOS5软件仓库里面是没有memcache相应的包的,所以，我们的第一步就是导入第三方软件仓库，这里推荐的是 Dag Wieers 库（现在叫 RPMForge 了），安装方法如下：

wget http://dag.wieers.com/rpm/packages/rpmforge-release/rpmforge-release-0.3.6-1.el5.rf.i386.rpm
rpm -ivh rpmforge-release-0.3.6-1.el5.rf.i386.rpm
查找相关软件包
yum search memcache
有了，现在可以安装了
yum -y install –enablerepo=rpmforge memcached php-pecl-memcache
验证一下安装结果
memcached -h
php -m|grep memcache
启动memcached
/sbin/servive memcached start
============

Update Dec 16th

Linux下Memcache服务器端的安装
服务器端主要是安装memcache服务器端，目前的最新版本是 memcached-1.3.0 。
下载：http://www.danga.com/memcached/dist/memcached-1.2.2.tar.gz
另外，Memcache用到了libevent这个库用于Socket的处理，所以还需要安装libevent，libevent的最新版本是libevent-1.3。（如果你的系统已经安装了libevent，可以不用安装）
官网：http://www.monkey.org/~provos/libevent/
下载：http://www.monkey.org/~provos/libevent-1.3.tar.gz

用wget指令直接下载这两个东西.下载回源文件后。
1.先安装libevent。这个东西在配置时需要指定一个安装路径，即./configure –prefix=/usr；然后make；然后make install；
2.再安装memcached，只是需要在配置时需要指定libevent的安装路径即./configure –with-libevent=/usr；然后make；然后make install；
这样就完成了Linux下Memcache服务器端的安装。详细的方法如下：

1.分别把memcached和libevent下载回来，放到 /tmp 目录下：
# cd /tmp

@[memcache, 安装]

# wget http://www.danga.com/memcached/dist/memcached-1.2.0.tar.gz
# wget http://www.monkey.org/~provos/libevent-1.2.tar.gz

2.先安装libevent：
# tar zxvf libevent-1.2.tar.gz
# cd libevent-1.2
# ./configure –prefix=/usr
# make
# make install

3.测试libevent是否安装成功：
# ls -al /usr/lib | grep libevent
lrwxrwxrwx 1 root root 21 11?? 12 17:38 libevent-1.2.so.1 -> libevent-1.2.so.1.0.3
-rwxr-xr-x 1 root root 263546 11?? 12 17:38 libevent-1.2.so.1.0.3
-rw-r–r– 1 root root 454156 11?? 12 17:38 libevent.a
-rwxr-xr-x 1 root root 811 11?? 12 17:38 libevent.la
lrwxrwxrwx 1 root root 21 11?? 12 17:38 libevent.so -> libevent-1.2.so.1.0.3
还不错，都安装上了。

4.安装memcached，同时需要安装中指定libevent的安装位置：
# cd /tmp
# tar zxvf memcached-1.2.0.tar.gz
# cd memcached-1.2.0
# ./configure –with-libevent=/usr
# make
# make install
如果中间出现报错，请仔细检查错误信息，按照错误信息来配置或者增加相应的库或者路径。
安装完成后会把memcached放到 /usr/local/bin/memcached ，

5.测试是否成功安装memcached：
# ls -al /usr/local/bin/mem*
-rwxr-xr-x 1 root root 137986 11?? 12 17:39 /usr/local/bin/memcached
-rwxr-xr-x 1 root root 140179 11?? 12 17:39 /usr/local/bin/memcached-debug

安装Memcache的PHP扩展
1.在http://pecl.php.net/package/memcache 选择相应想要下载的memcache版本。
2.安装PHP的memcache扩展

tar vxzf memcache-2.2.1.tgz
cd memcache-2.2.1
/usr/bin/phpize
./configure -enable-memcache -with-php-config=/usr/bin/php-config -with-zlib-dir
make
make install

3.上述安装完后会有类似这样的提示：

Installing shared extensions: /usr/local/php/lib/php/extensions/no-debug-non-zts-2007xxxx/

4.把php.ini中的extension_dir = “./”修改为

extension_dir = “/usr/local/php/lib/php/extensions/no-debug-non-zts-2007xxxx/”

5.添加一行来载入memcache扩展：extension=memcache.so

memcached的基本设置：
1.启动Memcache的服务器端：
# /usr/local/bin/memcached -d -m 10 -u root -l 66.90.103.147 -p 12000 -c 256 -P /tmp/memcached.pid

-d选项是启动一个守护进程，
-m是分配给Memcache使用的内存数量，单位是MB，我这里是10MB，
-u是运行Memcache的用户，我这里是root，
-l是监听的服务器IP地址，如果有多个地址的话，我这里指定了服务器的IP地址192.168.0.200，
-p是设置Memcache监听的端口，我这里设置了12000，最好是1024以上的端口，
-c选项是最大运行的并发连接数，默认是1024，我这里设置了256，按照你服务器的负载量来设定，
-P是设置保存Memcache的pid文件，我这里是保存在 /tmp/memcached.pid，

2.如果要结束Memcache进程，执行：

# kill `cat /tmp/memcached.pid`

也可以启动多个守护进程，不过端口不能重复。

3.重启apache，service httpd restart

Memcache环境测试：
运行下面的php文件，如果有输出This is a test!，就表示环境搭建成功。开始领略Memcache的魅力把！
$mem = new Memcache;
$mem->connect(“66.90.103.147″, 12000);
$mem->set(‘key’,'This is a test!’, 0, 60);
$val = $mem->get(‘key’);
echo $val;
?>

```
安装memcache时，需要建立文件索引或者说文件连接（link），类似windows下的快捷方式
启动服务时出现 error while loading shared libraries: libevent-2.0.so.5: cannot open shared object file: No such file or directory
>whereis libevent-2.0.so.5
libevent-2.0.so.5: /usr/local/lib/libevent-2.0.so.5
> ldd /usr/local/bin/memcached   （ldd指令不熟悉的去查看下）
libevent-2.0.so.5 => not found    （没有找到该文件）
libpthread.so.0 => /lib64/libpthread.so.0 (0x00002b83fce0e000)
libc.so.6 => /lib64/libc.so.6 (0x00002b83fd029000)
librt.so.1 => /lib64/librt.so.1 (0x00002b83fd381000)
/lib64/ld-linux-x86-64.so.2 (0x00002b83fc9b0000)
> LD_DEBUG=libs ./memcached -v
找到默认路径 /usr/lib/
>sudo ln -s /usr/lib/libevent-2.0.so.5 /usr/lib64/libevent-2.0.so.5
>sudo ldd /usr/local/bin/memcached
libevent-2.0.so.5 => /usr/lib64/libevent-2.0.so.5 (0x00002b83fcbcd000)
libpthread.so.0 => /lib64/libpthread.so.0 (0x00002b83fce0e000)
libc.so.6 => /lib64/libc.so.6 (0x00002b83fd029000)
librt.so.1 => /lib64/librt.so.1 (0x00002b83fd381000)
/lib64/ld-linux-x86-64.so.2 (0x00002b83fc9b0000)

```

-   /usr/local/bin/memcached -d -u root -l 127.0.0.1 -m 2048 -p 11211