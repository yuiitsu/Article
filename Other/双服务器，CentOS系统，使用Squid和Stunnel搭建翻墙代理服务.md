# 双服务器，CentOS系统，使用Squid和Stunnel搭建翻墙代理服务

Squid是很好的代理服务器，但它不能直接翻墙，因为在连接到Squid的过程中，就已经被墙了。所以得把传输的数据先进行加密再传输，这样等于是绕过了防火墙。这里使用的软件就是Stunnel。下面就是实施的过程以及在安装配置过程中可能遇到的问题和解决办法。

## 服务器需求：

1. 一台可以正常访问互联网的服务器，代号A（国外的）
2. 一台可以正常访问服务器A的国内服务器，代号B

## 使用系统：

CentOS 6.5（随意）

## 访问过程：

1. 用户设置代理指向B服务器的Stunnel服务监听的端口，访问网站
2. B服务器的Stunnel会将信息做加密处理，然后发送到A服务器的Stunnel服务上
3. A服务器的Stunnel会将加密的信息做解密处理，然后发送给A服务器的Squid服务上
4. A服务器的Squid会向目标网站请求数据，然后将信息返回。

## 实施步骤：

一、在服务器A上安装Squid和Stunnel

1、安装Squid
到www.squid-cache.org上找最新版本的Squid，目前是3.4.5，下载地址是：http://www.squid-cache.org/Versions/v3/3.4/squid-3.4.5.tar.gz

```
//下载软件包
$wget http://www.squid-cache.org/Versions/v3/3.4/squid-3.4.5.tar.gz
//解压
$tar -zvxf squid-3.4.5.tar.gz
//进入软件包
$cd squid-3.4.5
//编译安装
$./configure  --with-openssl=/usr/lib64/openssl
$make
$make install
```

因为Squid需要openssl，所以configure需要带上openssl，openssl的路径需要根据你的系统来确定，我这里是在/usr/lib64/openssl，所以带上了它。

如果没有错误提示，说明基本上安装成功

启动Squid

```
$/usr/local/squid/sbin/squid -s
```

如果什么配置都不做修改，那默认的商品号为3128，这时候，可以打开你的浏览器，设置代理，填上A服务器的IP和3128端口号，访问baidu，看看remote是不是A服务器的地址和端口号，如果是，说明Squid已经能正常使用了。只是这时还不能翻墙。

安装过程中可能会遇到的一些问题：

**configure: error: no acceptable C compiler found in $PATH**

本问题是没有安装GCC，安装即可：

```
$yum install gcc
```

**configure: error: C++ compiler cannot create executables**

本问题是安装了GCC，但没有安装C++，安装即可：

```
$yum install gcc-c++
```

**/usr/bin/ld cannot find -lssl**

本问题是没有指定openssl目录位置，在configure的时候把系统中openssl所在的目录带上

```
$./configure --with-openssl=/usr/lib64/openssl
```

**make时报错：ext_file_userip_acl.cc:254: error: 'errno' was not declared in this scope**

请修改文件：helpers/external_acl/file_userip/ext_file_userip_acl.cc

在#include "util.h"下面加一行

```
#include <cerrno>
```

**外网访问提示：Access Denied**

说明配置只允许本地访问，修改配置文件/usr/local/squid/etc/squid.conf

```
http_access allow all
```

2、安装Stunnel
先到http://www.stunnel.org/downloads.html上找到最新的版本，目前是5.01，下载地址为：http://www.stunnel.org/downloads/stunnel-5.01.tar.gz

```
//下载软件包
$wget http://www.stunnel.org/downloads/stunnel-5.01.tar.gz
//解压
$tar -zvxf stunnel-5.01.tar.gz
//进入软件包
$cd stunnel-5.01
//编译安装
$./configure
$make
$make install
```

正常情况下不会出什么问题，如果make失败，大多会是没有安装openssl-devel，安装上即可：

```
$yum install openssl-devel
```

再次make应该能成功，如果再不行，请google，一般都会是一些软件未安装倒至的。之后，可以使用stunnel -version来查看是否安装成功。

配置Stunnel：
Stunnel比Squid要麻烦一些，因为Squid如果你什么都设置，同样能运行，而Stunnel就不行。它需要一个配置文件，还需要一个签名证书。

帮助文档上说安装完后系统会自动在/usr/local/etc/stunnel/里生成一个密钥文件stunnel.pem。but，我硬是没看到它，所以这里我们还得自己来生成一个。

