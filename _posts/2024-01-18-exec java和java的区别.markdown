---
layout: post
title:  "exec java和java的区别"
date:   2024-01-18 18:47:00 +0800
category: linux
---

最近在线上遇到了一个问题：本地写了一个spring-boot服务，想实现一个优雅关闭的逻辑，有两种可能的实现，一种是通过Spring的注解：
```java
@PreDestroy
public void destroy() {
	// some exit logic
}
```

还有一种是注册一个jvm的钩子：

```java
Runtime.getRuntime().addShutdownHook(new Thread(this::destroy));

public void destroy() {
	// some exit logic
}
```

理论上这两种方式都能截获到`SIGTERM`或者`SIGINT`触发优雅关闭。但是我们推荐使用第一种`@PreDestroy`的方式。因为在触发jvm的`shutdownhook`时某些bean已经被spring关闭了，此时代码中有些逻辑可能无法正常执行。

本地IDEA运行时，`@PreDestroy`能够正确生效，但是发布到kubernetes环境中，执行`kill`时发现程序并没有按照预期正常触发`@PreDestroy`，而是在30s后被突然结束进程。

后来发现是image镜像里的命令有问题：
```shell
# start client-service
java -javaagent:xxxxxxx.jar
```

这里的命令是直接执行java，会导致起来的java进程是shell的子进程，当pod被kill尝试发送`SIGTERM`时，shell会收到`SIGTERM`而jvm收不到，从而30秒后达到`terminationGracePeriodSeconds`设定的默认值被强制被`SIGKILL`。

此处应该修改成：
```shell
# start client-service
exec java -javaagent:xxxxxxx.jar
```
这样就可以避免这个问题，因为exec不会起一个子进程，而是在保持PID不变的情况下，让jvm进程替代shell进程，这样收到`SIGTERM`信号的就是jvm本身了，可以正常触发优雅关闭。

我们这里用`echo`命令在本地试验一下。

## 直接运行

先开启一个shell tab，运行`echo $$`看一下这个shell的PID：

![get shell pid](/PNG/get_shell_pid.png)

可以看到PID是63855。

我们`tail -f`进入一个长期运行命令：

![tail -f](/PNG/tail_-f_without_exec.png)

我们另启一个shell，运行`ps -ef | grep tail`看一下有哪些进程：

![ps -ef grep tail](/PNG/ps_-ef__grep_tail.png)

可以看到有两个进程，一个是PID为`63855`的shell进程，另一个是PID为`64663`的`tail`进程，且通过PPID可以看到，`shell`进程是`tail`进程的父进程，这符合预期。

我们尝试在第二个shell中`SIGINT`也就是`kill -2`一下第一个shell进程，看看会发生什么：

![kill -2 63855](/PNG/kill_-2_63855.png)
![tail alive](/PNG/tail_alive.png)

可以看到`tail`进程并未退出，那么如果直接对`tail`进程执行`SIGINT`呢：

![kill -2 64663](/PNG/kill_-2_64663.png)
![tail exit](/PNG/tail_exit.png)

这次`tail`正常结束了，且**回到了父进程shell的交互界面**。可以看到，以这种方式直接启动的进程是收不到来自系统的信号的。

## exec运行

接下来进行第二次试验，回到第一个shell界面，以`exec`前缀的方式启动`tail`命令：

![tail -f with exec](/PNG/tail_-f_with_exec.png)

接下来回到第二个shell并尝试`kill -2 63855`(第一个shell在上一个实验中并没有退出，所以PID还是`63855`没变)：

![kill -2 63855 2nd](/PNG/kill_-2_63855_2.png)
![Process completed](/PNG/Process_completed.png)

可以看到这次`tail`直接结束了，而且与第一次实验不同的是，不仅`tail`结束了，而是直接显示`Process completed`，并没有**返回到shell的交互界面**。这两个实验验证了之前线上的jvm收不到pod被kill的`SIGTERM`信号的case和猜测。