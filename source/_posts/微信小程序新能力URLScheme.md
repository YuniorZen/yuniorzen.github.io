---
title: 微信小程序新能力：URL Scheme，可从短信跳转小程序
date: 2021-03-10 15:16:30
categories: 小程序
tags:
    - 小程序
    - URLScheme
    - 短信跳转小程序
    - h5跳转小程序
---


最近小程序上线了一个超级流量的新入口：URL Scheme。通过小程序页面的URL Scheme，可以在短信、邮件或微信外部的网页中打开小程序。

那么如何实现呢？[官方文档](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/url-scheme.html)已经写的很清楚啦，这里简单介绍一下。

**首先，获取URL Scheme**，通过服务端接口可以获取打开小程序任意页面的URL Scheme，支持生成到期失效和永久有效的URL Scheme。
![](https://mmbiz.qpic.cn/mmbiz_png/1pod2O2UeC7WIdibfwnmPDsBQGicsZQK2CThM8hFx3zNicCbFUiaUiaZpbdbscA6oZSD0BLAXgClZ9KDa8XLKrq0uIw/0?wx_fmt=png)

然后，通过短信群发平台将获取的URL Scheme + 营销文案发送到用户的手机上。

最后，用户收到短信后，直接点击URL Scheme唤起微信，跳转到对应小程序页面，就是这么简单。
除此之外，还可以通过邮件或外部浏览器打开跳转小程序。

由于部分操作系统仍不支持直接识别URL Scheme，因此直接将Scheme发送给用户可能存在无法打开小程序的情况。
为此，我们可以先准备一个H5页面，再从H5页面跳转到URL Scheme实现打开小程序。
```
location.href = 'weixin://dl/business/?ticket= *TICKET*'
```
[H5的示例代码我已经更新到Github](https://github.com/YuniorZen/minicode-debug)，可以复用起来，基于[官方的案例](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/guide/staticstorage/jump-miniprogram.html)做了些改动，增加PC端打开时生成二维码方便手机扫码使用。

这次新能力的更新将使微信小程序不再局限于微信内部的流量，天花板被掀开啦。  
而且短信和邮件营销的触达成本非常低，营销成本的压低也会催生出很多新的流量玩法，我们敬请期待吧。