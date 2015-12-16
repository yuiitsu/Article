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

之所以把curl和gd拿来说，目的是要搞清楚，这些三方库在编译PHP之前得安装好，可以直接使用yum安装在默认位置，也可以编译安装到指定位置，yum安装后，编译时不用指定库的安装位置，简单一点，看看基本的configure

```
./configure --prefix=/apps/php/php7.0 --enable-mbstring --with-curl --with-gd --with-config-file-path=/apps/php/php7.0/etc/ --enable-fpm --enable-mysqlnd --with-pdo-mysql=mysqlnd
```
