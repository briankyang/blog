---
title: thinkphp 3.2.0 field bug
date: 2017-05-31 02:16:45
tags: php
---

## bug source
最近在用thinkphp框架写东西的时候，遇到了需要连接查询，先看两张表结构

<!-- more -->
![1](/img/1.png)
![2](/img/2.png)

关联字段为 *表一.id* 和 *表二.cid*

thinkphp指定查询字段的方法为

`public function Model::field($field,$except=false) Model`

当 *field* 字段为 *true* 时，获取当前表的所有字段，注意是 **名称**，比如 `id` 而非 `xxx.id`,如果第二张表也有id,数据库在查询的时候会报错 `#1052 - Column 'id' in field list is ambiguous`

解决办法： 暂时性在field中指定带有前缀的字段名
