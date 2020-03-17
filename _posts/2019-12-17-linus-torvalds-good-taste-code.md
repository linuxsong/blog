---
title: Linus Torvalds 提及的关于代码的"good taste"
key: Linus Torvalds 提及的关于代码的"good taste"
tags: linux c
author: LinuxSong
---

在一个关于 [Linus Torvalds 的一个访谈](https://www.ted.com/talks/linus_torvalds_the_mind_behind_linux){:target="_blank"}中, Linus Torvalds 举了一个关于什么是代码好的品味的例子（视频14:20左右)。

他先是展示了一个C代码片断。

![poor taste code example](/assets/images/posts/poor-taste-code-example.png)

这段代码的作用是删除链表中的一个节点, Linus 认为这不是一段具有好品味的代码。原因是循环后面的 if 判断，删除链表节点时需要考虑两种不同的情况，一种是删除的是链表的head节点，另外一种是删除的中间节点或者尾部节点。

接下来, Linus 又展示了另外一段具有好品味的代码,和上面的一段代码实现的是同样的功能。

<!--more-->

![good taste code example](/assets/images/posts/good-taste-code-example.png)

Linus 解释了这段代码为什么更好, 它消除了边缘情况。

我花了一点时间来理解这段代码，代码使用了指针的指针来直接修改节点的指针地址。对于head节点就是修改&head,下一节点就是&(*indirect->next),消除了特例。

类似的一个例子是交换链表中的两个节点（不是交换节点的值），也可以使用上面的方法来实现,比如[leetcode上有一个题目](https://leetcode-cn.com/problems/swap-nodes-in-pairs/){:target="_blank"}两两交换其中相邻的节点,按上面的方式来实现，代码会简洁许多：

{% highlight c++ linenos %}
    ListNode* swapPairs(ListNode* head) {
        ListNode **node = &head;
        while (*node != NULL && (*node)->next != NULL) {
            ListNode *tmp = *node;
            *node = (*node)->next;
            tmp->next = (*node)->next;
            (*node)->next = tmp;
            node = &((*node)->next->next);
       }
       return head;
    }
{% endhighlight %}

我们在写代码时，很重要的一点就是要消除代码中的特例，让代码变得更简洁，比如面向对象中类的抽象，从某种意义上说就是为了消除特例。

从可读性上来讲，Linus举例的第一段代码可能更容易理解，第二段代码不能一眼看出意图。在我们平常处理链表的时候，通常为了处理方便会添加一个空头指针，来实现简化处理逻辑，消除边缘情况。

但是这个例子给我们提供了一种思路，有些看上去很简单或者看上去理所当然的代码，也会有改进的方法，以精益求精的方式面对你的代码，会让你写出的代码越来越好。就像Linus在访谈中所说的,有的时候你可以换个角度看问题，重写代码排除特例,完美覆盖所有情况。


