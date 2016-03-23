---
layout: post
title:  "angular material md-slider实现基本原理"
date:   2016-01-16 00:50
categories: ngMaterial slider md-slider
---


## background
最近因项目原因接触了下[angular material][angular material], 其中重点使用了一下[md-slider][md-slider]的模块。

再加之以前都没有看过相关slider的源码，这次就顺手撸了一遍它的实现代码，在这记一下实现原理，以后万一需要重新造轮子的时候可以回顾一下。

这篇文章也会简单提及一下[坐标系bug][issue7569]及[md-reverse的功能拓展][issue7666]。


## 基本原理


### 思路整理

显示的东西就这么多，然后我们想一下，实现slider最最重要的是什么。

我觉得应该是*当前值与按钮的显示位置始终保持对应*。这也是md-slider的核心。

如果我们建立了当前值与按钮显示位置之前能够互相转换的函数，那么我们就应该可以简单的实现slider的功能了。（当然实现这个转换关系也不难...）

如果当前值在外部更新，我们只需要根据当前值计算出按钮应该出现的位置，并改变按钮位置即可。

那么如果反过来，用户可以通过slider直接改变当前按钮位置，从而计算出当前值应该改变为多少。

* 通过鼠标改变位置: 拖动，直接点击某一位置

* 通过键盘改变位置: 用户按上下左右


### 显示元素

以一个vertical的slider的html为例

{% highlight html %}
<md-slider ng-model="vol" min="0" max="100"> 
    <div class="_md-slider-wrapper">
        <!-- position: relative -->
        <div class="_md-slider-content">
            <!-- 轨道 position: absolute -->
            <div class="_md-track-container">
                <div class="_md-track"></div>
                <!-- 按钮另一侧的颜色可以设置为不一样的颜色 -->
                <div class="_md-track _md-track-fill" style="height: 55%;"></div>
                <div class="_md-track-ticks"></div>
            </div>
            <!-- 按钮 position: absolute -->
            <!-- 注意此处style, 是通过js计算出位置设置上去的 -->
            <div class="_md-thumb-container" style="bottom: 55%;"> 
                <div class="_md-thumb"></div>
                <div class="_md-focus-thumb"></div>
                <div class="_md-focus-ring"></div>
                <div class="_md-sign">
                    <span class="_md-thumb-text">55</span>
                </div>
                <div class="_md-disabled-thumb"></div>
            </div>
        </div>
    </div>
</md-slider>
{% endhighlight %}

md-slider还有一个md-discrete模式，是指在有拖动时将按钮上的数字放大，可以更容易的看到当前值。


### 当前值与位置的转换

当前值则是ng-model传出来的`value`, 位置在这里不是用直接的位置，而是用百分比`percent`来表示。
请回看上面的html片段，按钮的位置改变由绝对位置的bottom: 55%控制，此处的55则是这个`percent`。
不过看另一外一个地方可能会更好理解，那就是`_md-track-fill`，**此段位置即代表从当前值`value`到最小值`min`的距离**，只不过设置style时使用的百分比`percent`。

我们来看`valueToPercent`, 其实也就是在通过值来计算刚才所说的百分比`percent`。
`(val - min)`是上述`_md-track-fill`到底端的距离所代表的值，`(max - min)`是`track`总长所代表的值。
{% highlight js %}
    function valueToPercent( val ) {
      return (val - min)/(max - min);
    }
{% endhighlight %}
这样我们就有了值到位置的转换，当从外部获取值的时候，通过值计算出百分比，设置style，即可以改变按钮所在的位置了。

我们再来看一下反向转换,

`percent * (max - min)` 此段就是`_md-track-fill`的长度代表的值，再加上最小值，则就是当前值了。

{% highlight js %}
    function percentToValue( percent ) {
      return (min + percent * (max - min));
    }
{% endhighlight %}

这样`percent`和`value`之间的转换就完成了~~

但是...是不是觉得少了点什么? 说了半天我们都还没有说到**位置`position`**呢！！

那么，我们来看看没有位置是不是也没有关系呢，毕竟百分比也就是用来通过style而改变位置的。

value值从外面传进来的时候，可以直接得到百分比，然后设置style，即可改变按钮的位置。

键盘上下左右，可以直接改变value值，再通过上面方式，也可以算出位置。

鼠标拖动或直接点击，嗯....好像我们不知道值改变了多少，也没办法直接拿到百分比。所以我们就得用其它手段，也就是我们要引入位置`position`了。

鼠标拖动或直接点击，我们都可以通过`MouseEvent`来获取鼠标的坐标系位置`ev.point.y`或`pageY`，其实此时鼠标位置就应该是按钮的新的位置。

如果现在我们能够从鼠标的position计算出`percent`，那么就可以再通过`percentToValue`得到新的值`value`了。


{% highlight js %}
    // 此片段稍作修改，去掉了水平slider的相关信息
    function positionToPercent( position ) {
      // 传入的position是鼠标的坐标位置
      // sliderDimensions 其实就是 trackContainer.getBoundingClientRect();
      var offset = sliderDimensions.top;
      var size =  sliderDimensions.height;
      // 鼠标坐标 - track顶部(左部)的坐标 就是 track总长 - _md-track-fill的长度
      // 也就是 按钮到轨道顶端的距离
      // cacl 就是我们要的百分比了, 只不过我们需要的百分比是按钮到底端这段所占的百分比，而非按钮到顶端的，所以后面需要1 - cacl
      var calc = (position - offset) / size;

      return Math.max(0, Math.min(1, 1 - calc));
    }
{% endhighlight %}

这样我们就可以通过 鼠标位置 到 百分比 再到 值 的转换了。

再看一下，我们所需要的功能是不是都可以实现了?

* 直接给值，js改变位置
* 通过鼠标改变位置: 拖动，直接点击某一位置
* 通过键盘改变位置: 用户按上下左右


## 坐标系
我们再回过头来看一下`positionToPercent`这段代码，这里有个很不巧的事情。

我们来说一说[不同坐标系的bug][issue7569]，[这篇文章][coordinate]介绍的坐标系知识很清晰易懂。（没错，我就是看这篇文章学习的）

position都是通过`ev.pointer.y`获取**document坐标系**的值, 而`sliderDimensions`是通过`getBoundingClientRect`获取的**window坐标系**的值。
所以在`(position - offset)`的时候，应该先将两个值统一到相同的坐标系再进行计算。

> Document coordinates are window coordinates plus scroll
> document坐标系 = window坐标系 + 滚动条

## md-reverse
md-slider vertial: 上面是max值，下面是min值
然后我需要的是要做一个分页的滚动条，需要上面是最小值，下面是最大值。
所以这里就需要稍作改动，其实改动可以特别简单。

{% highlight js %}

    function percentToValue( percent ) {
      var adjustedPercent = reverse ? (1 - percent) : percent;
      return (min + adjustedPercent * (max - min));
    }

    function valueToPercent( val ) {
      var percent = (val - min) / (max - min);
      return reverse ? (1 - percent) : percent;
    }

{% endhighlight %}
完整代码可以到[这里][md-reverse-pr]查看


[angular material]: https://material.angularjs.org
[md-slider]: https://material.angularjs.org/latest/demo/slider
[issue7666]: https://github.com/angular/material/issues/7666
[issue7569]: https://github.com/angular/material/issues/7569
[md-reverse-pr]: https://github.com/soooooot/material/commit/0107f0fb64d1c4f8d57832ef78054edf1b1d113d 
[coordinate]: http://javascript.info/tutorial/coordinates

