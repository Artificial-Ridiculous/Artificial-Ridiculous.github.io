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
@PreDestroy
Runtime.getRuntime().addShutdownHook(new Thread(this::destroy));

public void destroy() {
	// some exit logic
}
```

理论上这两种方式都能截获到`SIGTERM`或者`SIGINT`触发优雅关闭。但是我们推荐使用第一种`@PreDestroy`的方式。因为在触发jvm的`shutdownhook`时某些bean已经被spring关闭了，此时代码中有些逻辑可能无法正常执行。

本地IDEA运行时，`@PreDestroy`能够正确生效，但是发布到kubernetes环境中，执行`kill`时发现程序并没有按照预期正常触发`@PreDestroy`而是在30s后被突然结束进程。

后来发现是image镜像里的命令有问题：
```shell
# start client-service
java -javaagent:xxxxxxx.jar
```

这里的命令是直接执行java，这样会导致起来的java进程是shell的子进程，当pod被kill尝试发送`SIGTERM`时，shell会收到`SIGTERM`而jvm收不到，从而30秒后达到`terminationGracePeriodSeconds`设定的默认之后，强制被`SIGKILL`。

应该修改成：
```shell
# start client-service
exec java -javaagent:xxxxxxx.jar
```
这样就可以避免这个问题，因为exec不会起一个子进程，而是在保持PID不变的情况下，让jvm进程替代shell进程，这样收到`SIGTERM`信号的就是jvm本身了，可以正常触发优雅关闭。


