---
layout: post
title: 使用 Laravel 框架开发是什么样的体验
category: PHP
tags: php,Laravel
keywords: php,Laravel
description: 
---

<img src="http://img.gitdc.com/blog/2016/07/laravel-auth-tutorial.png" alt="使用 Laravel 框架是什么样的体验" />

在程序界的远古时期，大神们手持键盘敲着机器语言跟庞大的机器打着交道，那时候机器语言还没有语义和语法，更没有封装的概念。后来进化到汇编语言，C语言时期，基础的功能特性就已经能满足当时的需求了，比如函数封装使其得于复用，但随着计算机的普及，操作系统的到来，面向过程语言已无法继续满足复杂的需求。

历史的变迁促使了高级语言的诞生，到了我们这个阶段，已经是高级语言百花齐放的时代，Java、Python、Ruby、PHP、Javascript、Objective-C、Android 等等都在各自领域中发挥着重要的作用。而各大语言的框架更是集自身语言和其他优秀语言特性之大成者，以PHP来说，Laravel、Symfony、CodeIgniter、ThinkPHP 等都是其优秀的思想结晶之一，其中的 Laravel 就是其创始人 Taylor Otwell 结合了 Ruby on Rails 的思想开发出来的，并且以强大的框架生态和组件化思想成为全世界最热门的PHP框架没有之一。

随着近两年[中文文档][1]的完善，Laravel 渐渐被国内的开发者所使用，去年我们开始采用「组件化」思路来建设整个服务，业务重构选择了 Laravel 来作为后端的业务框架，到现在已经一年时间，我对 Laravel 框架也有了一些了解，从路由到单元测试，Laravel 几乎无所不能，它不局限于“神圣”的MVC模式，让开发者发挥自己的想象力去搭建自己想要的业务架构，不用再想着什么类要放在Model文件夹，而是怎么划分层次，每个层次需要干什么，就像是乐高积木，你可以根据图纸或者自己想象出的变形金刚的样子一层层的搭建出最后的模型。

在 Laravel 中，高级积木（组件）可以用 [Composer][2] 管理工具来引入，Composer 作为PHP中最好用的依赖管理工具之一（或许没有之一）已经被很多框架使用，就不详述它的故事了。绝大多数的 Laravel 组件都能在 [Github][3] 上找到，如果找不到肯定是你的搜索方式有问题，换了搜索方式还找不到，那就自己写一个。Composer 有这么一些常用指令：


* `composer install` - 如有 composer.lock 文件，直接安装，否则从 composer.json 安装最新扩展包和依赖； composer update - 从 composer.json 安装最新扩展包和依赖；
* `composer update vendor/package` - 从 composer.json 或者对应包的配置，并更新到最新；
* `composer require new/package` - 添加安装 new/package, 可以指定版本，如： composer require new/package ~2.5.


我在引入和更新组件时都是使用 `composer require` ，这样子做的原因是因为在生产环境中使用其他两种方法来引入的话，会把其他组件也给更新了，导致不兼容的情况发生，具体看这篇文章：「[正确的 Composer 扩展包安装方法][4]」。

如果是 Composer 的「组件化」让我采用了这个框架来重构，那你就大错特错了，真正让我着迷的是 Laravel 的核心「Ioc容器」，它有效解决了对象依赖的问题，举个栗子：

要实例化一个孩子对象出来，必须要实例化出爸爸对象和妈妈对象，用原本的对象依赖调用方式就是这样的：

        $father = new Father();
        $mother = new Mother();
        $child = new Child($father, $mother);


从编程角度来说，依赖关系越复杂，可变性就会越低，而且这么写等于暴露了三个对象给用户，是不可取的。

从现实角度来看，我只想知道你叫什么名字，你连你爸妈的信息都告诉我了，这......不合适吧。

上面的代码用 Laravel 来写会是这样：

        $child = app()->make(‘Child’);


接着 Ioc 就会帮你调用依赖于 `Child` 的所有类，并且赋予 `Child` 对象，整个过程只需要一句代码，甚至能更简洁：

        $child = app(‘Child’);


Ioc 的好用之处还有很多，基本上 Laravel 的基础服务都是围绕着 Ioc 来搭建的，Router、Middleware、Eloquent ORM等等等等，如果你想感受下 Ioc 的魔力，看[源码][5]吧。

在 Ioc 的基础上，各个子系统也有着自己的闪光点，Router 的 RESTful 定义，Eloquent ORM 的预加载查询优化等等让整个框架生态变得多样化和易用。但也因为应有尽有使得框架的性能比其他高性能框架低了些，Laravel 为此给了几个解决方案：


* 路由缓存；经有关部门研究，路由缓存可有效加快访问速度500ms以上。
* 源码缓存；把源码集合在一个类中，减少资源调用耗费的时间。
* 数据查询优化；就是上面提到的 Eloquent ORM 的预加载查询优化。


这些都做了的话，性能不会差到哪里去，我认为中小体量的网站的瓶颈是在数据IO，PHP性能还不用太纠结，如果真达不到你的性能要求，请转用 [Lumen][6] 或者其他框架。

有朋友说 PHP 框架最重要的东西是路由，我倒认为最重要的是框架中异于其他框架且能解决痛点的东西，如 Laravel 的 Ioc。让我们改变编程思维是很少框架能做到的，Laravel 能做到这点，正所谓框架常有，好框架难求，如果你问我 Laravel 好不好，是不是最好，我只能用邓小平爷爷的名言告诉你：

        不管黑猫白猫，捉到老鼠就是好猫。



[1]: http://www.golaravel.com/
[2]: http://weizhifeng.net/manage-php-dependency-with-composer.html
[3]: https://github.com/
[4]: https://phphub.org/topics/1901
[5]: https://github.com/laravel/laravel
[6]: http://lumen.golaravel.com/
