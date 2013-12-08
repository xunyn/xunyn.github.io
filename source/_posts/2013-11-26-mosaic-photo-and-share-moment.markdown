---
layout: post
title: "制做自己的马赛克微信头像"
date: 2013-11-26 22:49
comments: true
categories: technical

---
##想法
2013年接近尾声了，有的媒体会用马赛克的方式，把一年中的大事件汇集到一张图片上。印象中天才少年[Matrix67](http://www.matrix67.com/)曾使用[Mathematica](http://zh.wikipedia.org/zh-cn/Mathematica)制作过自己马赛克头像，于是便有了用微信朋友圈里的图片来制作一个自己的微信头像。  
![马赛克头像](http://xunyanan.com/images/blog/mosaic_me_2011.jpg)  
觉得不像我，可以离开座位，退后一点，离显示器远点看这张图片。
<!--more-->  
##动手
使用到的工具：  
*Mac(os x 10.9)电脑(windows也可以做，替换相应的工具即可)  
*[Mathematica 9.0](http://www.wolfram.com/mathematica/)  
*[Chrome](http://www.google.cn/intl/zh-CN/chrome/) and [batch image download](https://chrome.google.com/webstore/detail/bulk-download-imageszig/gfjhimhkjmipphnaminnnnjpnlneeplk)  
*Preview and [A better finder rename](http://ifunapple.com/2013/07/a-better-finder-rename-9/) 
###筹集素材
制作马赛克头像的第一步是什么？要有足够的马赛克（心中无码可跳过），脑子的想法便是微信朋友的头像，有足够的好友数量，便有很多素材，这些素材，还是有统一尺寸的。要获得这些素材，有几个方法。
####从手机中获得素材
从手机将微信App导出，从中找到头像文件夹。这是第一想法，没试成，用的iPhone手机，升了iOS7.0，越不了狱，看不到微信的文件夹。android的童鞋可以试下，但有可能腾讯没把素材按JPEG放到文件，可能放到SQLite里，这样的话，这个方法就用不了了。  
####从微信网站下载素材
[微信新推出了网页版](https://wx.qq.com)，可以做的事情就多了一些。[打开微信网页版后]((https://wx.qq.com)),用手机扫描二维码登陆后，可以看到微信网页版的聊天界面，看到朋友们的信息。在这里我走个小弯路（点击跳过此段内容），这个弯路是用[Javascript](http://www.w3school.com.cn/js/)和[nodejs](http://nodejs.org/)来完成从网站扒朋友头像的任务。
#####JS获得头像url代码
```js 
function a () {var a = document.images; for(i=0;i<a.length;i++){ if(a[i].getAttribute("hide_src")){console.log(a[i].getAttribute("hide_src"))} } } a(); 
```
#####使用nodejs的request下载素材到本地文件
```js
// Dependencies 
var fs = require('fs'); 
var url = require('url'); 
var http = require('http'); 
var exec = require('child_process').exec; 
var spawn = require('child_process').spawn; 
// App variables 
var file_url = 'http://xunyanan.com/images/blog/PowerOfNow.png'; 
var DOWNLOAD_DIR = './';
// We will be downloading the files to a directory, so make sure it's there 
// This step is not required if you have manually created the directory 
var mkdir = 'mkdir -p ' + DOWNLOAD_DIR; 
var child = exec(mkdir, function(err, stdout, stderr) { 
    if (err) throw err; 
    else download_file_httpget(file_url); 
});  
// Function to download file using HTTP.get 
var download_file_httpget = function(file_url) { 
var options = { 
    host: url.parse(file_url).host, 
    port: 80, 
    path: url.parse(file_url).pathname 
}; 
var file_name = url.parse(file_url).pathname.split('/').pop(); 
var file = fs.createWriteStream(DOWNLOAD_DIR + file_name); 
http.get(options, function(res) {
    console.log(res.statusCode);
         console.log(res.data);
    res.on('data', function(data) { 
            file.write(data); 
        }).on('end', function() { 
            file.end(); 
            console.log(file_name + ' downloaded to ' + DOWNLOAD_DIR); 
        }); 
    }); 
};
```
#####一切准备妥当，发现微信的链接是https的，直接用url是新建http连接是获得不了东西的。这就和泡好了方便面，发现没配叉子一样。就换了条其他的路，放代码出来，大家可以研究下，如何用脚本建立上https的请求，毕竟脚本的方法能控制更多东西。
####新的道路，使用[Chrome](http://www.google.cn/intl/zh-CN/chrome/) and [batch image download](https://chrome.google.com/webstore/detail/bulk-download-imageszig/gfjhimhkjmipphnaminnnnjpnlneeplk)插件
如果去年抢过火车票，那估计电脑里都还有Chrome，[batch image download](https://chrome.google.com/webstore/detail/bulk-download-imageszig/gfjhimhkjmipphnaminnnnjpnlneeplk)比较好用，安装完成后便在右键菜单中可以看到。就是要注意，要先把朋友列表拖动一下，让所有的朋友头像都在网页上刷新出来，这样才可以下载全部的。
###整理素材
现在有了几百张朋友的头像，Mathematica需要的是统一尺寸的正方形jpg图像，大概每张图片的像素在20\*20~~40\*40之间,要先整理一下图片。  
####首先删除不是正方形的图像 
里面会有群组的头像，这个也可以选择删掉 
####重命名整理图片（可跳过）
1. 使用shell 命令批量重名  
```
j=100;for i in *;do let j+=1;mv "$i" "${i%.*}$j.jpg";done 
```
2. 使用[A better finder rename](http://ifunapple.com/2013/07/a-better-finder-rename-9/)  
  
####将132\*132的头像图片像素变小
推荐使用Mac自带的[atuomator](http://zh.wikipedia.org/zh-cn/Automator)做这项任务，通过鼠标拖动，就能安排这个小机器人给你干活，是见很有意思的事情。对于[自定义缩放图像](http://www.macx.cn/thread-2030978-1-1.html)这里有一份详细的教程。  
在脑海中先好要让机器人做什么，比如第一步先把图片拷贝到每个目录，然后在把这些图像尺寸缩小。编辑好flow后，保存好后，便可以通过右键看到自己命名的操作，然后选中文件操作。用完之后，如果不想右键点击后，还出现自定义的操作，可以在automator删除services工作流，进到目录[username]/Library/Services下删除对应名字的workflow，就可以了。
####制作黑白素材
将所有的头像图片，制作一份黑白素材，放在一起组成新的图像库。从Mathematica生成马赛克图片的上说，素材图库中图片越多，然后有黑白色图片低对比度图片，生成出来的头像，会效果更好。每一个素材库的图片都会是当一个像素去对应到最终生成的头像图片上。制作黑白素材的方法有三种方法：  
1. automator小机器人可以做  
2. 使用priview调节饱和度(Saturation)  
3. 使用photoshop调节颜色通道
效果上从上到下越来越好，功夫上也是从上到下越费越多。建议第二种。
###制作图片
制作马赛克图片我们用的是[Mathematica 9.0](http://www.wolfram.com/mathematica/) ([用R语言的看这里](http://www.bjt.name/2011/06/mosaic-plot-with-r/))。先看看下这经典的三行代码，向[Theodore Gray](http://blog.wolfram.com/2008/05/02/
making-photo-mosaics/)致敬。  
```  
imagePool = Map[With[{i = Import[#]}, {i, N@Mean[Flatten[ImageData[i], 1]]}] &, 
   FileNames["/Users/xunyanan/Desktop/webwx_img/*.jpg"]];
closeMatch[c_] := 
 RandomChoice[Nearest[imagePool[[All, 2]] -> imagePool[[All, 1]], c, 20]]
ImageAssemble[ 
 Map[closeMatch, 
  ImageData[Import["/Users/xunyanan/Desktop/2011me.tiff"]], {2}]]
```  
第一行就是把建立了一个制定路径的图像库;第二行写了个匹配函数，函数的内容是找出最接近的20个图片，并随机选择;第三行是导入目标图片，并调用函数closeMatch生成图像结果。
####目标图片tiff格式的头像照片
找一张自己的头像图片，然后把它处理成20*20像素的tiff格式(**注意在保存tiff格式时，一定不要勾选Alpha通道**)。如果选择像素大于20的图片，生成时间会长一些。
####生成马赛克头像
在[Mathematica 9.0](http://www.wolfram.com/mathematica/) 新建note book，然后，将三行代码拷入其中，并把路径改成自己的文件位置。右键选择Evaluation cell，便可看到结果。
##分享到微信朋友圈
可以用[微信的开放平台](http://open.weixin.qq.com/intro/)把自己的头像分享回微信。我的[个人博客网站](http://xunyanan.com/blog/2013/11/16/build-octopress-blog/)用的的是加网分享，点击分享，扫描二维码即可完成。
###在个人博客网站添加加网的方法是
####注册加网后，获得加网生成的js
```html
<!-- JiaThis Button BEGIN -->
<script type="text/javascript">
var jiathis_config = {data_track_clickback:'true'};
</script>
<script type="text/javascript" src="[your url]" charset="utf-8"></script>
<!-- JiaThis Button END —>
```
将此段代码添加到个人网站中，[详见这篇](http://xunyanan.com/blog/2013/11/16/build-octopress-blog/)。
