---
layout: post
title:  "python中*和**的使用"
date:   2019-05-09 11:30:00 +0800
category: python
---

调用函数时用`*`：

```python
def test1(a,b,c):
    print(a,b,c)

args=(1,2,3)
test1(*args)
```

输出：

```python
1 2 3
```

---

调用函数时用`**`：

```python
def test2(d,e,f):
    print(d,e,f)

kwargs = {'d':4,'e':5,'f':6}
test2(**kwargs)
```

输出：

```python
4 5 6
```

---

定义函数时用`*`：

```python
def test3(*args):
    print(args)

test3(7,8,9)
```

输出：

```python
(7, 8, 9)
```

---

定义函数时用`**`：

```python
def test4(**kwargs):
    print(kwargs)

test4(g=10,h=11,i=12)
```

输出：

```python
{'g': 10, 'h': 11, 'i': 12}
```