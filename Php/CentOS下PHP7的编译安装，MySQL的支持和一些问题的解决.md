# CentOS下PHP7的编译安装，MySQL的支持和一些问题的解决

最近试了一下PHP7，在编译和支持MySQL上都遇到一些问题，相信不少同学也同样遇到，所以在这里聊一下这个过程。简单来讲编译PHP7只需要3步：

> 1、./buildconf --force

> 2、./configure

> 3、make && make install

1、3步，都没啥好管的，configure是编译的关键，涉及到PHP对一些库的支持，这里我们使用最小的支持，包括MySQL：

> curl

> gd

> fpm

> mysqlnd

之所以把curl和gd拿来说，目的是要搞清楚，这些三方库在编译PHP之前得安装好，可以直接使用yum安装在默认位置，也可以编译安装到指定位置，yum安装后，编译时不用指定库的安装位置，关于安装这些库，可以搜索一下有很多。下面看看基本的configure

```
./configure --prefix=/apps/php/php7.0 --enable-mbstring --with-curl --with-gd --with-config-file-path=/apps/php/php7.0/etc/ --enable-fpm --enable-mysqlnd --with-pdo-mysql=mysqlnd
```

## PHP-FPM

关于fpm，相信不用多说，用它来支持PHP是一个比较好的选择，PHP5.3.3开始就已经内置了php-fpm，所以PHP7里当然也有，只需要--enable-fpm一下就可以了

php-fpm参数：

> --start 启动

> --stop 强制终止

> --quit 平滑终止

> --restart 重启

> --reload 重新平滑加载php的php.ini

> --logrotate 重新启用log文件

## MySQL支持

重点讲一下这个，因为在它上面花了一点时间，不知道从哪一版本开始，PHP不在希望使用mysql的库来支持mysql的连接，启用了mysqlnd来支持，听说比libmysql要快很多，PHP5.x还可以使用libmysql，PHP7貌似已经取消了支持，编译都没有了--with-mysql参数，只支持--with-mysqli和--with-pdo-mysql，可以通过查看configure的参数来知道：

```
./configure -help | grep mysql
```

![image](https://github.com/onlyfu/Blog/blob/master/static/images/php/20151216/001.png)

可以看到，PHP希望使用mysqlnd来支持MySQL，所以参数可以这样写：

> --enable-mysqlnd

> --with-mysqli=mysqlnd

> --with-pdo-mysql=mysqlnd

mysqlnd是不需要mysql支持的，所以不用先安装好mysql一样可以编译通过，启动php-fpm，查看一下phpinfo，能看到mysqlnd和pdo_mysql表示php已经可以支持mysql了（这里用的是pdo，mysqli同理）

![image](https://github.com/onlyfu/Blog/blob/master/static/images/php/20151216/002.png)

![image](https://github.com/onlyfu/Blog/blob/master/static/images/php/20151216/003.png)

## 几个问题

### 编译问题：cc: Internal error: Killed (program cc1)

这个问题是第一次遇到，原来是我的阿里云服务器关掉了swap，内存不够用，就报了这个错。
解决办法很简单，configure时加上--disable-fileinfo参数就可以了。

### PHP报找不到mysql服务

正如它所说，确实没找到，看看phpinfo中pdo_mysql.default_socket项

![image](https://github.com/onlyfu/Blog/blob/master/static/images/php/20151216/004.png)

mysql.sock在哪里，再看一下mysql.sock的真正位置，使用命令：ps -ef|grep mysql查看：

![image](https://github.com/onlyfu/Blog/blob/master/static/images/php/20151216/005.png)

明显不在一个位置上，我的正确位置是：/var/lib/mysql/mysql.sock

所以，修改一下php.ini，找到pdo_mysql.default_socket，改为你的实际位置，重启一下php-fpm，如果还不行，可以到/tmp目录下建立一个mysql.sock的软链接：

```
ln -s /var/lib/mysql/mysql.sock mysql.sock
```

再重启一次php-fpm，相信已经正常运行了。如果需要PHP支持的库更多，可以再次编译，在configure时把需要的支持加上，就是--with-xxx这中，记得如果是三方的，要先安装这些库才行哦。
