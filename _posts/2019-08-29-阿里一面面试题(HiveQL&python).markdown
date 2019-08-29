---
layout: post
title:  "阿里一面面试题(HiveQL&python)"
date:   2019-08-29 23:15:00 +0800
category: SQL
---

## SQL相关

1.假设有三个表A、B、C，每个表有一个字段为user_id(int)
用SQL找出 A表 中不在B表，但是在C表的所有user_id 

```sql
from
(
    select A.user_id as user_id 
    from A left join B 
    on A.user_id=B.user_id 
    where B.user_id is null
)t
select t.user_id 
from t left semi join C.user_id 
on t.user_id=C.userid
```

---

2.假设有表A，其中有两个字段，分别为 user_id(用户id，int), stay_time(单次停留时长，double)。  
注： 同一个用户可能有多行记录
用SQL求出表A的 用户数，人均停留时长

```sql
select count(distinct user_id),sum(stay_time)/count(distinct user_id) 
from A
```

---

3.假设有表A，其中有三个字段，分布为subject_id(科目id), user_id(学生id), score(分数)  
注: 同一个学生在多个科目下有分数
用SQL找出每个科目下分数前10的学生id

```sql
from
(
    select user_id
    ,subject_id,row_number() over(partition by subject_id order by score desc) as rank 
    from A
)t
select t.user_id,t.subject_id from t where t.rank<=10
```

---

4.假设有表A，其中有两个字段，content_id(内容id), pool_id_list(每个内容所在的池子id列表，以,分割)
注: 每个内容可能在一个或者多个池子里，表A里每个内容id只有一行数据
用SQL求出每个pool_id下的内容数。注意某些池子下的内容id非常多，有数据倾斜问题

```sql
from
(
    select content_id,pool_id
    lateral view explode(pool_id_list,",") as pool_id 
    from A 
    group by content_id,pool_id,pool_id
)t
select pool_id,sum(1) from t group by pool_id
```

---

## Python相关
1.用python输出斐波拉契数列

```python
def fib(n):
    res = [1,1]
    for i in range(2,n):
        res.append(res[i-1]+res[i-2])
    print(res)
```

---

2.用python实现: 删除一个list里面重复元素，且保持原有顺序（重复元素取第一个）

```python
lenl = len(l)
i = 0
while(i+1 < lenl):
    if l[i]==l[i+1]:
        l.pop(i+1)
        lenl-=1
    else:
        i+=1
```




