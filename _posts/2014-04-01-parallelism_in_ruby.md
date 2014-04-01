---
layout: post
title: ruby的并行
description: "ruby中的并行是个神话"
modified: 2014-04-01
tags: [ruby]
image:
  feature: texture-feature-01.jpg
comments: true
share: true
---

<section id="table-of-contents" class="toc">
  <header>
    <h3>Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->


[Parallelism is a Myth in Ruby](http://www.igvita.com/2008/11/13/concurrency-is-a-myth-in-ruby/)

#### 并行与并发

并发Concurrency：独立的、执行中的事务的组成，在ruby中是可以实现的
并行Parallelism：同时执行多个事务，在ruby中不能实现

#### 全局解释器锁

启动ruby应用后，会启动一个ruby解释器实例来解析代码，创建一个抽象语法树，然后执行应用。同时解释器会实例化一个全局解释器锁（GIL），这个锁导致不能实现并行。

GIL时一个互斥锁，由解释器线程持有，用来避免非线程安全的共享代码。使用GIL限制了多线程的并行能力。

#### 进程并行

在不同的进程间分发工作，这样不仅可以绕过并发引起的一系列问题，并且在应用上使用横向扩展

1. 分解工作或分解应用程序
2. 添加通信/工作队列(Starling, Beanstalkd, RabbitMQ)
3. Fork或运行多个应用实例

很多ruby程序已经采取这种策略

- 一个典型的Rails部署是应用服务器集群(Mongrel, Ebb, Thin),
- 替代方案可使用EventMachine和Revactor，使用简单的办法来实现推迟和并发来处理网络IO，而不是引入线程
