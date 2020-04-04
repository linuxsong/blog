---
title: Shell中按指定分隔符读取文件字段
key: Shell中按指定分隔符读取文件字段
tags: shell
author: LinuxSong
---


在很多情况下我们要对文本文件进行读取操作，Shell中读取文件的方法有很多种。比如简单的按行读取：
{% highlight bash %}
#!/bin/bash

for line in `cat file.txt`
do
    echo $line
done
{% endhighlight %}

每一次循环会读入一行文件内容放入$line变量中。

另外一种情况下，我们要读取像csv类似的文件，每行的字段按指定的分隔符分隔，该如何处理呢？
可以利用IFS环境变量来指定分隔符并用read命令来读取各个字段。

<!--more-->

IFS环境变量是shell中的字段分隔符，默认以空格、tab、换行符为分隔符。修改该环境变量可以指定字段分隔符。

比如有一个以逗号分隔的file.csv文件，内容如下：

```
111,222,333
444,555,666
```
读取方法：

{% highlight bash %}
#!/bin/bash

IFS=,
cat file.csv | while read a b c
do
    echo $a $b $c
done
{% endhighlight %}

输出：

```
111 222 333
444 555 666
```

若行内字段的数量超过了read命令指定的变量数量，那么多余的字段会被分配到最后的一个变量中。

当每行字段数量比较多的时候，分配多个变量并不是很方便，read命令支持将内容分配到数组中。比如：

{% highlight bash %}
#!/bin/bash

IFS=,
cat file.csv | while read -a line
do
    echo ${line[0]} ${line[1]} ${line[2]}
done
{% endhighlight %}

每行的各个字段被存储到$line数组中（下标以0开始）
输出：

```
111 222 333
444 555 666
```

若要访问数组中的所有内容可以用 ${line[*]}

传递文件内容还有另外一种写法：

{% highlight bash %}
#!/bin/bash

IFS=,
while read a b c
do
    echo $a $b $c
done < file.csv
{% endhighlight %}

使用文件重定向的方式，输出结果和前面的写法一样, 但是更简洁。

另外如果想让分隔符的修改不影响当前Shell中的后续操作，可以把IFS的环境变量放在命令前，例如：
{% highlight bash %}
#!/bin/bash

while IFS=, read a b c
do
    echo $a $b $c
done < file.csv
{% endhighlight %}

这样修改的分隔符只对上面while中的read命令生效，若Shell中还有其它需要用到分隔符的命令，不会产生影响，仍然会使用Shell默认的分隔符（空格、tab、换行符）。





