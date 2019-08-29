---
layout: post
title:  "Hive:在insert语句中使用自动分区报错的解决方案"
date:   2019-08-29 14:04:00 +0800
category: hive
---

Hive支持基于文件夹的分区功能,默认的`hive.exec.dynamic.partition`设置为`false`(静态分区),如果需要开启动态分区,则需要修改该值:

```sql
hive> hhive.exec.dynamic.partition=true;
```

---

在执行HQL语句时,如果使用了自动分区(`partition(country,state)`),可能会不成功并有以下提示:

```
Error: Error while compiling statement: FAILED: SemanticException [Error 10096]: Dynamic partition strict mode requires at least one  column. To turn this off set hive.exec.dynamic.partition.mode=nonstrict (state=42000,code=10096)
```
这是因为 `hive.exec.dynamic.partition.mode` 的默认值是 `strict`,意为在设定分区时,至少要指定一个静态分区.如果需要纯动态分区,则需要修改该参数:

```sql
hive> set hive.exec.dynamic.partition.mode=nonstrict;
```

---

有时即使这样设置了,并且MR的任务也确实跑起来了,但是在跑到一半的时候还是会异常终止:

```
INFO  : 2019-08-29 13:42:29,086 Stage-1 map = 0%,  reduce = 0%
ERROR : Ended Job = job_local251946303_0002 with errors
Error: Error while processing statement: FAILED: Execution Error, return code 2 from org.apache.hadoop.hive.ql.exec.mr.MapRedTask (state=08S01,code=2)
```

是因为还有一个设置项 `hive.exec.max.dynamic.partitions.pernode` ,其默认值为`100` .意为每个每个MR节点最多可以创建多少个动态分区.当所执行的HQL语句设计的分区总数量>MR节点数*该值时,必然会造成无法执行的后果.需要适当增大该值:

```sql
hive> set hive.exec.max.dynamic.partitions.pernode=500;
```