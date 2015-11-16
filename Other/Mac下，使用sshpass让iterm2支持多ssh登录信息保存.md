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
