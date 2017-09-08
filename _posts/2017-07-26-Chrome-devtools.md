---
layout: post
title: 如何用好 Chrome devtools
categories: 工具/软件/技巧
description: Chrome devtools
tags: chrome Chrome devtools
---
* content
{:toc}
<div class="postImg" style="background-image:url(http://ovl1kjv88.bkt.clouddn.com/media/chrome-devtools.jpeg"></div>

相信绝大部分的前端是使用Chrome进行开发的，一方面Chrome浏览器确实做得好，更重要的一方面是因为Chrome有一个无法匹敌的调试工具。用好这个调试工具可以提高编程效率，帮助快速地定位问题。




## 01 打印
### （1）console.table
最常用的打印是console.log，console.log有时候打印一些复杂的数据结构显得有点吃力，如打印一个元素是object的数组：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012060131331.jpg)



为了查看每个数组的元素，必须得一个个展开，就显得有点麻烦了，其实可以用console.table：

![15012060543721](http://ovl1kjv88.bkt.clouddn.com/media/15012060543721.jpg)


瞬间就变得非常清爽，同时console.table还支持打印对象属性，如下的Student对象，有name和score两个属性：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012061528112.png)




### （2）console.dir
console.log是侧重于字符串化的打印，而console.dir能递归打印对象的所有属性，如下打印一个dom结点：

![](http://ovl1kjv88.bkt.clouddn.com/media/15012061662568.png)

console.log把它的html打印出来了，而console.dir把它的所有属性打印出来，方便进行检查。
### （3）打印带样式
经常会看到有些网站会在控制台打印一些提示语，并且这些提示语还带样式，这个是用%c加上的样式：

![](http://ovl1kjv88.bkt.clouddn.com/media/15012062185118.png)


## 02 检查没有用的CSS/JS



Chrome 49新增了一个功能，能够检查页面上的CSS/JS没有用到的比例，如下打开devtools的Coverage标签栏，然后点记录按钮，刷新页面，页面加载完之后，点击停止。就会显示页面用到和没用的CSS/JS占比。没有用到的用红色表示，用到的用绿色表示：

![](http://ovl1kjv88.bkt.clouddn.com/media/15012062628062.png)

可以看到第二个CSS文件有大部份是没用到的，JS也有很大的比例没有用到。在上面中间的窗口会把具体没使用到的代码标红，就能知道具体哪些代码没有被用到。可以看到，CSS有一半没有用到，因为有一些是小屏的响应式CSS，大屏没用到。另一些是common-chunk提出来的，所以可以考虑把大小屏的CSS分开（但是不适合于内联Style），如下用媒体查询去加载不同的CSS：
`<link rel="stylesheet" href="large.css" media="screen and (min-width:500px)">`
JS也是因为require了比较大的公有模块导致的，这些模块比较大，但是只使用了其中小一部分功能，可考虑把大模块拆细，但是粒化太细可能会增加复杂度，所以要权衡一下。
这些功能可以用Chrome Canary尝鲜版的使用。

## 03 截全屏的功能
除了Corverage之外，Chrome 49还新增了截全屏的功能：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012062982540.png)



就不用去装一个第三方的插件。

## 04 debugger
可以在代码里面写一句debugger，一旦运行到dubugger就会自动卡在那里进入调试模式，使用这个有两个好处，一个是不用手动去展开文件找到对应的位置，因为现在很多人都用第三方的打包，导致跑的代码和本地的代码行数不能保持一致，所以需要去搜索相应地代码，比较麻烦。另外一个好处是：当代码是一个很大的循环的时候，并且在特定的情况下代码会挂，由于要循环好多次，所以有时候不可能在循环里面打个断点，然后不断地跳到下一个断点直到出现问题。所以这时候怎么办呢？可以用条件结合debugger。如下代码：
```
var scores = [90,70,58,60, ...];
var newScores = [88, 55, 60, ...];for(var i = 0; i < scores.length; i++){    for(var j = 0; j < newScores.length; j++){        if(scores[i] !== newScores[j]){
            scores.push(newScores[j]);
        }
     }
}
```
发生了死循环，怎么定位在哪里出了问题呢？方法一在循环里面打个断点，一次次执行分析，直到发现问题，方法一有时候挺好用的，可能很快就可以发现问题了。方法二，发生死循环了肯定是i不断得在变大，但是为什么i不会停止变大，可以加个条件，当达到那个条件的时候进入断点：
      `if(i > 1000000){debugger}`
如上面第7行，当i大于1000000时进入断点，这个时候检查一下，发现scores已经变成了一个很大的array了：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012063749111.png)



所以可以进一步发现问题。这样就解决了大循环里面打断点调试的问题。
使用debugger还有一个小技巧——可以解决一个检查页面元素的问题。如下图所示，当hover到那个绿色的标签时，会出来个详情框：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012063960038.png)



