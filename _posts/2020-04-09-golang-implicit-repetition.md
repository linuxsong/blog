---
title: Golang常量声明中的隐式重复与iota
key: Golang常量声明中的隐式重复与iota
tags: go
author: LinuxSong
---


Go语言中批量声明常量时，除了第一个常量外其它的常量右边的初始化表达式都可以省略，如果省略初始化表达式则表示使用前面第一个非空表达式常量的初始化表达式及其类型（如果有的话），标识符的数量必须等于上一个列表中的表达式的数量，这个特性被称作“隐式重复(implicit repetition)”。

如果只是简单地复制右边的常量表达式，其实并没有太实用的价值。但是它可以带来其它的特性，那就是iota常量生成器语法。

看下面的例子：

{% highlight Go linenos %}
const (
    a = iota
    b = iota
    c = iota
)
{% endhighlight %}

<!--more-->
在其它语言中，这种形式通常被称作“枚举”。利用隐式重复的特性，我们可以这样写：

{% highlight Go linenos %}
const (
    a = iota
    b // 隐式重复b = itoa
    c // 隐式重复c = itoa
)
{% endhighlight %}

需要注意的一点就是,隐式重复不仅仅是重复的表达式，而且类型也会同样重复。

看下面的例子：

{% highlight Go linenos %}
type ByteSize float64
const (
    _ = iota // 通过赋值给空白标识符来忽略值
    KB ByteSize = 1<<(10*iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)
fmt.Println(reflect.TypeOf(MB)) 
{% endhighlight %}

iota 表示的是没有类型化的整数常量。上面的例子中，通过reflect我们可以看到，MB常量的类型是重复了KB的ByteSize类型，而不是int类型。

在同一个常量表达式列表定义中多次使用iota都具有相同的值。

如下例：

{% highlight Go linenos %}
const (
    bit0, mask0 = 1 << iota, 1<<iota - 1  // bit0 == 1, mask0 == 0  (iota == 0)
    bit1, mask1                           // bit1 == 2, mask1 == 1  (iota == 1)
    _, _                                  //                        (iota == 2, unused)
    bit3, mask3                           // bit3 == 8, mask3 == 7  (iota == 3)
)
{% endhighlight %}

更多细节可以查看[Go语言规范](https://golang.org/ref/spec#Constant_declarations){:target="_blank"}