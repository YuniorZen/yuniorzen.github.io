---
title: CenterOS安装Puppeteer踩坑记录分享
date: 2019-12-16 16:27:30
categories: NodeJS
tags:
    - Puppeteer
    - node安装puppeteer
    - center安装puppeteer
---

最近闲来无事，决定写个爬虫抓抓玩，用request+cheerio的方式已经没有新鲜感，那就试试谷歌浏览器推出的神器Puppeteer,官方描述如下：
>Puppeteer is a Node library which provides a high-level API to control Chrome or Chromium over the DevTools Protocol. Puppeteer runs headless by default, but can be configured to run full (non-headless) Chrome or Chromium.

- 生成页面的屏幕截图和PDF。
- 抓取SPA并生成预先呈现的内容（即“SSR”）。
- 自动表单提交，UI测试，键盘输入等。
- 创建一个最新的自动化测试环境。使用最新的的JavaScript和浏览器功能，直接在最新版本的Chrome浏览器中运行测试。
- 捕获您网站的时间线跟踪，以帮助诊断性能问题。

你可以在浏览器中手动完成的大部分事情都可以使用Puppteer完成，但是我们今天不说怎么入门，况且很好上手。

## 安装步骤
```
//全局安装  #方便其余项目使用
npm install -g puppeteer
```
新建test.js，coding
```
//使用方法  #生成页面截图
const puppeteer = require('puppeteer');
(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://baidu.com');
  await page.screenshot({path: 'baidu.png'});
  await browser.close();
})();
```
执行生成截图
```
node test.js
```
然后我们看下，ok，目录下生成一张baidu.png!!!
*** 有这么容易，我就不分享了，见到那张截图是三天之后的事情，且听我细细道来。*** 

## 真实情况是这样的，发生在center6.7服务端
```
npm install -g puppeteer
```
安装这个puppeteer本地很快1分钟的样子，无奈服务器网络问题安装到40分钟时不动了，出于信任我又等了它十分钟！
最后还是没有成功，**卧槽 无情！**
然后切换国内淘宝镜像,118mb,很快1分钟***安装完成，
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install -g puppeteer
```
高高兴兴的执行node test.js
```
(node:20042) UnhandledPromiseRejectionWarning: Error: Failed to launch chrome!
xxx/xxx/node_modules/_puppeteer@1.19.0@puppeteer/.local-chromium/linux-674921/chrome-linux/chrome: error while loading shared libraries: libatk-bridge-2.0.so.0: cannot open shared object file: No such file or directory
```
**卧槽 无情！**
Chromium执行需要安装特定依赖，搜索一番去官方找了下依赖，然后执行yum install，拭目以待吧！
```
#依赖库
yum install pango.x86_64 libXcomposite.x86_64 libXcursor.x86_64 libXdamage.x86_64 libXext.x86_64 libXi.x86_64 libXtst.x86_64 cups-libs.x86_64 libXScrnSaver.x86_64 libXrandr.x86_64 GConf2.x86_64 alsa-lib.x86_64 atk.x86_64 gtk3.x86_64 -y
```
```
No package gtk3.x86_64 available.
```
**卧槽 无情！**
我又搜索了很久，各种大佬各种说，复制黏贴一把梭，最后还是无情！
*** 此时已是凌晨2点，算了还是睡觉吧 ***

两天后点一个夜里... 我又没事啦，想起了无情，操起键盘又是一堆检索！
无果，我又运行了下 node test.js
>libatk-bridge-2.0.so.0: cannot open shared object file: No such file or directory

回到问题本身，我换成检索词 *** libatk-bridge-2.0.so.0 *** 再次在知识的海洋里遨游
又过了许久，我翻到一篇也是归因安装依赖的问题，只不过依赖是其它应用copy过来，这不正符合no package的我嘛，又是一顿梭！！！
```
#1 检测缺失的依赖包
ldd chrome | grep not
```
```
# 检测结果
libatk-bridge-2.0.so.0 => not found
libgtk-3.so.0 => not found
libgdk-3.so.0 => not found 
libGL.so.1 => not found
```
```
#2 查找云端哪些软件下面包含对应的依赖包
yum provides libatk-bridge-2.0.so.0
```
```
...
firefox-60.1.0-6.el6.centos.i686 : Mozilla Firefox Web browser
Repo        : updates
Matched from:
Filename    : /usr/lib/firefox/bundled/lib/libatk-bridge-2.0.so.0

#这个就是我们要找的可用的依赖包
firefox-60.1.0-6.el6.centos.x86_64 : Mozilla Firefox Web browser
Repo        : installed// 已安装状态
Matched from:
Filename    : /usr/lib64/firefox/bundled/lib64/libatk-bridge-2.0.so.0
```
>注意：如果不是已安装状态，要先安装一下对应的软件名，

```
#3 复制
cp /usr/lib64/firefox/bundled/lib64/libatk-bridge-2.0.so.0  usr/lib64/
```
找到对应的可用的依赖包后，将依赖包复制/映射到对应的动态共享库目录中，一般是/usr/lib64/目录，复制/映射执行一个就可以了。
如上，依次安装其它依赖，再一次满怀期待的执行 node test.js
```
(node:32351) UnhandledPromiseRejectionWarning: Error: Failed to launch chrome!
/usr/local/lib/node_modules/puppeteer/.local-chromium/linux-706915/chrome-linux/chrome: /lib64/libc.so.6: version `GLIBC_2.14' not found (required by /usr/local/lib/node_modules/puppeteer/.local-chromium/linux-706915/chrome-linux/chrome)
/usr/local/lib/node_modules/puppeteer/.local-chromium/linux-706915/chrome-linux/chrome: /lib64/libc.so.6: version `GLIBC_2.15' not found (required by /usr/local/lib/node_modules/puppeteer/.local-chromium/linux-706915/chrome-linux/chrome)
/usr/local/lib/node_modules/puppeteer/.local-chromium/linux-706915/chrome-linux/chrome: /lib64/libc.so.6: version `GLIBC_2.16' not found (required by /usr/local/lib/node_modules/puppeteer/.local-chromium/linux-706915/chrome-linux/chrome)
```
怎么办？继续检索啊！
了解到这是chrome运行需要更高版本的libc，涉及底层库的升级，有很多反馈升级不当导致很多系统命令不可用！** 卧槽 无情！ 吓到睡觉！！** 


center os 安装puppeteer ---> cnpm 安装  ---> fail lauch chrome ---> chrome 依赖没安装 ---> yum install no package
---> 找其它依赖的cp /usr/lib64/ --->/lib64/libc.so.6: version `GLIBC_2.16' not found --->


## 最后
