# 一起来做chrome扩展《基础介绍》

首先说明，chrome的扩展并不它的插件，网上很多说写插件，其实都是说的扩展。写扩展并不复杂，只要根据chrome提供的一系列的API进行就可以实现很多的功能。只是对API的学习是有代价的，加上国内访问chrome官网文档并不顺利。虽然360提供了一个翻译文档，而且有不少例子，但它的内容还是太少，有些问题它仍然没有涉及。所以，如果是做一个简单的应用没有什么问题，但实际的项目开发往往得不到有用的介绍和解释。

这个系列的文章将从解决一些实际的问题入手，涉及的内容大致有以下几个：

- 基础配置
- content_script和background_script
- cookie的使用
- 本地存储
- ajax请求远程数据

博主也是在学习中，所说的也是开发过程中的一些总结，并不一定正确，如果有错，还请包涵。

写扩展只有一个文件是固定的，其它的并没有什么固定的要求，如固定的格式，固定的文件夹等。这个固定的文件就是它的配置文件，一个JSON格式的文件：manifest.json，基本内容如下：

##### manifest.json

```json
{
	"manifest_version": 2,
	"name": "My Extension",
	"description": "Extension description",
	"version": "1.0"
}
```

这是一个最简单的文件，有2个需要注意的问题：

1. 目前的规定，manifest_version必须写为2，所以其它的数字就不要写了，特别是1
2. JSON中，最后一项，是不能有,号的，不然它会启用失败的。

这几项不用介绍，应该都知道它们是干嘛的，它们会出现在扩展界面里，别的就没什么用了。一个真正的扩展当然不可能只有这些。它能涉及到的，博主简单归纳为两类：

1. chrome浏览器的元素，如Tab，书签，历史记录等。
2. 页面内容

那么，如果你的扩展需要涉及什么，就得把它添加到配置JSON里，也就是manifest.json文件中，如需要用到Tab：

```json
{
	"manifest_version": 2,
	"name": "My Extension",
	"description": "Extension description",
	"version": "1.0",
	"permissions": [
		"tabs"
	]
}
```

permissions即为允许的，它会告诉chrome，这个插件是允许操作Tab的，不然chrome就允许让你使用它Tab相关的API。所以，如果还有使用书签，或是别的什么东西时，都可以写在这里面。如，我们要使用代理相关的API：

```json
{
	"manifest_version": 2,
	"name": "My Extension",
	"description": "Extension description",
	"version": "1.0",
	"permissions": [
		"tabs",
		"proxy"
	]
}
```

proxy即代表可以使用代理相关的API，诸如比类还有cookie，history等，具体相关可以查阅官方文档

chrome扩展对目录并没有严格的要求，所以，除了manifest.json这个文件必须，其它的都是按需增加。目录结构可以按你的习惯建立的命名，都没有问题。