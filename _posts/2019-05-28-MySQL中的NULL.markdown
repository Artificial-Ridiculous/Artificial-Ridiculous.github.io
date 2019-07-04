---
layout: post
title:  "MySQL中的NULL"
date:   2019-05-28 16:58:00 +0800
category: MySQL
---

# 标题

A值|判断|结果
-|-|-
10|A IS NULL|FALSE
10|A IS NOT NULL|TRUE
NULL|A IS NULL|TRUE
NULL|A IS NOT NULL|FALSE
10|A = NULL|UNKNOWN (FALSE)
10|A != NULL|UNKNOWN (FALSE)
NULL|A = NULL|UNKNOWN (FALSE)
NULL|A != NULL|UNKNOWN (FALSE)
NULL|A = 10|UNKNOWN (FALSE)
NULL|A != 10|UNKNOWN (FALSE)
NULL|A > 10|UNKNOWN (FALSE)

`NULL`排序时比其他数据都大（索引默认是降序排列，小→大）， 所以NULL值总是排在最后。   
`IS NULL` 和`IS NOT NULL` 是不可分割的整体。  
任何和`NULL` 的比较操作，如`<>`、`=`、`<=`等都返回`UNKNOWN`（这里的`UNKNOWN`就是`NULL`，它单独使用和布尔值`FALSE`类似）：

```
+--------------+
|   10 is NULL |
|--------------|
|            0 |
+--------------+
```

```
+----------------+
|   NULL is NULL |
|----------------|
|              1 |
+----------------+
```

```
+-------------+
|   10 = NULL |
|-------------|
|      <null> |
+-------------+
```

```
+--------------+
|   10 != NULL |
|--------------|
|       <null> |
+--------------+
```

```
+---------------+
|   NULL = NULL |
|---------------|
|        <null> |
+---------------+
```

```
+-------------+
|   NULL = 10 |
|-------------|
|      <null> |
+-------------+
```

```
+--------------+
|   NULL != 10 |
|--------------|
|       <null> |
+--------------+
```

```
+-------------+
|   NULL > 10 |
|-------------|
|      <null> |
+-------------+
```


