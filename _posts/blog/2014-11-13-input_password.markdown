---
layout: post
title: "手机端密码输入显示方式研究"
date: 2014-11-13 11:10:01
author: 李琨
categories:
- blog
- form
- input
img: post01.jpg
thumb: thumb01.jpg
---

前言

许多人应该都注意到在手机端输入密码时，会先显示短暂时间的明文，直到下一位密码输入或显示超时（3秒左右），再变成替换符（通常是实心圆）。由于大量的移动端设备以触屏取代了实体按键，没有了按下按键的触觉反馈，用户很容易担心自己刚刚输入的密码是对是错，造成不必要的重复输入。此种交互方式可以让用户在输入密码时可以再次确认，确实是一种好的体验。

pwd1.gif


但是在特定情况下，比如输入支付密码、解锁手机密码或其他重要密码时，人们对此类信息的安全程度是非常重视的。这里想到有异曲同工的九宫格图案密码，在安卓手机解锁时，可以在系统设置中显示输入的图案或者隐藏。如果在手机端web中能在适当的情况下改变显示状态，一定是件美妙的事。

那么问题来了：如何在手机web浏览器下使密码不显示明文而直接显示为替换符？

在iOS开发中，可以用secureTextEntry = YES使UITextField设置为密码形式。类似的，web下也有一个CSS属性使常规文本显示为密码形式：
-webkit-text-security: none | circle | disc | square

四个属性值依次表示：无（正常文本），空心圆，实心圆，实心方块

在PC上的浏览器中查看显示效果，结果是除了<input type="password">，其他常规文本以及<input type="text">等都完整支持该属性。而<input type="password">，由于该元素本身的语义，浏览器不允许修改其文本表现形式，所以-webkit-text-security对它无效。下面分别是safari和chrome的默认样式：

屏幕快照-2014-11-10-下午6.04_.jpg


在iOS的safari上查看显示效果，属性值为none时显示为正常文本，circle、 disc和square都显示为实心圆；当然<input type="password">仍不允许修改其样式。更重要的是：设置了-webkit-text-security：disc的元素输入文本时，仍是先显示明文再变成实心圆。所以，用其他元素代替<input type="password">来实现，这一方法也无情地被浏览器否掉。

解决方案

至此，我们被浏览器逼上绝路，也只能想个“下策”来实现想要的效果，那就是用“障眼法”，大致思想是：一个input元素用来输入密码，但不显示出来；通过js把这个input里输入的密码传给另一个input元素负责输出。输出的密码其实并不是通过用户输入，而是js赋值取得，只有这样，密码才不会被浏览器展示为正常输入的情况。具体代码如下：

（HTML）
<input type="password" class="pwd" id="input_pwd">
<input type="password" class="pwd" id="show_pwd">

（CSS）
.pwd {position:absolute;top:0;left:0;}
#input_pwd{opacity:0;z-index:2;}
#show_pwd{opacity: 1;}

（JavaScript）
var ipwd = document.getElementById("input_pwd"),
spwd = document.getElementById("show_pwd");
window.onload = function() {
    ipwd.oninput = function() {
        spwd.value = ipwd.value;
    }
}

实现效果如下图：

pwd.gif


这个方法并不尽善尽美，从效果图可以看到，输入时的光标位置显示是有问题的。因为真正输入的是透明的input，所以说白了光标的位置和看到的密码显示根本就是两回事。比较明显的，比如效果图最后的一位密码，输入的是小写j，然而英文字母的字宽不尽相同，小写j的字宽比较窄。这就造成了刚输入j时，光标的位置是小写j的后面，之后位置又正常了，是因为小写j变成了实心圆，光标的位置便正常了。
另一个不足是，当输入密码长度超过input框本身长度，实心圆会出现半个圆被阶段的情况，这也是因为负责输出的input没有光标位置，所以密码显示不会左移至最末尾，如下图：

屏幕快照_2014-11-10_下午7.23_.08_.png


文章至此，初探了手机端密码输入的显示方式，最后希望今后的CSS属性可以给我们更多的惊喜。
感谢小志的审阅。