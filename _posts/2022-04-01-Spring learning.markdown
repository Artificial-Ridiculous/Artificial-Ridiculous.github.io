---
layout: post
title:  "Spring learning"
date:   2022-04-01 15:25:00 +0800
category: Java
---

**以下所有代码均已脱敏**

以一个Spring实现的Swagger为例，一个典型的实现结构是：

```java
- 包
    - controller  // 负责对外暴露接口，定义HTTP请求类型和拼接的URL
        - public class InstanceController
            - @Autowired private InstanceService InstanceService;  // autowire一个service实例
            - public ReturnMessage getAllInstanceHosts(){}  // 通过实例调用对应的implement方法，将bean类型封装进Message返回
    - service  // 负责内部逻辑
        - public interface InstanceService  // 接口
            - List<String> getAllInstanceHosts();  // 定义方法
        - impl  // 实现接口
            - InstanceServiceImpl
                - @Autowired private InstanceMapper instanceMapper
                - @Override getAllInstanceHosts(){}
                    - return instanceMapper.getAllInstanceHosts()  // 这里的实例是从mapper的interface autowire
    - mapper  // DAO，是一个接口，封装impl中调用的方法，真正用SQL去实现getAllInstanceHosts(),依赖po中定义的bean（甚至返回类型就是bean）
        - public interface InstanceMapper
            - List<String> getAllInstanceHosts();
    - po  // 包装field和getter() setter()，一个个的bean
        - public class Instance {
            - field
            - getField(){}
```

调用过程：
- 外部通过swagger调用某个接口
    - 例如通过`get`请求调用`getAllInstanceHosts()`，拼接的域名是class级别的`@RequestMapping("/instaces")`加上方法级别的`@GetMapping("/hosts")`，最后生成的URL就应该是`http://{ip-address}:{port}/instaces/hosts`
- 调用请求打到controller
    - controller在对应的方法中，用面向对象的封装特性，先`@Autowired`一个服务接口`InstanceService`的某个实例`instanceService`出来，再调用它的相应的`getAllInstanceHost()`方法返回一个统一的`ReturnMessage`
    - 这就引出了服务接口`InstanceService`以及它的实现`InstanceServiceImpl`
- 服务
    - 接口`InstanceService`
        - 单纯的定义了关于`Instance`的各种方法
    - 实现`InstanceServiceImpl`
        - `@Autowired`相关bean(也就是`Mapper`)，获得与之相关的所有方法
        - 通过`Mapper`封装的各种SQL逻辑，简单地实现接口里声明的各种方法
- mapper
    - DAO接口，通过SQL真正实现方法，返回相应地bean
- po
    - 上述代码中各种定义的实体类(bean)