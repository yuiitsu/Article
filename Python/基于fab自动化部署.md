# 基于fab自动化部署
fab是一个python库，强大好使，可以做很多帮助你减轻工作量的事情，比如在多台服务器上部署web项目，这里就讲讲使用它简单的方法来执行部署的过程。

关于fab的安装的基本使用，网上一搜一大把，内容都差不多，所以这里就不介绍，下载去官网：[http://www.fabfile.org/](http://www.fabfile.org/)

本文会涉及到以下几个内容：
1. 自动SSH登录
2. 运行命令
3. 错误处理
4. 带颜色输出

## 自动ssh登录
一般自动部署都不会只有一台机器，所以这里主要说多台机器，当然，一台也是一样的

首先，导入库
```
from fabric.api import *
```
fabric.api里封装了很多的方法，用来做一般的事情，两个方法就够了
```
1. run() 用于执行命令
2. cd() 用于进入目录
```
跑题了，基本操作用这两个就可以了，具体操作后面再讲，回到ssh登录上，我们知道，登录3要素，用户，地址和密码，所以先就定义一下。当然，不能随便定义，得按fab的方法来
```
# 定义host
# 方法 env.hosts = ['user@host1', 'user@host2']
# 示例
env.hosts = ['onlyfu@unjs.dinlei.com', 'root@unjs.dinlei.com']

# 设置密码
# 方法 env.passwords = {'host1:22': 'password1', 'host2:22': 'password2'}
# 示例
env.passwords = {'onlyfu@unjs.dinlei.com:22': '1234565', 'root@unjs.dinlei.com': 'ojoajewpb'}
```
这里要特别说明一下，在passwords里host一定要带上端口，默认端口本身来讲是不用带的，但在这里，如果要让密码跟host对上，就必须带着端口号才能成功。

env.passwords用于服务器的密码不一至的情况，如果几台服务器的密码都是相同的，可以直接使用env.password = '123456'
## 运行命令
正如前面所说，主要命令是run，它可以执行命令，如
```
# 执行单条命令
run("ls -l")
run("echo 'hello'")

# 执行多条命令
run("ls -l; echo 'hello'")

# 带变量的也一样，看看kill tomcat进程的命令
# 获取到pid，然后执行kill命令，就这么简单
run("pid=`(ps -ef | grep tomcat | grep -v \"grep\") | awk '{print $2}'`; kill -9 $pid")
```
再来说说cd命令，可能有人会说，既然run可以运行命令，那直接run("cd deploy")不就可以了嘛。事实证明，看上去对的，不一定真对，这个命令没错，是执行了，但它没进去，真的，很假，等于是告诉你：哥，我进去了哦，但没动，所以后面要做的操作，还是在当前目录下的，不信可以试
```
run('cd deploy')
run('ls -l')
```
看输出的目录是哪儿就明白了。不过虽然没进去，但如果执行run('cd deploy')时，没有deploy这个目录，程序是会报错的，有病，是不是。那要怎么做呢，这里就到cd方法出场了，不过光是它也不行，还要和with一起用：
```
with cd('deploy'):
    run('ls -l')
```
看看输出，是不是已经到deploy目录了，这样就可以执行艇作用于该目录的命令了
## 错误的处理
标准Python程序，出错程序就停止结束了，很多时间，也许并不想让它就此结束，想让它跳过继续执行后面的，那这里就要用到fab的错误处理方法了

使用错误处理，首先得告诉fab，我要进行这样的处理，它才会去处理，所以先得说一下：
```
with settings(warn_only = true):
    pass
```
warn_only = true就是在告诉fab这个事情，有了这一句，后面的语句都能享受到。

再说一个，run的每一条命令都有返回，成功或失败，判断，如果失败，执行confirm方法，来提示确认要不要继续，如：
```
# 导入confirm
from fabric.contrib.console import confirm

res = run('cd deploy') # 没有deploy这个目录
if res.failed and not confirm("cd deploy failed, Continue?"):
    abort('Abort')
```
运行上面的程序，res就会得到返回，这时候用res.failed来判断是否失败，如果失败，执行comfirm等待确认，如果选择N，执行abort终止程序
## 带颜色输出
这个其没什么用，只是想让输出漂亮一点，所以就说一下，方法很简单，使用colors包
```
# 导入包
from fabric.colors import *

print green('green txt')
print red('red txt')
print yellow('yellow txt')
```
太简单了
## 看一个完整一点的示例
```
应用场景：
共有4台服务器，要做Java Spring项目部署，服务器使用的tomcat
1. 登录第一台服务器
2. 进入jar包的目录
3. 使用git拉取新的jar包
4. 停止tomcat
5. 删除web目录，有可能多个
6. 将jar包拷到服务目录中，有可能性于多个不同的目录
7. 启动tomcat

登录下一台服务器循环
```
```
# -*- coding:utf-8 -*-
from fabric.api import *
from fabric.contrib.console import confirm
from fabric.colors import *

env.hosts = ['host1', 'host2', 'host3', 'host4']
env.passwords = {'host1:22': 'password1', 'host2:22': 'password2', 'host3:22': 'password3', 'host4:22': 'password4'}

def getRedMessage(txt, t):
	return "[%s] faield, Continue?" % txt if t == 'msg' else "Aborting at [%s]" % txt

def gitPull():

	actionName = "get pull"

	res = run('git pull origin master')
	if res.failed and not confirm(getRedMessage(actionName, 'msg')):
		abort(getRedMessage(actionName, 'abort'))

def copyFile():

	actionName = "copy file"

	res = run('cp -rf jar/* /jardir/; cp -rf war/* /warjar/')
	if res.failed and not confirm(getRedMessage(actionName, 'msg')):
		abort(getRedMessage(actionName, 'abort'))

def delWeb():

	actionName = "delete web"

	res = run('rm -rf /webapps/*')
	if res.failed and not confirm(getRedMessage(actionName, 'msg')):
		abort(getRedMessage(actionName, 'abort'))

def stopTomcat():

	actionName = "stop tomcat"

	res = run("pid=`(ps -ef | grep tomcat | grep -v \"grep\") | awk '{print $2}'`; kill -9 $pid")
	if res.failed and not confirm(getRedMessage(actionName, 'msg')):
		abort(getRedMessage(actionName, 'abort'))

def startTomcat():

	action = "start tomcat"

	res = run("/sh/start.sh")
	if res.failed and not confirm(getRedMessage(actionName, 'msg')):
		abort(getRedMessage(actionName, 'abort'))

def doploy():

	with settings(warn_only = true):
	with cd('cd deploy'):
	res = run('ls -l')
	if res.failed and not confirm(getRedMessage('cd deploy', 'msg')):
		abort(getRedMessage('cd deploy', 'abort'))

	gitPull()

	stopTomcat()

	delWeb()

	copyFile()

	startTomcat()
```
将上面的代码保存为fabfile.py文件，只有这个文件名，fab默认才能找到它，不然就要使用参数-f来加载文件名，在终端里运行：
```
fab deploy
```
好了，fab还有很多方法可以实现一些更加方便的操作，没事可以翻翻Docs，[http://docs.fabfile.org/en/latest/tutorial.html](http://docs.fabfile.org/en/latest/tutorial.html)
