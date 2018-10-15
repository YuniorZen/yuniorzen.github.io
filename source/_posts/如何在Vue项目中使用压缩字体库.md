---
title: 如何在Vue项目中使用压缩字体库
date: 2018-09-10 16:27:30
categories: 前端技术
tags:
    - Vue
    - WebFont
    - 字体库
---


如题，在一些项目中经常会碰到在web端使用设计图上的特殊字体，目前已知的解决方案：
1. 购买字体库服务器，对方会提供个引入脚本置于你的站点中，不过天下没有免费的午餐。
2. 通过搜集项目需要的字型集，然后生成压缩后的字体库文件用于项目中，目前成熟的是使用字蜘提供插件可以自动生成。

作为一个创业团队显然不愿意花钱付费解决了，但是字蜘的也只能针对于静态的html提取的文字，现在的前端开发环境已经不在像以前单纯操作html文件，比如：vue项目中，我们通常的代码和文字都会写在.vue中，或者js文件中。
那么我想到是解决办法是：
> 首先获取到.vue、.js文件中字型集，然后使用字体库工具把你需要的文字生成压缩后的字体库文件，最后在项目中引用即可。

## 第一步：手写一个获取项目中字型集的node代码 getFonts.js
```
#源码地址：https://github.com/YuniorZen/js-tool/blob/master/getFonts.js

let fs=require('fs'),
    files,
    content=[];

let  walk=function(dir,exclude) {
    let results = [];
    let  list = fs.readdirSync(dir);
    list.forEach(function(file) {
        file = dir + '/' + file;
        let stat = fs.statSync(file);
        if(stat&& stat.isDirectory()){           
            results = results.concat(walk(file))
        }else{
            if(/.(vue|html|js)$/.test(file)){ //也可以是其它文件比如json，后缀名添加上即可
                results.push(file)
            }    
        } 
    });
    return results
};

((dir)=>{
    files=walk(dir);
    files.forEach(file=>{
        let text=fs.readFileSync(file);
        text.toString().split('').forEach(letter=>{
            if(/[a-zA-Z0-9\u4e00-\u9fa5]/.test(letter)&&content.indexOf(letter)==-1){
                content.push(letter);
            }
        })    
    })
    console.log(content.join(''));
})('./src');
// ./src 是项目目录地址，可以是绝对地址也可以是相对的，视getFonts.js运行的目录而定
```

## 第二步：用获取的字型集生成压缩的字体库文件，这里使用[fontmin](http://ecomfe.github.io/fontmin/)的win客户端；
![fontmin客户端](https://yuniorzen.github.io/assets/image/app.png)

## 第三步：把生成的字体库文件放在用到的项目中，@font-face声明字体并使用
```
 @font-face {
    font-family: "PingFangSC";
    src:url("../fonts/PingFangSC.eot"), /* IE9 */
        url("../fonts/PingFangSC.woff") format("woff"), /* chrome、firefox */
        url("../fonts/PingFangSC.ttf") format("truetype"), /* chrome、firefox、opera、Safari, Android, iOS 4.2+ */  
        url("../fonts/PingFangSC.svg#PingFangSC") format("svg"); /* iOS 4.1- */
    font-style: normal;
    font-weight: normal;
  }

```

***

最后这个方案仅适用于项目中文字较固定，不会经常变更的，不然每次变更新增字体都要手动跑一边，类似社交、论坛这种UGC项目就不要这样折腾了，还是老老实实的购买字体服务器吧！

