---
layout: post
title: 一行代码引发的主从数据库300秒延迟
category: 服务器
tags: php,Mysql,主从延迟
keywords: php,Mysql,主从延迟
description: 
---

一个悠闲的上午，DBA 发群上的一条消息把我从代码的深渊拉了出来：“从机延迟了300s”。

我心中简单思考了一番确认了可能造成的原因，更重要的一点，确认这不是我干的好事后，继续淡定地敲着我的代码。But，这件事情要是没我什么事，我也不会写这篇文章，所以我们继续看。

这个延迟只有10点、11点、12点的整点前5分钟到整点后15分钟出现，也就是10点到12点每个整点左右的20分钟，DBA 折腾了一个上午没见什么效果（DBA 刚接手我们部门的 DB，不熟悉业务，后面 DBA 帮了我不少忙），这个「任务」就“幸运的”到我手上了。

<img src="http://img.gitdc.com/blog/2016/08/tmp6a014e7c-1024x234.png" alt="db" />

先来简单谈谈延迟300s的严重性，部门主要业务是汽车交易平台，从机延迟300s相当于商家刊登了一笔物件，5分钟后才能看到，我们平台上的活跃商家已经有上千位，延迟会让商家质疑平台的功能可用性，对他们和客服的影响可能不亚于down机的情况。

从机延迟主要的原因有几种：


1.  数据表无索引或SQL慢；
2.  主库上有大事务，导致从库延时；
3.  主库写入频繁，从库压力跟不上导致延时；


## 数据表无索引或SQL慢

监控系统每天都会传送 MySQL 的慢速日志到我们的邮箱以检查业务中比较慢的 sql 语句，出事的那段时间也没有修改过数据表结构删除数据表索引，所以这点可以排除。

## 主库上有大事务，导致从库延时

那段时间并没有特别大的功能上线，经 DBA 查证，也排除这个情况。

## 主库写入频繁，从库压力跟不上导致延时

这是从库延迟最经常出现的情况，主从同步的原理就是拉条线程去主库拿到 binlog 回来执行，由于是单线程，主库并发写入数据表很频繁时，就会造成线程堵塞。

## 从「主库写入频繁」切入

MySQL 的语句是记录在 binlog下面的，想查出问题就必须从 binlog 入手，从 binlog 中查到时间`09:50`到`10:03`，物件搜寻表`t_search***_item**`的`delete`和`update`语句数量分别有3W+和1W＋，排数量列表首位。

<img src="http://img.gitdc.com/blog/2016/08/sql.png" alt="sql" />

继续查到语句所在的类，是个一分钟跑一次的自动脚本，这个脚本因为历史遗留问题，逻辑已经复杂到无法细看的程度，找到相关的同事了解后，代码逻辑里就算是为了方便直接`delete`后再`insert`数据，也不可能会短时间产生这么多`delete`操作。已经嗅到真相味道的我欢快的回到自己座位上翻着 Git 时间线，试图找出导致这次事件的真凶，为我逝去的一天光阴画上圆满的句号......

但是，把 Git 前面2个月的时间线翻遍后，居然没有这个脚本相关的 Commit 记录，万只可爱的羊驼在心里飘过后，不由得开始怀疑自己的推理是不是错的，一天的光阴啊......

这时候部门技术 Leader 路过看了下，把问题简述给他听之后，他微微一笑，深藏功与名的用手指指了指我的 Git 分支，“原来我没切换到 Master 分支！”，我恍然大悟。

低级错误被纠正后，顺利找到了相关的 Commit 记录，也找到了导致从库延迟的原因，同事在为这个脚本加功能的时候，没看上下文，在没刷新字段前return了出去。导致`delete`，`insert`之后没刷新字段，下次运行脚本又被取了出来重复`delete`，`insert`了。

那为什么会在整点的时候才有这种情况呢？这是因为商家物件有定时更新的功能，通常这三个整点的物件定时更新比较多，而定时更新会`update`物件搜索表，也就造成了整点会有从库延迟得这么厉害的情况发生，而这一切，仅仅是一个`return`造成的。

