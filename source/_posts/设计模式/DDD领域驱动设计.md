---
title: DDD领域驱动设计
date: 2024-12-25 21:39:20
tags:
- 设计模式
- DDD
categories:
- 设计模式
---

<center>
    引言：从OOP，到经典的设计模式，再到SOLID，再到DDD领域驱动设计
</center>

<!-- more -->

# 编程思想：从OOP到DDD

# OOP

我们经常接触OOP的语言：Java、C++，他们的核心就是**面向对象**

区别于面向过程编程，OOP的重点在于从实际的业务模型中抽取出**类**来

并且设计时要满足OOP的四大特点：**封装、继承、多态、抽象**（抽象很多人认为是基本原则）

- 封装：将类的属性与行为封装在类中，只对外暴露公开的接口
- 继承：从现有的类中继承属性和行为，从而实现代码复用和扩展
- 多态：父类可以有不同的实现的子类，他们有共同的属性和行为在父类中，子类自身体现不同之处
- 抽象：提供抽象的接口定义，而不是具体的实现（这个很多人认为是基本的原则）

# Design Pattern

从OOP开发之后，引申出了很多设计模式（关于设计模式，看[此篇博客](https://www.yesmylord.cn/2022/02/14/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8Fplus/)）

- 创建型：单例、工厂、建造者、原型
- 结构型：代理、桥接、装饰者、适配器、门面、组合、享元
- 行为型：观察者、模版、策略、职责链、状态、迭代器、访问者、备忘录、命令、解释器、中介

从不同的设计模式思想中，可以抽象出几个比较核心的，基本都遵守了设计思想：SOLID

# SOLID

所谓SOLID指的是：

- **S单一职责原则**（Single Responsibility Principle，SRP）：每个类只有一个职责
- **O开放封闭原则**（Open/Closed Principle，OCP）：软件实体（类、模块、函数等）应该对扩展开放，对修改封闭
- **L里氏替换原则**（Liskov Substitution Principle，LSP）：子类应该能完全实现父类的行为
  - 举个违反LSP原则的例子：比如父类`bird`有行为`fly`，他的子类有一个鸵鸟，而鸵鸟是不会飞的，因此最好的办法是，将`bird`拆为会飞的鸟和不会飞的鸟，将`fly`这个行为给予会飞的鸟
- **I接口隔离原则**（Interface Segregation Principle，ISP）：接口应该被拆成最小，以便于客户端进行最小化的调用
- **D依赖倒置原则**（Dependency Inversion Principle，DIP）：模块间的依赖使用抽象实现（接口或是抽象类），而不去使用具体的实现

# DDD领域驱动设计

## DDD 是什么

以上的一些设计思想，都局限在了一些具体的实现上，在现在的开发过程中，往往会引入一大堆中间件、一大堆存储池，而旧的设计思想可能会不太够用，因此DDD是非常好用的工具，去剖析复杂的业务领域

> **领域驱动设计 DDD，Domain-Driven Design**
>
> DDD是一套方法论，是优秀设计方式的缩影聚合，你可以在DDD中看到一大堆设计思想的缩影，比如DP、OP、工厂模式

DDD依赖于是对传统的三层设计的一种改进方案，我们先从传统的三层开始介绍：

传统的三层：

![传统三层](http://img.yesmylord.cn//image-20240530222031333.png)

- **Presentation 表示层**：比如前端UI相关的代码，后端的Controller以及与前端进行交互的代码，都属于表示层
- **Application 应用层**：就是Service，常写的业务逻辑、数据校验逻辑，CRUD
- **Infrastructure 基础设施层**：包括DAO相关、存储相关、消息队列相关等等的代码。

传统的三层之间，是从上到下依次依赖的，即Controller依赖于Service、Service依赖于DAO

存在的问题：

1. Application可能会过于臃肿，不利于新人接受与维护
2. Infrastructure可能会过分依赖于某个组件，比如过分依赖于mysql，后期打算切换数据库或是消息队列，但因为耦合过于深入，导致放弃

**DDD核心概念**：

> 关于此部分的概念可以结合下一节的Demo配合理解

DDD优化后，提出Domain层，而且调整了他们之间的依赖关系，DDD的四层结构：

![DDD四层](http://img.yesmylord.cn//image-20240530222103293.png)

- **Presentation 表示层**：基本不变
  - Presentation与Application交互使用REST DTO
- **Application应用层**：接入相同业务的不同场景（use case），比如电商系统的用户登录和店家登录
  - Application与Domain交互使用Entity，此处的Entity不同于POJO（**贫血模型**），可能还会含有校验逻辑（**充血模型**）
- **Domain领域层**：Service层可能存在一些逻辑是固定不变的（业务逻辑），这些放在Domain层，而且此处基本全部面向接口
  - **Aggregate** 聚合：当一个操作会涉及到多个对象的状态发生改变，那么我们应该将这一群对象聚合为一个对象，对象内部对其进行操作，只对外暴露一个接口（即暴露聚合根）聚合可以保证一群对象的状态一致
    - **VO**值对象（也称为DP，即Domain原语）：抽象并封装自检和隐性属性的计算逻辑，且这些属性**无状态**
    - **Entity** 实体：抽象并封装单对象**有状态**的逻辑

  - **Domain Service**：可能存在一些逻辑是固定不变的（业务逻辑）

- **Infrastructure基础设施层**（也叫**数据层**）：此层依赖于其他三层，传输的数据是DataBase DTO，这一层需要提供具体的实现，比如Mapper、RPC
  - **Repository** 存储：抽象并封装外部数据的访问逻辑（比如DAO、RPC等）
  - 还包括网关、缓存、第三方工具等

> 领域层中的VO和Entity的区别在于：
>
> - 是否有状态，什么是状态？即在系统的整个运行过程中，可能会发生变化的对象
> - 是否唯一：实体每一个都有唯一的标识符
> - 是否有生命周期
>
> 比如在一个抢票系统中，如果抢到了票，并且票固定了座位，那么这个座位就是有状态的；如果抢到票就可以去，随便坐，那么座位此时就是无状态的。

## DDD 防腐层

防腐层（Anti-Corruption Layer）的位置通常在**应用层和基础设施层之间**，由于基础设施可能会更换（比如Mysql换为了Mq），但应用层需要一个稳定的接口，因此就提出了防腐层。

防腐层有时被看做基础设施层的一部分，应用层通过防腐层与基础设施层进行交互，无需了解具体细节。

引入防腐层可以进一步降低耦合度，可以说本质就是额外加了一层接口。

## DDD 代码模型

具体来看，代码的分包基本是这样的：

- interfaces（表示层）
  - Controller：HTTP服务
  - API接口：其他的DTO服务
- application（应用层）
  - Service：业务逻辑（大的业务需求，需要多个domain service组合实现）
  - facade：将用户请求委托给一个或多个应用服务进行处理。
  - event：包括两个子目录publish和subscribe。主要存放事件发布、订阅的相关代码(事件处理的核心业务逻辑在领域层实现)
- domain（领域层）：
  - service：领域服务
  - entity：实体类
  - Repository：做entity到DO的类型转换，调用DAO
- infrastructure（基础设施层）：
  - config：基础的配置
  - common：公共组件
  - dao：DB操作

## DDD demo

现在有这样一个场景：[demo来自于视频](https://www.bilibili.com/video/BV11q4y1q74f/?spm_id_from=333.788&vd_source=c8709f8826bf296abab8aeee72b0e338)

> 假设现在要做一个**数据统计系统**：
>
> 市场专员输入客户的**姓名**和**手机号**，
>
> 根据客户手机号的归属地和运营商将客户群体分组，分配给相应的销售组，由销售组跟进后续业务

我们瀑布式开发马上就可以实现这个接单的业务：

1. 输入用户名、手机号
2. 校验参数
3. 获取手机号的归属地和运营商编号（查数据库）
4. 获取到分组编号
5. 构建客户对象，存入数据库（写数据库）

大致代码如下：

```java
public class RegisterServiceImpl implement RegisterService{
    private SaleRepository saleRepo;
    private UserMapper userMapper;
    
    public User register(String name, String phone) throw ValidationException{
        // 参数校验
        if(name == null || name.length() == 0){
            throw new ValidationException("name");
        }
        if(phone == null || isValidPhone(phone)){
            throw new ValidationException("name");
        }
        String areaCode = getAreaCode();
        String opCode = getOpCode();
        Sale sale = saleRepo.findRep(areaCode, opCode);
        // 创建用户、落盘
        User user = new User();
        user.name = name;
        user.phone = phone;
        if(sale != null){
            user.saleId = sale.id;
        }
        return userMapper.insert(user);
    }
}
```

这样的代码会有几个问题：

1. 输入参数是两个String，他人调用可能会出现顺序颠倒的问题；
2. 如果之后扩展注册方式，比如使用身份证注册，也会传入一个String，当前代码不好扩展
3. 参数校验逻辑写在了register逻辑里面，如果校验规则发生改变，我们有需要重新修改这段代码
4. 这个方法的目的是为了注册，而获取手机号的归属地和运营商好像使注册业务不再纯粹（这种为了调用某个API，而去补充参数的方式，称为**胶水操作**）

DDD的一大思想就是**充血模型**，即我们其实可以定义一个类PhoneNumber，其内部进行逻辑校验（所谓贫血模型就是普通的POJO类，即只有get、set方法）

我们给类PhoneNumber加了校验逻辑、获取手机号的归属地和运营商的逻辑，让业务逻辑变得纯粹，PhoneNumber这种充血模型设计的Bean，称为DP（Domain Primitive 领域原语），其可以自我验证，拥有自我的行为，是领域的最小组成部分

```java
public class PhoneNumber{
    private final String number;
    private final String pattern = "^0?[1-9]{2,3}-?\\d{8}$";
    public getNumber(){
        return number;
    }
    public PhoneNumber(String number){
        if(!isValidPhone(number)){
            throw new ValidationException("number格式错误");
        }
        this.number = number;
    }
    public String getOpCode(){}
    public String getAreaCode(){}
}
```

这样业务逻辑就变的十分清晰

```java
public class RegisterServiceImpl implement RegisterService{
    private SaleRepository saleRepo;
    private UserMapper userMapper;
    
    public User register(String name, PhoneNumber phone) throw ValidationException{
        Sale sale = saleRepo.findRep(phone.getAreaCode(), phone.getOpCode());
        User user = new User();
        user.name = name;
        user.phone = phone;
        if(sale != null){
            user.saleId = sale.id;
        }
        return userMapper.insert(user);
    }
}
```

DP三原则：

- 让隐性的概念显性化（比如归属地和运营商其实是Phone的隐性属性）
- 让隐性的上下文显性化
- 封装多对象行为

但目前我们的实现，**外部依赖**十分严重，比如我们直接依赖了MybatisPlus，我们在业务逻辑中，直接使用了Mapper创建了User，并且还实际存储了User对象

> 外部依赖：不属于当前Domain的设施和服务都属于外部依赖
>
> 比如数据库、schema、RPC、ORM、中间件

如果之后我们想切换中间件，那么此处的实现就仍需要改动，此处可以进行DP（依赖倒置）原则，面向接口而不是具体的实现

```java
public class RegisterServiceImpl implement RegisterService{
    private SaleRepository saleRepo;
    private UserRepository userRepo;
    
    public User register(String name, PhoneNumber phone) throw ValidationException{
        Sale sale = saleRepo.findRep(phone.getAreaCode(), phone.getOpCode());
        
        User user = new User(name, phone, sale.Id);

        return userRepo.save(user);
    }
}
// userRepo 是一个接口，可以有不同的实现，比如使用MybatisPlus实现
```

而且此时，我们是直接通过new创建的对象，如果为了更加通用，可以使用工厂类对对象进行创建。

## DDD总结

DDD是一套开发可拓展性强、可维护性高的应用的一套方法论，其包含了很多设计模式的设计思想，比如OP、DP、工厂模式、面向接口等等。

DDD的关键思想是：基础设施不再是最底层的直接依赖，而是作为旁系进行依赖

DDD关键的概念是：VO、Entity、聚合根

DDD常用的方式就是：充血模型，VO和Entity都可以进行充血









