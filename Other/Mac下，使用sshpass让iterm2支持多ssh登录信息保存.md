# Mac下，使用sshpass让iterm2支持多ssh登录信息保存
windows里有个Xshell非常的方便好使，因为它能保存你所有的ssh登录帐号信息。MAC下并没有xshell，有些也提供这样的功能，但效果都不好。iterm2是很好的终端，但却不能很好的支持多profiles，当要管理的机器较多时，就比较麻烦了。好在它有profiles设置，只是不能保存ssh登录帐号及密码，它还提供了加载profiles时执行外部命令的功能，因此，这里就可以使用sshpass来帮它执行。
## 安装iterm2
直接到官网下载安装: [http://iterm2.com/](http://iterm2.com/)，mac上装软件，是件很轻松的事情
## 安装sshpass
下载：[http://sourceforge.net/projects/sshpass/files/](http://sourceforge.net/projects/sshpass/files/)
解压后，进入sshpass目录，执行安装
```java
./configure
make
make install
```
理论上不会出什么问题，安装好后，执行命令检查是否已经OK
```shell
sshpass -h
```
## 准备密码
让sshpass使用ssh密码，需要先将密码保存在一个文件里，再通过sshpass读文件来获取密码，iterm2就可以通过这样的命令来登录主机，密码文件很简单，取一个好名字，把密码写进去就可以了，没有别的任何东西，如，在用户目录的sshpass目录建一个名为pass的文件，里面写上主机密码：123456，文件地址为：/Users/用户名/sshpass/pass
## 配置iterm2
打开iterm的profiles选项
![image](https://github.com/onlyfu/Blog/blob/master/static/images/01.png)
