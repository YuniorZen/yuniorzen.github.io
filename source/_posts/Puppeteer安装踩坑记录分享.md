---
title: CenterOS安装Puppeteer踩坑记录分享
date: 2019-12-16 16:27:30
categories: NodeJS
tags:
    - Puppeteer
    - node安装puppeteer
    - centeros安装puppeteer
---

最近闲来无事，决定写个爬虫玩，用request+cheerio的方式已没有新鲜感，那试试谷歌推出的神器***Puppeteer***。
- 生成页面的屏幕截图和PDF。
- 抓取SPA并生成预先呈现的内容（即“SSR”）。
- 自动表单提交，UI测试，键盘输入等。
- 创建一个最新的自动化测试环境。使用最新的的JavaScript和浏览器功能，直接在最新版本的Chrome浏览器中运行测试。
- 捕获您网站的时间线跟踪，以帮助诊断性能问题。

你可以在浏览器中手动完成的大部分事情都可以使用Puppteer完成。但是今天不说Puppeteer怎么入门，来说说我是怎么被它无情折磨的。

## 安装步骤
> 全局安装  #方便其余项目使用

```
npm install -g puppeteer
```

> 新建test.js，编码实现生成页面截图

```
const puppeteer = require('puppeteer');
(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://baidu.com');
  await page.screenshot({path: 'baidu.png'});
  await browser.close();
})();
```

> 执行生成截图

```
node test.js
```
然后我们看下，ok，目录下生成一张baidu.png!!!
*** 这是想象的完美流程，这张截图我到现在还没有见到，且听我细细道来。*** 

## 安装Puppeteer的坑
> 全局安装puppeteer

```
npm install -g puppeteer
```
安装这个puppeteer本地很快1分钟的样子，无奈服务器网络问题安装到40分钟时不动了，出于信任我又等了它十分钟！最后还是没有成功，**卧槽 无情！**

>然后切换国内淘宝镜像,118mb,很快1分钟安装完成，

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install -g puppeteer
```

> 高高兴兴的执行node test.js

```
(node:20042) UnhandledPromiseRejectionWarning: Error: Failed to launch chrome!
xxx/xxx/node_modules/_puppeteer@1.19.0@puppeteer/.local-chromium/linux-674921/chrome-linux/chrome: error while loading shared libraries: libatk-bridge-2.0.so.0: cannot open shared object file: No such file or directory
```
**卧槽 无情！** Chromium执行需要安装特定依赖，搜索一番去官方找了下依赖。

> 执行yum install安装依赖库，拭目以待吧！

```
yum install pango.x86_64 libXcomposite.x86_64 libXcursor.x86_64 libXdamage.x86_64 libXext.x86_64 libXi.x86_64 libXtst.x86_64 cups-libs.x86_64 libXScrnSaver.x86_64 libXrandr.x86_64 GConf2.x86_64 alsa-lib.x86_64 atk.x86_64 gtk3.x86_64 -y
```
```
No package gtk3.x86_64 available.
```
**卧槽 无情！** 我又搜索了很久，各种大佬各种说，复制黏贴一把梭，最后还是无情！
*** 此时已是凌晨2点，算了还是睡觉吧 ***

***

两天后点一个夜里... 我又没事啦，想起了无情，操起键盘又是一堆检索！
>无果，我又运行了下 node test.js

```
libatk-bridge-2.0.so.0: cannot open shared object file: No such file or directory
```
回到问题本身，我又换回了检索词 *** libatk-bridge-2.0.so.0 *** 再次在知识的海洋里遨游
又过了许久我翻了到一篇，也是归因安装依赖的问题，只不过依赖是其它应用copy过来，这不正符合no package的我嘛，又是一顿梭！！！
> 检测缺失的依赖包

```
ldd chrome | grep not
```
```
# 检测结果
libatk-bridge-2.0.so.0 => not found
libgtk-3.so.0 => not found
libgdk-3.so.0 => not found 
libGL.so.1 => not found
```

> 查找云端哪些软件下面包含对应的依赖包

```
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
***注意：如果不是已安装状态，要先安装一下对应的软件名***

找到对应的依赖包后，将依赖包复制或映射到对应的动态共享库目录中，一般是/usr/lib64/目录，复制或映射可以。
> 复制包到 /usr/lib64/

```
cp /usr/lib64/firefox/bundled/lib64/libatk-bridge-2.0.so.0  /usr/lib64/
```

> 如上，依次安装其它依赖后，再一次满怀期待的执行 node test.js

```
(node:32351) UnhandledPromiseRejectionWarning: Error: Failed to launch chrome!
/usr/local/lib/node_modules/puppeteer/.local-chromium/linux-706915/chrome-linux/chrome: /lib64/libc.so.6: version `GLIBC_2.14' not found (required by /usr/local/lib/node_modules/puppeteer/.local-chromium/linux-706915/chrome-linux/chrome)
/usr/local/lib/node_modules/puppeteer/.local-chromium/linux-706915/chrome-linux/chrome: /lib64/libc.so.6: version `GLIBC_2.15' not found (required by /usr/local/lib/node_modules/puppeteer/.local-chromium/linux-706915/chrome-linux/chrome)
/usr/local/lib/node_modules/puppeteer/.local-chromium/linux-706915/chrome-linux/chrome: /lib64/libc.so.6: version `GLIBC_2.16' not found (required by /usr/local/lib/node_modules/puppeteer/.local-chromium/linux-706915/chrome-linux/chrome)
```
** 卧槽 无情！** 了解到这是chrome运行需要更高版本libc，涉及底层库升级，有反馈升级不当导致很多系统命令不可用的风险！** 吓到睡觉！！！** 

***

次日，我又想起了无情，再次检索一番，得知需要更新glibc到2.16，复制黏贴又是一梭。
> 下载，解压，编译等操作（用 root 全权操作，最后两步用时比较久）

```
wget https://ftp.gnu.org/gnu/glibc/glibc-2.16.0.tar.gz
tar -zxvf glibc-2.16.0.tar.gz
cd glibc-2.16.0 
mkdir build
cd build
../configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
make -j 8
make install
```

> 查看安装后的glibc有没有更新到2.16

```
strings /lib64/libc.so.6 | grep GLIBC
```
```
...
GLIBC_2.14
GLIBC_2.15
GLIBC_2.16
GLIBC_PRIVATE
```

> 更新完毕，最后一次执行node test.js

```
symbol lookup error: /usr/lib64/firefox/bundled/lib64/libcairo-gobject.so.2: 
undefined symbol: cairo_region_destroy
```
*** 卧槽，无情！睡觉去啦！ ***

***

待修复bug...


以下是假如修复啦：
*** 卧槽，终于跑起来啦！ *** 为自己鼓掌，好在能运行起来，真是折腾。

## 最后
如上分享，希望能帮助到遇到同样场景的你，无情就让我一人承受吧。
很多看起来正常的流程就是会出现你意想不到的意外，人生亦然，能做的只有坚持不懈的去寻找答案和解决问题，当然主要还是对linux不熟悉太菜，这又给我增加了学习linux的理由。