现在我要检查这个详情框，但是我一检查，鼠标离开了，那个框也消失了，导致检查不了了，因为触发了mouseout事件，那怎么办呢？假设这个框是css的hover控制的，那么可以用控制台的伪类窗口，在:hover那里打个勾，就有hover的效果：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012064284480.png)



但是这个不是用css的hover实现的，所以没办法用这个。这里有一个小技巧，就是用debugger让页面卡住了，mouseout事件的响应函数就不会执行了。如下，先在左边的控制台点一下进入编辑模式，然后把鼠标挪到左边的页面的标签，让那个框出来，然后再在控制台输入debugger回车，这个时候页面就卡住了，接着就可以愉快地进行检查了，如下图所示：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012064459372.png)



## 05 IOS真机模拟
这个虽然和Chrome没关系，但是也可以提一下，Mac的XCode可以开一个ios系统，操作如下图所示，然后可以用Mac的Safari接连这个ios里面的Safari进行检查：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012064637322.png)


如下例子调试iPhone的Safari：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012064814217.png)

上面右图弹了一个原生的下拉，因为它是真的ios系统，这个的好处就在于不用老是连真机调试，适合于有Mac但是没有iPhone的同学，缺点是没办法和真机100%一样，例如它不弹键盘，它的输入是用电脑键盘的输入。不过它是100%的ios系统，另外需要装一个很大的XCode。

## 06 用console.trace追踪函数调用
现在遇到了一个问题，就是点击X按钮的时候重复发了两个请求，如下下图的红框：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012064998212.png)



当上一次请求还没完成又要发一个新的请求的时候，代码里面会把上一次请求abort掉，所以会看到上一个变红了：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012065202049.png)

但是这样还是会有问题，所以要看一下究竟是在哪里触发了两次请求。由于请求会走一个通用的发请求的接口，所以可以在那里追踪一下：
```
ajax: function(curPage){
    console.trace("search.js ajax");
    //other code
}
```
然后控制台就打印了两次trace：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012065343793.png)


分别点开这两次的代码，就会发现，两次触发分别是：一次是X的click事件调的搜索，另外一次是map的zoom_changed触发的搜索，知道了调用的地方就好办了，就可以做进一步分析，然后去掉其中一个。
另外一个看函数调用栈的地方是在右边的窗口：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012065490319.png)


调用栈最多只打印5个，有时候可能会不太够。

## 07 查看某个函数绑的事件
控制台Elements的下面有个Event Listeners，点最后面那个可以看到和它最相关的事件
![](http://ovl1kjv88.bkt.clouddn.com/media/15012065619103.png)


其次，用好快捷键可以事半功倍，常用的快捷键：
```
F10 下一步
F8 跳到下一个断点
command/ctrl + ;            step into进入函数执行
shift + command/ctrl + ; step out跳出当前函数
```

一个实例——研究一下鼠标hover的时候它的边界是怎么画出来的：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012065796738.png)


首先定位到mouseover事件，因为它必定是mouseover触发的：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012065934674.png)


一打开发现代码是压缩的：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012066063035.png)


压缩的代码可以点左下角的大括号，进行美化：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012066202521.png)



然后再用快捷键一步步的step over/step into，如果你不用快捷键老是一个个去点那个调试的按钮还是挺麻烦的。如果不小心过了，就重新来一遍。最后会找到在这个函数里面画的边界：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012066317050.png)


## 08 Open in Sources Pannel
在sources pannel面板如果要查看某个源文件的时候，需要一步步展开文件夹，这样比较麻烦，而在network面板里面可以用各种筛选，但是在network里面是不能打断点的，这个时候可以用右键，然后点击Open in Sources Pannel的功能：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012066451932.png)


就会在sources面板打印相应的文件，然后可以在sources里面进行调试了。

## 09 模拟断网做一些出错处理


devtools还支持模拟网络情况，例如可以模拟断网的情况，突然挂掉了会怎么样，然后相应的做一些出错处理：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012066591461.jpg)



如上图出错的时候给一个提示文案，同时恢复upload按钮，就可以让用户再重新上传。

## 10 研究重绘
devtools有一个Renders可以用来研究重绘，如下图右下角，把面板里面的勾都打上：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012066723912.jpg)


