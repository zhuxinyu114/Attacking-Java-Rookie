# 线程




#### 一个异常 Thread starvation or clock leap detected
```text
2018-05-27 13:56:50.820  WARN 111644 --- [      Thread-49] c.g.htmlunit.IncorrectnessListenerImpl   : Obsolete content type encountered: 'text/javascript'.
2018-05-27 13:58:26.957  WARN 111644 --- [l-1 housekeeper] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Thread starvation or clock leap detected (housekeeper delta=51s792ms365µs576ns).
2018-05-27 14:02:55.861  WARN 111644 --- [l-1 housekeeper] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Thread starvation or clock leap detected (housekeeper delta=1m33s765ms258µs244ns).
2018-05-27 14:08:47.880  WARN 111644 --- [l-1 housekeeper] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Thread starvation or clock leap detected (housekeeper delta=6m53s824ms351µs439ns).
2018-05-27 14:17:02.136  WARN 111644 --- [l-1 housekeeper] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Thread starvation or clock leap detected (housekeeper delta=8m17s388ms195µs302ns).
```


This runs on the housekeeper thread, which executes every 30 seconds. If you are on Mac OS X, the clockSource is System.currentTimeMillis(), any other platform the clockSource is System.nanoTime(). Both in theory are monotonically increasing, but various things can affect that such as NTP servers. Most OSes are designed to handle backward NTP time adjustments to preserve the illusion of the forward flow of time.

This code is saying, if time moves backwards (now < previous), or if time has "jumped forward" more than two housekeeping periods (more than 60 seconds), then something strange is likely going on.

A couple of things might be going on:

You could be running in a virtual container (VMWare, AWS, etc.) that for some reason is doing a particularly poor job of maintaining the illusion of the forward flow of time.

Because other things occur in the housekeeper thread -- specifically, closing idle connections -- it is possible that for some reason closing connections is blocking the housekeeper thread for more than two housekeeping periods (60 seconds).

The server is so busy, with all CPUs pegged, that thread starvation is occurring, which is preventing the housekeeper thread from running for more than two housekeeping periods.

Considering these, maybe you can provide additional context.

EDIT: Note that this is based on HikariCP 2.4.1 code. Make sure you are running the most up-to-date version available.


###### 翻译如下：

这在管家线程上运行，该线程每30秒执行一次。如果在Mac OS X上，clockSource是System.currentTimeMillis（），则任何其他平台上的clockSource是System.nanoTime（）。从理论上讲，两者都在单调增加，但是诸如NTP服务器之类的各种因素都可能影响到这一点。大多数操作系统旨在处理向后NTP时间调整，以保留对时间的前向错觉的幻想。

这段代码说的是，如果时间倒退（现在<以前），或者如果时间“跳跃”了两个以上的内务处理周期（超过60秒），那么可能会发生一些奇怪的事情。

可能发生了几件事情：

您可能正在某个虚拟容器（VMWare，AWS等）中运行，由于某种原因，该容器在维持时间上的错觉方面做得特别差。

由于管家线程中发生了其他事情-特别是关闭空闲连接-出于某种原因，关闭连接可能会阻塞管家线程两个以上的维护周期（60秒）。

服务器太忙了，所有CPU都挂了，导致线程出现饥饿，这导致管家线程无法运行两个以上的管家周期。

考虑到这些，也许您可​​以提供其他上下文。

编辑：请注意，这是基于HikariCP 2.4.1代码的。确保您正在运行最新的可用版本。
