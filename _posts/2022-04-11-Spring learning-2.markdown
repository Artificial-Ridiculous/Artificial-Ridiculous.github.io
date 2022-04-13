---
layout: post
title:  "Spring learning-2"
date:   2022-04-11 15:31:00 +0800
category: Java
---

**以下所有代码均已脱敏**

### `@Value`注解用法
格式： 
`@Value(${application.properties.key:defaultValue})`
举例： 

```java
@Value("${cdc.check.status:false}")  
private String checkStatus;
```

---

### `JsonProperty`注解用法
格式： `@JsonProperty(value = “name”, required = false)`
举例： 

```java
@JsonProperty("another_name")
private String name;
```

上述写法应该是从`application.properties`配置文件中去找名为`status`的属性值，inject给checkStatus，如果找不到该属性值，则给默认值false

---


### Reference

- [view+control+service+dao+model](https://www.cnblogs.com/printN/p/7219444.html)
- [Spring @Component和@Value](https://codedec.com/tutorials/component-value-annotation-in-spring/)
- [@Value是如何处理占位符${}数据的](https://www.jianshu.com/p/a54535f9c640)