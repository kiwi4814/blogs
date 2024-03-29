+++
title = "Redis常用数据结构小结"
date = 2022-09-11 22:50:47
slug = "/redis-data-structures"
draft = false
tags = ["技术","Redis"]
series = ["Redis实战"]
toc = false

+++
>  本文的内容为阅读笔记，主要内容源自[图解 Redis 数据结构](https://xiaolincoding.com/Redis/data_struct/data_struct.html)以及[Redis核心技术与实战 | 数据结构](https://time.geekbang.org/column/article/268253)，仅为了总结和方便理解。



如果被问到Redis支持哪些数据结构，我们一般都会脱口而出这五种：String、List、Hash、Set、Sorted Set，但其实严格来说这五种只是Redis键值对种值得数据类型，也就是数据得保存形式。而我们所说的数据结构，其实是Redis底层的实现，如下图所示，Redis底层的实现有很多种，在不同的版本也在不断的迭代，而这些数据结构正是【Redis为什么这么快】这个问题的主要原因之一。



> 【题外话】
>
> 我们经常能看到有序列表（Sorted Set）也被称作为Zset，这种叫法的直接来源是因为有序列表的命令就是zset，而设计者为什么这么命名的原因其实并不复杂，只是一种文化而已，在redis的issue中有人提到了[这个问题](https://github.com/redis/redis/issues/4024)，感兴趣的可以看看原答案。





<img src="https://kiwi4814-1256211473.cos.ap-nanjing.myqcloud.com//img202208161405062.png" alt="img" style="zoom: 50%;" />





接下来，让我们具体分析下Redis底层的数据结构，当然，在分析键值对种“值”的底层结构之前，我们需要先了解下键和值是如何映射的。

## 键和值的结构组织

为了实现从键到值得快速访问，Redis使用了一个hash表来保存所有键值对。hash表中的每一个元素叫hash桶，hash桶中包含两个指针，分别指向实际的键和值，这样即使值是一个集合，也可以被查找到。



<img src="https://kiwi4814-1256211473.cos.ap-nanjing.myqcloud.com//img1cc8eaed5d1ca4e3cdbaa5a3d48dfb5f.jpg" alt="img" style="zoom: 50%;" />



**第一个问题：hash冲突**

我们知道Hash表查找的时间复杂度是O(1)，其查找过程主要依赖哈希计算，但是我们知道hash计算是会有冲突的，所以当Redis中被写入越来越多的数据时，hash冲突就在所难免。

解决的方法很简单，Redis采用了链式哈希，类似Java 8之前的HashMap的数据结构，当哈希冲突时，落到同一个Hash桶的数据用一个链表保存。但是，这样仍然有问题，随着数据越来越多，链表上的数据也会越来越多，会导致某些元素的查找时间会变长，这对于追求快的Redis来说是不可接受的。



**第二个问题：链表过长导致查询效率降低**

首先说解决方案：Redis会对哈希表做rehash操作，也就是增加现有的哈希桶数量，让元素分散保存。为了让rehash操作更高效，Redis默认使用两个全局哈希表，一开始插入数据时只会使用哈希表1，随着数据逐渐增多，Redis开始进行rehash操作：

- 给哈希表2分配更大的空间
- 将哈希表1的数据拷贝到哈希表2
- 释放哈希表1的空间

**第三个问题：rehash导致的线程阻塞**

rehash的过程中涉及到大量的数据拷贝，这时候是会导致线程阻塞，为了解决这个问题，redis采用了**渐进式 rehash**。

简单来说，拷贝数据时，redis仍然正常处理客户端请求，但是每处理一个请求时，都会将哈希表1中的对应索引位置的数据拷贝到哈希表2，这样就巧妙的将一次拷贝的开销，分摊到了多次处理请求的过程中，避免了耗时操作。

## 底层数据结构

对于String来说，找到哈希桶就可以直接进行增删改查操作了，所以复杂度仍然是O(1)。但是对于集合类型来说，其操作效率取决于集合底层的数据结构。



对于集合类型而言，底层的数据结构主要包括：**整数数组、双向链表、哈希表、压缩列表和跳表**，redis新版本中还引入了quicklist和listpack，用来替换双向链表和压缩链表。接下来，让我们逐一分析下这几种数据结构的特点。

### 哈希表

哈希表的特质在键值对的结构组织中基本已经说过了，这里有几点需要注意下。

1. 当我们存入的数据类型为hash对象时，底层编码不一定是hash。当哈希中的元素个数比较少并且每个元素的值占用空间比较小的时候，Redis就会使用压缩列表做为哈希的内部编码。
2. rehash是一个过程，在这个过程中，查找值的时候会先去哈希表1查再去哈希表2查，因为新增的数据会增加到哈希表2，哈希表1的数据会越来越少，直到完成rehash操作。

### 整数数组

整数集合是 Set 对象的底层实现之一。当一个 Set 对象只包含整数值元素，并且元素数量不大时，就会使用整数集这个数据结构作为底层实现。

整数集合本质上是一块连续的内存空间，整数数组有个扩容升级的过程，当你存入超过阈值大小的数据时触发且不会降级。

### 双向链表

在Redis 3.0 的版本中，Redis 的 List 对象的底层实现之一就是双向链表。



双向链表的优点在于获取表头节点、表尾节点，以及任意节点的前置节点和后置节点的时候时间复杂度只需要O(1)，并且由于redis的链表结构存储了节点数量，所以获取链表的节点数量也仅需要O(1)的时间复杂度。



但是链表的缺点也很明显，链表每个节点之间的内存是不连续的，意味着无法很好地利用CPU缓存。因此，Redis 3.0 的 List 对象在数据量比较少的情况下，会采用「压缩列表」作为底层数据结构的实现，它的优势是节省内存空间，并且是内存紧凑型的数据结构。而在Redis3.2之后，List 对象的底层改由 「**quicklist**」数据结构实现。



### 压缩列表

压缩列表是由一个**连续内存组成的顺序型数据结构**。一个压缩列表可以包含任意多个节点，每个节点上可以保存一个字节数组或整数值。它是Redis为了节省内存空间而开发的。与数组不同的是，压缩列表在表头有三个字段 zlbytes、zltail 和 zllen，分别表示列表长度、列表尾的偏移量和列表中的 entry 个数；压缩列表在表尾还有一个 zlend，表示列表结束。

<img src="https://kiwi4814-1256211473.cos.ap-nanjing.myqcloud.com//img9587e483f6ea82f560ff10484aaca4a0.jpg" alt="img" style="zoom:50%;" />

在压缩列表中，如果我们要查找定位第一个元素和最后一个元素，可以通过表头三个字段的长度直接定位，复杂度是 O(1)。而查找其他元素时，就没有这么高效了，只能逐个查找，此时的复杂度就是 O(N) 了。



在redis中，下面的这些情况都会使用到压缩列表：

- 当哈希（hash）中的元素个数比较少并且每个元素的值占用空间比较小的时候，Redis就会使用压缩列表做为哈希的内部编码。
- 当有序集合（zset）中的元素个数比较少并且每个元素的值占用空间比较小的时候，Redis也会使用压缩列表做为有序集合的内部编码。



### quicklist

在 Redis 3.2 的时候，List 对象的底层改由 quicklist 数据结构实现。



其实 quicklist 就是「双向链表 + 压缩列表」组合，因为一个 quicklist 就是一个链表，而链表中的每个元素又是一个压缩列表。结构如下所示：



<img src="https://kiwi4814-1256211473.cos.ap-nanjing.myqcloud.com//imgf46cbe347f65ded522f1cc3fd8dba549.png" alt="img" style="zoom: 88%;" />

在向 quicklist 添加一个元素的时候，不会像普通的链表那样，直接新建一个链表节点。而是会检查插入位置的压缩列表是否能容纳该元素，如果能容纳就直接保存到 quicklistNode 结构里的压缩列表，如果不能容纳，才会新建一个新的 quicklistNode 结构。

### 跳表

跳表是有序列表（zset）的内部编码之一，也是一种重要的数据结构。



我们知道zset可以将元素及其score值加到有序集中，那么作为底层数据结构的跳表，具体是怎么实现的呢？



首先，我们知道对于有序数组，我们可以使用二分查找的方式快速定位一个数字，是因为数组具有随机查找的特性，每次找中位数比较即可。但如果对于一个有序的链表，二分查找就不适用了。所以对于链表，出现了一种跳表的数据结构，其原理就是在有序链表上面增加了多级索引，下图演示了在跳表的结构下查找数据的过程：

<img src="https://kiwi4814-1256211473.cos.ap-nanjing.myqcloud.com//img1eca7135d38de2yy16681c2bbc4f3fb4.jpg" alt="img" style="zoom:50%;" />



Redis对于增加到zset中的每一个元素，其层数都是随机分配的。跳表采取了一种空间换时间的方法，将链表的时间复杂度降到了O(logN)。至于为什么不采用平衡树？主要是基于以下几点：

- **从内存占用上来比较，跳表比平衡树更灵活一些**。
- **在做范围查找的时候，跳表比平衡树操作要简单**。
- **从算法实现难度上来比较，跳表比平衡树要简单得多**。



## 总结

Redis 之所以能快速操作键值对，一方面是因为 O(1) 复杂度的哈希表被广泛使用，包括 String、Hash 和 Set，它们的操作复杂度基本由哈希表决定，另一方面，Sorted Set 也采用了 O(logN) 复杂度的跳表。不过，集合类型的范围操作，因为要遍历底层数据结构，复杂度通常是 O(N)。这里，我的建议是：用其他命令来替代，例如可以用 SCAN 来代替，避免在 Redis 内部产生费时的全集合遍历操作。

<img src="https://kiwi4814-1256211473.cos.ap-nanjing.myqcloud.com//imgfb7e3612ddee8a0ea49b7c40673a0cf0.jpg" alt="img" style="zoom:50%;" />



当然，我们不能忘了复杂度较高的 List 类型，它的两种底层实现结构：双向链表和压缩列表的操作复杂度都是 O(N)。因此，我的建议是：因地制宜地使用 List 类型。例如，既然它的 POP/PUSH 效率很高，那么就将它主要用于 FIFO 队列场景，而不是作为一个可以随机读写的集合。



