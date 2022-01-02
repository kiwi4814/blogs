+++
title = "如何在MySQL中设计一个树形存储结构？"
date = 2020-09-01T20:35:47+08:00
slug = "/mysql-tree"
tags = ["MySQL"]
categories = ["技术"]
+++


我们经常需要在数据库中去维护一个树形结构，通常普遍的做法有以下几种：

- ***Adjacency List***
  
    每一条记录存在一个`parent_id`
    
- ***Path Enumerations***
  
    每一条记录存整个`tree path`经过的`node`枚举
    
- ***Nested Sets***
  
    每一条记录存 `nleft` 和 `nright`
    
- ***Closure Table***
  
    额外维护一个表，所有的t`ree path`作为记录进行保存。
    

[各类方法的操作代价](https://www.notion.so/322ad02521884b51bade610e1b42ee98)