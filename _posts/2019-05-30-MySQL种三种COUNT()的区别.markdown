---
layout: post
title:  "MySQL种三种COUNT()的区别"
date:   2019-05-30 15:31:00 +0800
categories: MySQL
---

`count(*)`对行的数目进行计算，包含`NULL`；  
`count(column)`对特定的列的值具有的行数进行计算，不包含`NULL`值；  
`count()`还有一种使用方式，`count(1)`这个用法和`count(*)`的结果是一样的。