```
//先进到目录
$cd /usr/local/etc/stunnel/
//生成密钥
$openssl req -new -x509 -days 365 -nodes -config openssl.cnf -out stunnel.pem -keyout stunnel.pem
//如果报错找不到openssl.cnf，可以把-config openssl.cnf去掉，即：
$openssl req -new -x509 -days 365 -nodes -out stunnel.pem -keyout stunnel.pem
```

这样会在/usr/local/etc/sutnnel/里生成密钥文件stunnel.pem

接着给它生成Diffie-Hellman部分：

```
$openssl gendh 512>> stunnel.pem
```

网络上说这是4.X版本必须要做的，但我使用的5.01也这么做了。

设置配置文件：
在/usr/local/etc/stunnel/目录下有一个stunnel.conf.simple文件（好像是这样的），可以cp一份为stunnel.conf或是新建一个stunnel.conf，这里使用新建

```
$vim stunnel.conf
```

将以下内容复制进去

```
cert = /usr/local/etc/stunnel/stunnel.pem
CAfile = /usr/local/etc/stunnel/stunnel.pem
socket = l:TCP_NODELAY=1
socket = r:TCP_NODELAY=1

;;;chroot = /var/run/stunnel
pid = /tmp/stunnel.pid
verify = 3

;;; CApath = certs
;;; CRLpath = crls
;;; CRLfile = crls.pem

setuid = stunnel
setgid = stunnel

;;; client=yes
compression = zlib
;;; taskbar = no
delay = no
;;; failover = rr
;;; failover = prio
sslVersion = TLSv1
fips=no

debug = 7
syslog = no
output = stunnel.log

[sproxy]
accept = 34567
connect = 127.0.0.1:3128
```

这里有几个设置要说明一下：

第一行和二行是密钥文件的位置，如果按前面的做法，这里肯定是正确的。

```
setuid = stunnel
setgid = stunnel
```

是设置用户和用户组，都为stunnel，一般情况下是不会有它们的，所以要新建用户和用户组：

```
$groupadd -g 122 stunnel
$useradd -c stunnel -d /nonexistent -m -g 122 -u 122 stunnel
```

accept = 34567 是监听的端口号，也就是B服务器要指向的位置

connect = 127.0.0.1:3128 是本服务器，也就是A服务器Squid监听的端口号，也就是3128啦。

保存退出后，就可以试着启动stunnel了

```
$stunnel
```

如果正常是没有输出任何内容的，如果有问题，它会给出问题所在，仔细排查，基本上不会有什么问题。

检查是否正常运行：

```
$ps -ef | grep stunnel
```

如果看到stunnel用户运行的stunnel，说明已经成功运行了。

这样A服务器的Squid和Stunnel都配置完成了，接下来配置国内B服务器的Stunnel。B服务器不需要Squid，所以只需要配置Stunnel

二、在服务器B上安装配置Stunnel

安装同A，密钥不要再生成了，从服务器A上拷过来

登录服务器A，进到/usr/local/etc/stunnel/目录，向服务器B的/usr/local/etc/stunnel/目录里拷贝stunnel.pem密钥：

```
$cd /usr/local/etc/stunnel/
$scp stunnel.pem root@服务器B的IP:/usr/local/etc/stunnel/
```

接着登录服务器B，设置配置文件，同服务器A，可以cp一个，也可以新建，这里同样新建：

```
$cd /usr/local/etc/stunnel/
$vim stunnel.conf
```

将下面的内容复制到里面：

```
cert = /usr/local/etc/stunnel/stunnel.pem
socket = l:TCP_NODELAY=1
socket = r:TCP_NODELAY=1
verify = 2
CAfile = /usr/local/etc/stunnel/stunnel.pem
client=yes
compression = zlib
ciphers = AES256-SHA
delay = no
failover = prio
sslVersion = TLSv1
fips = no
[sproxy]
accept  = 0.0.0.0:7071
connect = 服务器A的IP:34567
```

这里要说明的是：

accept = 0.0.0.0:7071 中的7071是用户需要设置的代理端口，可以随意设置，只要大于500就好。0.0.0.0是为了让外网能使用，如果只是内部使用，改成127.0.0.1即可。

connect = 服务器A的IP:34567，很显然这里要填什么，34567是服务器A的Stunnel监听的端口号，保持和它一至就对了。

保存退出后，就可以启动Stunnel了

```
$stunnel
```

如果没有意外，整个代理就正常运行了。打开你的浏览器，将代理设置为你服务器B的IP+7071端口号，然后访问一些不能访问的站点试试看。
