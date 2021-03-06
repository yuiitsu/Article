# 我为什么要写YuiAPI

## 目录

- 关于YuiAPI
- 原因
- 需求是什么
- 使用YuiAPI

## 关于YuiAPI

YuiAPI在我的计划中分为两个部分

1. 基于Chrome Extenstions的http请求客户端
2. 集API测试，文档，监控，团队协作为一体的平台应用

目前Chrome Extensionts部分完成度有70%，基本上可以满足除分组测试，文档化之外的日常API调用功能。本文只是想聊一聊，我为什么要写YuiAPI，因为它看上去就是一个PostMan。

## 原因

其实这里仅仅只有三个原因：

1. 使用别家产品是或多或少都有一些不满足于自己的需求
2. 通过完成一个软件，可以更深入的了解一些相关的技术点
3. 我喜欢造轮子，我坚持认为，不喜欢造轮子的程序员并不爱程序

## 需求是什么

最开始，做API调试的时候，我使用的也是postman，但很快我就发现有几个地方不是很满意：

1. **host不能很灵活的切换**。因为我们有不同的服务器，有时候需要切换api host来实现使用同一组测试用例分别访问。但postman的解决方案异常邪道，<code>使用预定义的变量进行</code>（这还是后来看某个同学的postman才知道的）。为什么postman就不能提供一个选择的方式进行host的切换，我一直不明白。所以，YuiAPI就有这个功能。

   ![](https://www.colorgamer.com/usr/uploads/2018/11/4078681082.png)

2. **参数编辑/响应结果可视面积小**。我很多时候直接使用笔记本，屏幕较小，postman的参数编辑和响应结果都在右则，上下结构，如果参数多一点，响应结果长一些，看起来非常的费力。所以，YuiAPI将参数编辑和响就结果做成左右分屏，这样就有相对足够的高度显示参数和响应结果。

   ![](https://www.colorgamer.com/usr/uploads/2018/11/808335562.png)

3. **postman只记录请求参数，不记录响应结果，回头想再看，还得再请求一次**。很多时候，我都希望打开历史记录时，可以直接看到上次的响应结果，而不需要先发请求。所以YuiAPI将响应结果也存了起来。
4. **文档化**。我尝试过一些自动文档工具，但说实话，都无法很好的满足需求。我甚至想用Python自己写一个API文档生成工具，但仍然需要对API的代码进行规范定制，和直接到文档应用上创建没有太大的区别。当我发现API开发者都会使用client工具测试自己API时，我就觉得API的文档化应该在这个测试客户端上集成，因为既然都要测，参数结果都有了，为什么不利用起来呢？即使并没有说明。所以，YuiAPI会尽量的优化文档的生成，提高文档生成的便捷性。
5. **协作**。一个团队，开发与测试，往往会出现测试的同学在应用端测试时，发现了问题，开发的同学想利用他们的参数进行调试时，非常的麻烦，如果API的测试用例可以共享，相信大家都会很乐意。所以，YuiAPI有此打算。

就是这些小小的需求，都是我在使用过程中想得到的，所以，从这个角度出发，我自己花了大量时间写了YuiAPI，虽然现在还未完成，但客户端已经是可用的。在Chrome的应用商店目前有400的安装量，这是最开心的。

![](https://www.colorgamer.com/usr/uploads/2018/11/1647555530.png)

不管有多少人觉得这家伙别人已经做了，你做就是浪费时间，但我会坚持完成我所计划的所有功能，再简单的东西，只有你自己做出来了，你所得到的不仅仅是这个东西。

## 使用YuiAPI

请查看[YuiAPI的Github](https://github.com/yuiitsu/YuiAPI)

谢谢您的阅读

Yuiitsu 2018-11-08