# CentOS Python3使用pip安装pycurl

![](https://www.colorgamer.com/usr/uploads/2020/06/899164900.jpg)

*乱配图*

## 过程

在使用Tornado的AsyncHTTPClient的curl模式时，需要安装pycurl，因为是一套全编译环境，不太好直接使用easy_install来安装，只能使用pip：

```
pip install pycurl
```

很可能会遇到错误：

```
......
__main__.ConfigurationError: Could not run curl-config: [Errno 2] No such file or directory: 'curl-config': 'curl-config'
    ----------------------------------------
ERROR: Command errored out with exit status 1: python setup.py egg_info Check the logs for full command output.
```

没有curl-config，需要安装libcurl-devel

```bash
yum install libcurl-devel
```

再执行安装pycurl，如果报错：

```
error: command 'gcc' failed with exit status 1
```

安装gcc就好：

```
yum install gcc
```

这时，再安装pycurl，应该是可以安装了，但可能仍然有问题。在python里import pycurl，可能会报错：

```
ImportError: pycurl: libcurl link-time ssl backend (openssl) is different from compile-time ssl backend
```

说明还有依赖没有安装好，这时可以如下操作：

- pip uninstall pycurl
- export PYCURL_SSL_LIBRARY=openssl
- pip install pycurl

这样就可以使用ssl，但需要确保先安装好openssl，不然安装仍然会报错：

```
fatal error: openssl/ssl.h: No such file or directory
```

在CentOS下，执行安装：

```
yum install openssl-devel
```

再次安装pycurl，应该就可以了。

## 总结

安装之前的准备：

1. yum install libcurl-devel
2. yum install gcc
3. yum install openssl-devel
4. export PYCURL_SSL_LIBRARY=openssl
5. pip install pycurl

比较简单的方法是使用easy_install.