然后点击左边的页面的菜单按钮研究重排的情况。右上角黑色的框会显示当前的帧率。左边变蓝的区域表示需要进行重绘，由于要弹一个蒙层，所以页面可视区域进行重绘了。同时可以看到当前帧率是59，而且整一个过程帧率基本保持在55以上，因为这个是用transform做的动画，所以帧率比较高。再来看一个用postion做的动画：
![](http://ovl1kjv88.bkt.clouddn.com/media/15012066914661.jpg)


可以看到帧率跌到了45，并且有相当一小段时间帧率是在50以下，所以用postion做的动画流畅度没有transform的好。

## 11 用timeline看执行时间
如下图所示，可以查看很多有用的信息：

![](http://ovl1kjv88.bkt.clouddn.com/media/15012068119532.png)

这个我在《Effective前端6：避免页面卡顿》有详细的介绍。

## 12 检查内存泄漏
基本上只要存在一个引用就不会进行GC回收，有些DOM节点没有append到DOM，但是存在引用指向它，它就是一个分离的DOM结点，这个时候发生了DOM内存泄漏，如下面的代码：
```
var detached = null;
button.on(“click”, function(){
    detached = document.createElement(“div”);
})
```
由于闭包里面有一个变量指向一个分离的DOM结点，所以创建的那个变量指向的内存空间不会被释放掉。这个时候可以拍一张内存堆的快照，Chrome会帮你把这些分离DOM结点用黄色标出来。
先切到Profile标签，选中Take Heap Snapshot选项，然后点击Take Snapshot按钮：

![](http://ovl1kjv88.bkt.clouddn.com/media/15012068002677.png)

然后就会把当前内存的使用情况显示出来：

![](http://ovl1kjv88.bkt.clouddn.com/media/15012067920057.png)

在搜索框里搜一下detached，出来的结果里面红色的表示已经分离且没有引用，而黄色表示已经分离且有引用，所以重点是看这些黄色的。展开其中一个分析一下，可以看到这个HTMLDivElement有一个ImageShower里的$imgContainers指向它，所以导致它的内存空间不能被释放。具体看下代码可以看到这是DOM已经删了，但是图片懒惰加载里面没有清掉引用。所以解决办法是当删掉DOM节点时，把那个ImageShower里面的变量置为null，或者把整个实例对象置为null。

## 13 查看内存消耗
为了查看某个操作的内存消耗情况，可以用Record Allocation的功能记录某个操作内存的分配情况。

![](http://ovl1kjv88.bkt.clouddn.com/media/15012067826576.png)

点了start按钮之后，进行一个操作，例如弹一个框，然后点击停止记录的图标，就会出来使用情况的分析，如下图所示：

![](http://ovl1kjv88.bkt.clouddn.com/media/15012067742446.png)

点开最上面的Object，可以看到这个Object数组开销了553K的内存（下图倒数第二列第一行）：

![](http://ovl1kjv88.bkt.clouddn.com/media/15012067655031.png)


展开其中一个Object，可以看到它是一个jQuery对象，每一个消耗了88Kb：

![](http://ovl1kjv88.bkt.clouddn.com/media/15012067573461.png)


而总的内存可以用Chrome的任务管理器查看：

![](http://ovl1kjv88.bkt.clouddn.com/media/15012067502174.png)

可以看到当前页面消耗了193Mb的内存。所以如果当你觉得页面的内存比较大的时候，或者重复某个操作之后页面的内存不断增大，就可以用这种方法分析一下。

## 14 垃圾回收
垃圾回收可以在timeline里面查看，如下图所示：

![](http://ovl1kjv88.bkt.clouddn.com/media/15012067424282.png)

蓝线那里JS Heap骤降，说明进行了一次垃圾回收，如果发生得比较频繁的话，可能会有问题。

## 15 查看连接时间
如下图所示：DNS解析花了254ms，建立tcp连接花了1.98s，建立https连接花了1.69s，从建立完连接到接收到第一个字节的数据(TTFB，Time To First Byte)等了4.3s，下载时间花了306ms（基于某次实验）。


![](http://ovl1kjv88.bkt.clouddn.com/media/15012067330570.png)

另外图片的优先级会低于CSS/JS资源，并且同一个域最多只能同时加载6个资源，所以会有排队和等待时间，如下图所示：


![](http://ovl1kjv88.bkt.clouddn.com/media/15012067184535.png)

本文分析了一些devtools的一些比较好用的功能，结合自己的项目经验做了些实际的例子说明，希望对大家有帮助。


