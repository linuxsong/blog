---
status: publish
published: true
title: html5+javascript画谢尔宾斯基三角形
author:
  display_name: LinuxSong
  login: xks
  email: linuxsong@gmail.com
  url: http://www.linuxsong.org
author_login: xks
author_email: linuxsong@gmail.com
author_url: http://www.linuxsong.org
wordpress_id: 582
wordpress_url: http://www.linuxsong.org/?p=582
date: '2015-04-07 11:18:13 +0800'
date_gmt: '2015-04-07 03:18:13 +0800'
comments: []
---
<p>谢尔宾斯基三角形（Sierpinski triangle）是一种分形，由波兰数学家谢尔宾斯基在1915年提出。</p>
<p><a href="http://www.linuxsong.org/wp-content/uploads/2015/04/下载-1-e1428373711612.png"><img class="alignnone wp-image-583 size-full" src="http://www.linuxsong.org/wp-content/uploads/2015/04/下载-1-e1428373711612.png" alt="" /></a></p>
<p>我们来看一下这个谢尔宾斯基三角形的实现步骤：</p>
<p>1. 画一个三角形（通常为等边三角形）</p>
<p><a href="http://www.linuxsong.org/wp-content/uploads/2015/04/5CC053DC-AA5D-4EDD-9941-97958E6EC7AB.jpg"><img class="alignnone wp-image-584 size-medium" src="http://www.linuxsong.org/wp-content/uploads/2015/04/5CC053DC-AA5D-4EDD-9941-97958E6EC7AB-300x251.jpg" alt="" width="300" height="251" /></a></p>
<p>2. 假设上面的顶点为a,左边的点为b,右边的为c</p>
<p>选出三条边的中间点，a-b,a-c,b-c,画三个三角形</p>
<p>a a-b a-c</p>
<p>a-b b b-c</p>
<p>a-c b-c c</p>
<p><img class="alignnone size-medium wp-image-589" src="http://www.linuxsong.org/wp-content/uploads/2015/04/222-300x264.jpg" alt="222" width="300" height="264" /></p>
<p>3.对新画出的三个三角形按步骤二继续绘制（递归的过程），直到你认为绘制的三角形足够小。</p>
<p><a href="http://www.linuxsong.org/wp-content/uploads/2015/04/下载-1-e1428373711612.png"><img class="alignnone wp-image-583 size-full" src="http://www.linuxsong.org/wp-content/uploads/2015/04/下载-1-e1428373711612.png" alt="" width="405" height="332" /></a></p>
<p>把以上的步骤理解清楚以后，就很容易实现，递归的思想用递归来实现是最合适不过了。</p>
<!--more-->

{% highlight js linenos %}

//谢尔宾斯基三角形
function sierpinski(ctx, a, b, c)
{
    if (isFinished(a,b,c)) {
        return;
    }
    drawTriangle(ctx, a,b,c)
    sierpinski(ctx, a, middle(a,b), middle(a, c));
    sierpinski(ctx, middle(a,b), b, middle(b, c));
    sierpinski(ctx, middle(a,c), middle(b, c), c);
}
//定义一个点对象
function Point(x,y)
{
    this.x = x;
    this.y = y;
}
//画一个三角形
function drawTriangle(ctx,a,b,c)
{
    drawLine(ctx, a, b);
    drawLine(ctx, a, c);
    drawLine(ctx, b, c);
}
//取中间点
function middle(a,b)
{
    return new Point(Math.abs(a.x+b.x)/2, Math.abs(a.y+b.y)/2);
}
//画线
function drawLine(ctx,a,b)
{
    ctx.beginPath();
    ctx.moveTo(a.x, a.y);
    ctx.lineTo(b.x, b.y);
    ctx.stroke();
}
//判断是否停止绘制
function isFinished(a,b,c)
{
    return Math.abs(b.x-c.x) < 5;
}

(function() {
    var canvas = document.getElementById('canvas');
    var ctx = canvas.getContext('2d');
    ctx.strokeStyle = "blue";
    var a = new Point(200, 0);
    var b = new Point(27, 300);
    var c = new Point(373, 300);
    sierpinski(ctx, a, b,c, 900);
})();
{% endhighlight %}

<p>下面是用html5+javascript来实现的例子，需要Chrome 、 Firefox 、Safari,或ie9+浏览器查看,为查看中间步骤，加了一个时间延迟：</p>
<p><center><br />
<canvas id="canvas" width="500" height="500"></canvas></center><br />
<script type="text/javascript">// <![CDATA[<br />
function Point(x,y)
{
    this.x = x;
    this.y = y;
}
function sierpinski(ctx, a, b, c, delay)
{
    if (isFinished(a,b,c)) {
        return;
    }
    drawTriangle(ctx, a,b,c)
    setTimeout(sierpinski,delay, ctx, a, middle(a,b), middle(a, c), delay);
    setTimeout(sierpinski,delay, ctx, middle(a,b), b, middle(b, c), delay);
    setTimeout(sierpinski,delay, ctx, middle(a,c), middle(b, c), c, delay);
}
function drawTriangle(ctx,a,b,c)
{
    drawLine(ctx, a, b);
    drawLine(ctx, a, c);
    drawLine(ctx, b, c);
}
function middle(a,b)
{
    return new Point(Math.abs(a.x+b.x)/2, Math.abs(a.y+b.y)/2);
}
function drawLine(ctx,a,b)
{
    ctx.beginPath();
    ctx.moveTo(a.x, a.y);
    ctx.lineTo(b.x, b.y);
    ctx.stroke();
}
function isFinished(a,b,c)
{
    return Math.abs(b.x-c.x) < 5;
}
(function() {
    var canvas = document.getElementById('canvas');
    var ctx = canvas.getContext('2d');
    ctx.strokeStyle = "blue";
    var a = new Point(200, 0);
    var b = new Point(27, 300);
    var c = new Point(373, 300);
    sierpinski(ctx, a, b,c, 900);
})();
// ]]></script>
