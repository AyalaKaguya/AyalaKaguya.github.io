+++
title= '从Batch到JAVA Script，NowChat的脱胎换骨之路！'
author= 'AyalaKaguya'
date = "2020-02-22"
tags = [
    "nowchat",
    "即时聊天",
    "website",
    "development",
]
categories = [
    "开发",
]
keywords = [
    "nowchat",
    "即时聊天",
]
+++
> ❗ 警告
>
> 由于小站于近期更新，导致部分或全部功能无法使用，将会陆续恢复，特此告知。

## 前传：

![face](/images/nowchat/fd9fdd3f8794a4c2de693c1903f41bd5ad6e3955.jpg "face")

点击[这里][1]，查看首发帖。 

  * 2018-08-14 21:20 NowChat Batch Edition正式在百度bat吧公布
  * 2018-08-14 21:31 与吴先森交涉，决定使用自己的服务器
  * 2018-08-14 21:37 用户登陆的种种设想均已完备
  * 2018-08-14 21:53 决定使用GardensKernel进行聊天室开发
  * 2018-08-15 12:02 登陆部分完成
  * 2018-08-15 18:15 对所有的错误指定了错误页
  * 2018-08-15 18:22 更新了账号生成的规则，添加了密码保护
  * 2018-08-18 23:45 完成公共聊天室
  * 某年月日 GK开发到可以使用，准备让NowChat接轨
  * 某年月日 NowChat接轨完成，拥有较完美的GUI
  * 某年月日 为NowChat开发了一些GK专有模块
  * 某年月日 我的硬盘第一次炸
  * 某年月日 我开学了
  * 某年月日 2020年寒假，我重装了系统，但GK和NC开始鸽了
  * 某年月日 我的系统又一次炸，当天重装系统后心血来潮更新NC，但是排版爆炸
  * 2020-02-15 我通过Magic方法找到了Goeasy，正式开始了NC web的开发
  * 2020-02-17 NowChat Web Edition第一个预览版开发完成

## 正文：

### CMD版NowChat为什么弃坑？

![face](/images/nowchat/face.png "face")

这是最后一个CMD版本的NowChat，因为高三的缘故鸽了半年，在之前的帖子中rain提到，我正在开发其他东西，而这“东西”，正是“Gardens Kernel”（以下简称 GK），开发GK的初衷是简化复杂bat项目的构建，而GK正拥有这些特性： 

  1. 标准ini的读取 
  2. 众多的模块（包括对第三方的调用） 
  3. 完整的外部调用能力（调用里面的模块，并且返回） 
  4. 好用的Debug系统 
  5. windows平台统一化 

随着GK开发进度的推进，终于迎来了在TE内部的测试，但是测试结果不太理想，三个演示demo中最重要的那个出现了问题，而这问题只有在我的电脑上才正常，测试结果表明，在我的CMD中，没有字符间距，而其他人那里均出现夸张的排版问题，于是GK的开发便告一段落（其实是我自愧不如山归山的“batchhander”）。  
后来在我某次重装系统过后，我再一次打开了NowChat，排版问题出现了： 

![error](/images/nowchat/error.png "error")

这可能是导致NowChat工程项目再一次搁浅的间接原因，还有一部分原因是因为学业繁忙。

### WEB版NowChat，NC的春天？

不过没想到今年的寒假会那么的长，于是我便开发了NowChat的WEB版，不过这个WEB版只经历了三天左右的开发周期，经由TE和init的内部测试，尚且有许多bug（比如说网页劫持啊，登陆故障啊，移动设备兼容啊等等）。

开发过程中，我依然采用[MDUI][2]作为网页框架 ,而网络部分使用了基于WebSocket的[Goeasy][3](话说回来，Goeasy开发聊天室是真的方便！)，至于排版，emmmmm&#8230;&#8230;不喜勿喷！我最终的目标是在小屏设备上，这个聊天室可以撑满整个页面，目前宽度已做了响应式，但是影响原先排版，高度打算让js来，但如果MDUI有原生解决方案就最好了。

基于HTML5重做的NowChat具有以下优势：

  1. 多达60个用户的并发网络（Goeasy免费版）
  2. 信息传播达到毫秒级
  3. 响应式

![error](/images/nowchat/NCEB.png "error")

图片中的Demo网址可以直接访问，如果有问题，欢迎在评论区反馈，此项目开源（MIT协议），GitHub仓库访问地址： 

[https://github.com/AyalaKaguya/NowChat](https://github.com/AyalaKaguya/NowChat)

小站官方聊天室地址：[NowChat][4] （停止运营）

 [1]: http://tieba.baidu.com/p/5840318345
 [2]: https://mdui.org/
 [3]: https://goeasy.io/
 [4]: https://papernote.cn/pages/NowChat/