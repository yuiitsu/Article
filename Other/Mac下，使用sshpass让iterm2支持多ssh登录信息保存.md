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
添加一个新的profile，其它没什么好配置的，主要是在General的command中选择使用command，命令就是sshpass的执行命令，如：
```java
/usr/local/bin/sshpass -f /Users/fuwy/sshpass/pass ssh -p22 root@112.124.25.173
```
/usr/local/bin/sshpass是sshpass执行文件的路径，如果按默认情况安装，它肯定会出现在这个位置上

-f 是告诉sshpass加载文件

/Users/fuwy/sshpass/pass就是要加载的文件，即前面建的密码文件

ssh -p22 root@112.124.25.173是说用ssh链接，端口22，root帐号和IP地址

保存后，选择该profile，就可以实际ssh登录。只是如果是本机第一次登录，是不会成功的，因为ssh登录需要你yes确认，会写文件到hosts里，所以第一次会直接失败，再来一次或是先在终端里用ssh root@ip来登录一次，就可以了。
这样，新建多个profile，就可以实现管理登录了。

iterm2和xshell比起来还是有很多不好使的地方，比如，无法直接看到主机IP，有时候想复制一下不方便。tab上的名称不能自定义，多开几个之后，不太好区分等等。但是有总比没有好，所以，还是不错的。
