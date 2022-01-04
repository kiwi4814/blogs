+++
title = "Java数组All elements are null异常"
date = "2020-01-03T22:10:15+08:00"
tags = ["Java","错题集"]
slug = "/all-elements-are-null"
draft = false
categories = ["技术"]
+++

使用Mybatis查询数据库时，如果返回值是null并且以List<String>接收时，会出现一条空的记录。

```java
List<String> shldllArrays = ersJqxxExtMapper.selectShldllByJqbh(jqbhs);
```

这时候如果使用`CollectionUtils.isNotEmpty(shldllArrays)`去判断是无法判断这种情况的，从而导致后面可能会报错。在断点中可以发现，这个数组虽然size为1，但是提示：All elements are null。

解决办法：

```java
// 移除所有的null元素，为了防止出现空指针异常【查询返回String会返回一个空null】
shldllArrays.removeAll(Collections.singleton(null));
```

