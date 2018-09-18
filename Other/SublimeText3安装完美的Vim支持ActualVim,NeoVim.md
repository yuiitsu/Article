# Sublime Text 3安装完美的Vim支持，ActualVim/NeoVim

很多IDE和编辑器都有Vim插件用于支持Vim模式，但大多数都有些问题，拿我一直用的Idea来说，它的vim在ctrl+v后，选择多行的行前插入，如果这几行中有空行，它不会把空格算在内，所以最终是会少操作空行行数的内容。虽然它已经很完美，但这个缺陷有时候也会让你很不爽。
在Sublime Text上，前几年就已经有了不少Vim的插件，也都或是有问题的。很久没有再使用过，这次突然发现了完美的解决方案，非常开心。
## 需要软件：
1. NeoVim
2. ActualVim

## 1. NeoVim
NeoVim是一个vim编辑器，和Sublime Text其实没有什么关系，但它是必要依赖，所以需要先安装。

### 安装NeoVim
1. 下载：[https://github.com/neovim/neovim/releases](https://github.com/neovim/neovim/releases)
2. 解压放到你喜欢的位置
3. 配置环境变量，将neovim的目录加入到环境变量中

### 配置环境变量：
### windows:
neovim在D盘根目录，那它的路径就是：D:\Neovim\bin，将它添加到环境变量中即可(mac和linux可以直接修改.profile将路径加入)。

## 2. ActualVim
ActualVim就是Sublime Text的一个插件了，正常安装它
Install Package
找到ActualVim，安装即可

如上，重启Sublime后，完美的Vim已经可以正常使用了，非常简单。**最主要就是需要添加NeoVim的bin到环境变量**。
后话，虽然Sublime Text 3完美的支持了Vim，但如果因为要断点继续安装更多的插件，其实我是不愿意的，所以用它来写写前端还是可以的，Python/Java/Node之类的，我还是会用Idea，即使有点小缺陷。

谢谢阅读