---
layout: post
title: EventMachine
description: "ruby中的eventmachine学习"
modified: 2014-04-01
tags: [ruby]
image:
  feature: texture-feature-02.jpg
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


#### Reactor 模式
    一种并发编程模式：为了处理被一个或多个输入同 时发送到服务处理器的服务请求，服务处理器会分离传入的请求并且同 步地派发它们到相关的请求处理器。

    一个最基本原则：Never block the Reactor!

#### EM#run
    启动Reactor，运行下去直到调用EM#stop或EM#stop_event_loop后，EM#run后的代码才被执行
    通常在EM.run前调用Em.poll（EM默认使用select）

#### EM定时器
    add_timer 添加一次性定时器
    add_periodic_timer  添加周期性定时器

    EM::Timer.new(n) 和 EM::PeriodicTimer.new(n)效果相同
    如果要取消循环定时器，需要使用类方法EM::PeriodicTimer。这个给我们提供了 EM::PeriodicTimer#cancel 方法

#### 推迟和并发处理

    EM#defer和EM#next_tick：1.长任务应该放到后台执行，2.一旦这些任务被转移到后台，Reactor能够立即回来工作

##### EM#defer
    负责把一个代码块（block）调度到EM的线程池中执行（固定提供了20个线程），而defer的Callback参数指定的方法将会在主线程（即Reactor线程）中执行，并接收后台线程的返回值作为Callback块的参数。

##### EM#next_tick
    负责将一个代码块调度到Reactor的下一次迭代中执行，执行任务的是Reactor主线程。所以，next_tick部分的代码不会立刻执行到，具体的调度是由EM完成的
    一个很常见的用法是递归的调用方式，将一个长的任务分配到Reactor的不同迭代周期去执行。next_tick使单进程的Reactor给其他任务运行的机会——我们不想阻塞住Reactor，但我们也不愿引入Ruby线程，所以才有了这种方法。
    同样作用的EM.Schedule：不停判断当前线程是否Reactor  
    通过defer方法把一个代码端调度到线程池中执行，然后又需要在主线程中使用EM::HttpClient来发一个出站连接，这时你就可以在前面的代码段里使用next_tick创建这个连接

#### 轻量级并发

    Deferrables和SpawnedProcesses

##### EM::Deferrable

    如果在一个类中include了EM::Deferrable，就可以把Callback和Errback关联到这个类的实例。一旦执行条件被触发，Callback和Errback会按照与实例关联的顺序执行起来。

    对应实例的#set_deferred_status方法就用来负责触发机制：当该方法的参数是:succeeded，则触发callbacks；而如果参数是:failed，则触发errbacks。触发之后，这些回调将会在主线程立即得到执行。当然你还可以在回调中(callbacks和errbacks)再次调用#set_deferred_status，改变状态。

##### EM::SpawnedProcess

    这个方法的设计思想是：允许我们创建一个进程，把一个代码段绑定到这个进程上。然后我们就可以在某个时刻，让spawned实例被#notify方法触发，从而执行关联好的代码段

#### EM::Connection

    post_init：当实例创建好，连接还没有完全建立的时候调用。一般用来做初始化

    connection_completed：连接完全建立好的时候调用

    receive_data(data)：当收到另一端的数据时调用。数据是成块接收的

    unbind：当客户端断开连接的时候调用

    close_connection close_connection_after_writing：供用户断开连接。