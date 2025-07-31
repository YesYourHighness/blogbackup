---
title: Spring进阶
date: 2020-08-25 11:03:22
tags:
- Spring
- 循环依赖
- Spring三级缓存
categories:
- Spring
---

<center>
引言：Spring回顾与进阶
</center>

<!--more-->

# Spring回顾（未完成）

## Spring

Spring是流行的AOP、IOC核心框架，核心组件有七个：Spring Context、Spring Core、Spring Web、Spring MVC、Spring AOP、Spring Dao、Spring ORM

### 1、为什么要用Spring？

- 低侵入式：代码污染极低
- 集中管理：IoC让相互协作的组件保持松散的耦合
- 代码复用：AOP编程允许你把遍布于应用各层的功能分离出来形成可重用的功能组件
- 对主流框架的支持

### 2、什么是IOC？

IOC，控制反转，与DI 依赖注入其实描述的是一个事情。

IOC控制反转，就是将对bean的控制权，转交给容器，让容器来帮助我们管理；

DI依赖注入，就是将有关对象的信息，注入到容器中，让容器帮我们创建对象，在Spring中就是用`@Autoweired`，`populateBean`来帮我们完成的属性注入。

容器是什么呢？

​		容器是用来存储对象的，在Spring中，有三级缓存，`singletonObejcts`用来存放完整的bean对象，bean从创建到销毁的过程都是由容器来管理的。

> 容器的创建过程

1. 向`bean`工厂中设置一些参数（`BeanPostProcessor`，`Aware`接口的子类）等
2. 加载解析Bean对象，准备要创建的bean对象的定义对象`beanDefinition`（xml或者注解的解析过程）
3. `beanFactoryPostProcessor`的处理（扩展点，`PlaceHolederConfigureSupport`，`ConfigurationClassPostProcessor`）
4. `beanPostProcessor`注册功能，方便后续对bean完成具体的功能扩展
5. 通过反射将`beandefinition`对象实例化成具体的bean对象
6. bean对象的初始化过程（填充属性，调用Aware方法、调用`BeanPostProcessor`前置处理方法、调用`init-method`、调用`BeanPocessor`的后置处理方法）
7. 生成完成的bean对象，就可以通过getBean直接获取
8. 销毁过程

---

IOC的底层实现：

1. 先通过`createBeanFactory`创建出一个Bean工厂（`DefaultListableBeanFactory`）
2. 开始循环创建对象，因为容器中的bean默认都是单例的，所以优先通过`getBean`、`doGetBean`从容器中查找，找不到的话
3. 通过`createBean`、`doCreateBean`方法，以反射的方式创建对象，一般情况下使用的是无参的构造方法（`getDeclareConstructor`、`newInstance`）
4. 进行对象的属性填充`populateBean`
5. 进行其他的初始化操作（`initializingBean`）

### 3、什么是AOP？

AOP 面向切面，用于将那些与业务无关的代码，比如说日志、事务、权限验证等内容，抽成一个切面，降低代码的耦合程度

**AOP的具体实现**，就是代理模式的使用，代理模式分为静态代理和动态代理，在Spring中，静态代理为AspectJ、动态代理为JDK动态代理和CG lib动态代理

### 4、JDK动态代理和CG lib动态代理的区别

**JDK动态代理：**

​		只能代理那些具有接口的实现类，如果一个类没有实现任何接口，那么JDK动态代理将不能实现

具体的实现原理：动态代理类实现`InvocationHandler`接口，重写`invoke`方法（三个参数：代理类、代理的方法、方法参数）

```java
public class ProxyInvocationHandler implements InvocationHandler {
    // 真实对象
    private Object target;
    public void setTarget(Host target) {
        this.target = target;
    }
    // 获得代理类
    public Object getProxy(){
        return Proxy.newProxyInstance(
                this.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                this
        );
    }
    @Override
    // 处理代理实例，返回对象
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("A操作");
        Object invoke = method.invoke(target, args);
        System.out.println("B操作...");
        return invoke;
    }
}
```

**CG lib动态代理：**

​		CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时**动态的生成指定类的一个子类对象**，并覆盖其中特定方法并添加增强代码，从而实现AOP

​		CGLIB是**通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的**

具体实现：实现`MethodInterceptor`接口，重写`intercept`方法

```java
/**
 * 注意是org.springframework.cglib.proxy这个包下的MethodInterceptor
 */
public class MyCglibProxy implements MethodInterceptor {
    private ProductDao productDao;

    public MyCglibProxy(ProductDao productDao) {
        this.productDao = productDao;
    }

    public Object creatProxy(){
        //1. 核心类
        Enhancer enhancer = new Enhancer();
        //2. 设置父类
        enhancer.setSuperclass(productDao.getClass());
        //3. 设置回调，类似于InvocationHandler这个类
        enhancer.setCallback(this);
        Object proxy = enhancer.create();
        return proxy;
    }

    @Override
    public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        if("save".equals(method.getName())){
            // 这里意思是 当方法为 save ，就启动权限检验功能
            System.out.println("权限检验");
            return methodProxy.invokeSuper(proxy,args);
        }
        return methodProxy.invokeSuper(proxy,args);
    }
}
```

### 5、Bean的作用域

**七种**作用域：

- `singleton`：在SpringIOC容器中仅存在一个`Bean`实例，`Bean`以单例的方式存在
- `prototype`：每次调用`getBean()`时都会返回一个新的实例
- `request`：每次HTTP请求都会创建一个新的`Bean`（仅适用在WebApplicationContext环境）
- `session`：同一个`HTTP Session`共享一个`Bean`，不同的`HTTP Session`使用不同的`Bean`（仅适用在WebApplicationContext环境）
- `global-session`：全局作用域，所有会话共享一个实例
- `application`：每个`ServletContext`创建一个实例
- `websocket`：每个`websocket`创建一个实例

### 6、Bean的生命周期

简单来说，Spring Bean的生命周期只有四个阶段：

实例化 `Instantiation` -> 属性赋值 `Populate ` -> 初始化 `Initialization `-> 销毁 `Destruction`

![Bean生命周期](https://github.com/YesYourHighness/MyPicStore/raw/master/Spring/lifeCycle.png)

具体过程：

1、**实例化Bean**：反射方式生成对象

- 对于`BeanFactory`容器，当客户向容器请求一个尚未初始化的`bean`时，或初始化`bean`的时候需要注入另一个尚未初始化的依赖时，容器就会调用`createBean`进行实例化。

- 对于`ApplicationContext`容器，当容器启动结束后，通过获取`BeanDefinition`对象中的信息，实例化所有的bean。

（区别就是，`Application`**一次性全载**入了所有的`bean`，而`BeanFactory`采用的是**延迟加载**）

2、**设置对象属性**（**依赖注入**）

​		实例化后的对象被封装在`BeanWrapper`对象中，紧接着，Spring根据`BeanDefinition`中的信息以及通过`BeanWrapper`提供的设置属性的接口完成属性设置与依赖注入。

3、**处理`Aware`接口**

​		Spring会检测该对象是否实现了`xxxAware`接口，通过Aware类型的接口，可以让我们拿到Spring容器的一些资源：

- 如果这个Bean实现了`BeanNameAware`接口，会调用它实现的`setBeanName(String beanId)`方法，传入`Bean`的名字；
- 如果这个Bean实现了`BeanClassLoaderAware`接口，调用`setBeanClassLoader()`方法，传入`ClassLoader`对象的实例。
- 如果这个Bean实现了`BeanFactoryAware`接口，会调用它实现的`setBeanFactory()`方法，传递的是Spring工厂自身。
- 如果这个Bean实现了`ApplicationContextAware`接口，会调用`setApplicationContext(ApplicationContext)`方法，传入Spring上下文；

4、 **`BeanPostProcessor`前置处理**

​		（这个阶段实现了AOP）

​		如果想对Bean进行一些自定义的前置处理，那么可以让Bean实现了`BeanPostProcessor`接口，那将会调用`postProcessBeforeInitialization(Object obj, String s)`方法。

5、`InitializingBean`

​		如果Bean实现了`InitializingBean`接口，执行`afeterPropertiesSet()`方法。

6、`init-method`

​		如果Bean在Spring配置文件中配置了`init-method `属性，则会自动调用其配置的初始化方法。

7、`BeanPostProcessor`后置处理

​		如果这个Bean实现了`BeanPostProcessor`接口，将会调用`postProcessAfterInitialization(Object obj, String s)`方法；

​		由于这个方法是在**Bean初始化结束时调用**的，所以**可以被应用于内存或缓存技术**；

---

以上几个步骤完成后，Bean就已经被正确创建了，之后就可以使用这个Bean了

---

8、`DisposableBean`：当Bean不再需要时，会经过清理阶段，如果Bean实现了`DisposableBean`这个接口，会调用其实现的`destroy()`方法；

9、`destroy-method`：最后，如果这个Bean的Spring配置中配置了`destroy-method`属性，会自动调用其配置的销毁方法

### 7、Spring中的Bean是线程安全的么？如果线程不安全，如何处理？

分不同类型来讨论：

1、对于`prototype`作用域的Bean，每次都创建一个新对象，也就是线程之间不存在Bean共享，因此不会有线程安全问题。

2、对于`singleton`作用域的Bean，所有的线程都共享一个单例实例的Bean，因此是**存在线程安全问题**的。但是如果单例Bean是一个无状态Bean，也就是线程中的操作不会对Bean的成员执行查询以外的操作，那么这个单例Bean是线程安全的。

> **有状态Bean(Stateful Bean)** ：就是有实例变量的对象，可以保存数据，是**非线程安全**的。（比如Model和View）
>
> **无状态Bean(Stateless Bean)**：就是没有实例变量的对象，不能保存数据，是不变类，是**线程安全**的。（比如Controller类、Service类和Dao等，这些Bean大多是无状态的，只关注于方法本身）

解决办法：

1. 最简单的办法，将`singleton`改为`prototype`
2. 还可以采用`ThreadLocal`存

### 6、Bean的实例化方式

- 构造器实例化（默认的无参构造方法）
- 静态工厂方法实例化
  - 配置`<bean factory-method="xxx"/>`
- 实例工厂方式实例化
  - 要配两个bean
  - 实例工厂的bean要配置`<bean id="xxx" factory-bean="xxx" factory-method="xxx">`

### 7、Bean的依赖注入方式

有三种：

- XML注入属性

- 注解注入属性
- 自动注入属性

#### XML注入属性

Spring有两种基于XML的注入方式：**设值注入、构造注入**

原理：Spring先调用对象的构造方法，然后通过反射调用`setter`方法给对象设置属性

> 设值注入（必须满足两点要求）

1. 有默认的无参构造方法
2. 有`setter`方法

然后bean文件配置子标签`<property>`即可

> 构造注入

要提供带**所有参数**的有参构造方法

然后bean文件配置子标签`<constrictor-arg>`即可

#### 注解注入属性

注解注入，就比较简单了，常见有

```java
@Component // 任何层次
@Repository // 数据访问层DAO
@Service // 业务层
@Controller // 控制层
@Autowired // 对Bean的属性变量、属性的setter方法及构造方法进行标注；
			// 作用是：可以与对应的注解处理器，完成自动配置工作
@Resource // 作用与Autowired一样，区别是Resource按name装配、Autowired按type装配
@Qualifier // 与Autowired配合，将默认的按Bean类型装配修改位按Bean实例名称装配
```

注解使用时，要开启自动扫描注解

```xml
<context:component-scan base-package="bean所在的包">
```

#### 自动注入属性

有些项目并没有使用注解方式开发，为了减少代码量，才有了这种方式

在`bean`中配置`autowire`属性即可

```xml
<bean id="xxx" class="xxx" autowire="xxx"/>
```

## Spring MVC

## Spring Boot

# Spring进阶

## 循环依赖问题

> 循环依赖问题：类与类之间的依赖关系形成了闭环，就会导致循环依赖问题的产生

**循环依赖的理论依据其实是基于Java的引用传递**：当我们获取到对象的引用时，对象的field或则属性是**可以延后**设置的（**但是构造器**必须是在获取引用之前）

循环依赖问题在Spring中主要有三种情况：

- 通过**构造方法**进行依赖注入时产生的循环依赖问题。
- **通过`setter`方法**进行且是在**原型模式**下产生的循环依赖问题。
- **通过`setter`方法**进行依赖注入且是在**单例模式**下产生的循环依赖问题。

在Spring中，只有第三种问题被解决了，解决的办法是三级缓存

## Spring的三级缓存

​		三级缓存都是Map类型，这三个缓存是**彼此互斥**的，不会针对同一个Bean的实例同时存储

源码如下：**很重要**

```java
@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // 从一级缓存获取单例对象
    Object singletonObject = this.singletonObjects.get(beanName);
    // isSingletonCurrentlyInCreation 此方法是判断当前的单例bean是不是在创建中
    // 创建中是什么意思？就是A中用了B，而B还没被创建，所以得先去创建B
    // 此时的A的状态，就属于创建中
    // 这里判断第一级缓存没有这个对象并且这个对象还在创建中，才会继续执行以下代码
    if (singletonObject == null && 
        isSingletonCurrentlyInCreation(beanName)) {
        // 上锁，锁住第一级缓存
        synchronized (this.singletonObjects) {
			// 从二级缓存获取单例对象
            singletonObject = this.earlySingletonObjects.get(beanName);
            // allowEarlyReference
            // 是否允许从singletonFactorys中用getObejct拿到bean
            // 从第二级缓存也拿不到
            if (singletonObject == null && allowEarlyReference) {
                // 从第三级缓存获取
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                
                if (singletonFactory != null) {
                    // 通过单例工厂获取单例bean
                    singletonObject = singletonFactory.getObject();
                    // 下面两行代码是，从三级缓存移动到二级缓存
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return singletonObject;
}
```

如果调用`getBean`，则需要从三个缓存中依次获取指定的Bean实例

1. 一级缓存：`Map<String, Object> singletonObjects`
   - 用来存放单例模式下，外部创建的Bean（就是我们创建的`bean`，**已经创建完成了**）
   - key为bean的名称
   - value为bean的实例对象
2. 二级缓存：`Map<String, Object> earlySingletonObjects`
   - 用于存储单例模式下创建的Bean实例（注意：这个Bean是被提前暴露的引用，**该Bean还在创建中**）
   - Spring框架**内部逻辑使用该缓存**
   - 作用：为了解决第一个`classA`引用最终如何替换为代理对象的问题（如果有代理对象）
   - k-v与一级缓存相同
3. 三级缓存：`Map<String, ObjectFactory<?>> singletonFactories`
   - 通过`ObjectFactory`对象来存储单例模式下提前暴露的Bean实例的引用（正在创建中）
   - Spring框架**内部逻辑使用该缓存**
   - **此缓存是解决循环依赖最大的功臣**
   - Key 存储`bean`的名称
   - Value存储`ObjectFactory`，**该对象持有提前暴露的`bean`的引用**

> 为什么第三级缓存要使用`ObjectFactory`？

这个图非常棒，从左到右，从上往下看。

![图源见相关链接](http://img.yesmylord.cn//image-20210825200838536.png)

如果仅仅是解决循环依赖问题，使用二级缓存就可以了，但是**如果对象实现了AOP**，**那么注入到其他bean的时候，并不是最终的代理对象，而是原始的**

所以需要通过三级缓存的`ObjectFactory`才能提前产生最终的需要代理的对象

> 什么时候将Bean的引用提前暴露给第三级缓存的`ObjectFactory`持有？

时机就是在**实例化之后，依赖注入之前**，完成**将Bean的引用提前暴露给第三级缓存**

（**这就是解决了第三种循环依赖的关键**）

```java
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized (this.singletonObjects) {
        // 如果一级缓存没有才会去add
        if (!this.singletonObjects.containsKey(beanName)) {
            // 放入三级缓存中
            this.singletonFactories.put(beanName, singletonFactory);
            // 从二级缓存移除
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }
}
```

![理解图示](http://img.yesmylord.cn//image-20210826091358496.png)

> 为什么构造方法的循环依赖问题解决不了？

​		经过上一个问题，应该可以了解了，循环依赖问题是在实例化后、依赖注入之前，通过提前暴露给`SingletonFactory`的`ObjectFactory`

​		而构造方法的循环引用，还不会进入这个阶段，所以不能解决这种情况的问题。

> 如何解决构造方法循环依赖问题？

​		对于类A和类B都是通过构造器注入的情况，可以在A或者B的构造函数的形参上加个`@Lazy`注解实现延迟加载

（这应该算程序员外部解决内部循环依赖的一种办法）

> `@Lazy`实现原理：当实例化对象时，如果发现参数或者属性有`@Lazy`注解修饰，**那么就不直接创建所依赖的对象了，而是使用动态代理创建一个代理类**

​		比如，类A的创建：`A a = new A(B)`，需要依赖对象B，发现构造函数的形参上有`@Lazy`注解，那么就不直接创建B了，而是使用动态代理创建了一个代理类`B1`，此时A跟B就不是相互依赖了，变成了A依赖一个代理类B1，B依赖A

​		但因为在注入依赖时，类A并没有完全的初始化完，实际上注入的是一个代理对象，只有当他首次被使用的时候才会被完全的初始化。

## BeanFactory与FactoryBean的区别

相同点：都是用来创建对象的

不同点：使用`BeanFactory`创建对象时，必须要遵循严格的生命周期流程，太复杂了，如果想简单的自定义某个对象的创建，同时创建完成的对象想交给Spring来管理，就要实现`FactoryBean`接口

## Spring中用到的设计模式

- 单例模式
- 原型模式
- 工厂模式：BeanFactory
- 模板模式：HashMap
- 代理模式
- 策略模式：XMLBeanDefinition
- 观察者模式：listener、event
- 适配器模式：Adapter
- 责任链模式：使用aop会生成一个拦截器链

# Spring事务

## Spring事务的实现方式

当我们使用Spring来支持事务时，多半是由于数据库不支持事务（Mysql的Innodb是支持事务的）

Spring支持两种方式实现事务：

- 编程式事务：通过 `TransactionTemplate`或者`TransactionManager`手动管理事务，实际应用中很少使用
- 声明式事务：通过AOP实现，比如`@Transactional`

### 编程式事务

TransactionTemplate

```java
@Autowired
private TransactionTemplate transactionTemplate;

public void testTransaction() {
        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {
                try {
                    // ....  业务代码
                } catch (Exception e){
                    //回滚
                    transactionStatus.setRollbackOnly();
                }
            }
        });
}
```

TransactionManager

```java
@Autowired
private PlatformTransactionManager transactionManager;

public void testTransaction() {

  TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
          try {
               // ....  业务代码
              transactionManager.commit(status);
          } catch (Exception e) {
              transactionManager.rollback(status);
          }
}
```

### 声明式事务

使用注解`@Transactional`

- 方法：推荐将注解使用于方法上，不过需要注意的是：**该注解只能应用到 public 方法上，否则不生效。**
- 类：如果这个注解使用在类上的话，表明该注解对该类中所有的 public 方法都生效。

| 属性名      | 说明                                                         |
| :---------- | :----------------------------------------------------------- |
| propagation | 事务的传播行为，默认值为 REQUIRED，可选的值在上面介绍过      |
| isolation   | 事务的隔离级别，默认值采用 DEFAULT，可选的值在上面介绍过     |
| timeout     | 事务的超时时间，默认值为-1（不会超时）。如果超过该时间限制但事务还没有完成，则自动回滚事务。 |
| readOnly    | 指定事务是否为只读事务，默认值为 false。                     |
| rollbackFor | 用于指定能够触发事务回滚的异常类型，并且可以指定多个异常类型。 |

> @Transaction是如何实现的？

通过AOP，

如果一个类或者一个类中的 public 方法上被标注`@Transactional` 注解的话，**Spring 容器就会在启动的时候为其创建一个代理类**，在调用被`@Transactional` 注解的 public 方法的时候，实际调用的是，`TransactionInterceptor` 类中的 `invoke()`方法。这个方法的作用就是在目标方法之前开启事务，方法执行过程中如果遇到异常的时候回滚事务，方法调用完成之后提交事务

### 自调用问题

```java
@Service
public class MyService {

private void method1() {
     method2();
     //......
}
@Transactional
 public void method2() {
     //......
  }
}

```

使用注意：如果事务方法与调用方法处于同一个类，事务会失效，比如上面的代码。

原因是因为：**代理对象的判断逻辑是其他类的方法调用时会产生事务，本类的调用逻辑则不会**

为了解决自调用问题，可以：

- 避免同一类中自调用
- 使用 AspectJ 取代 Spring AOP 代理



### Spring实现事务的关键接口

Spring的事务实现方式，遵守了TCC（Try、Commit、Cancel）的方式：

- `PlatformTransactionManager`：（平台）**事务管理器**（TCC规定了事务管理器和资源管理器），是事务运行机制的管理者，是一个接口，为JDBC、JPA、Hibernate等等提供了事务管理的接口，具体还需要各自实现。

```java
public interface PlatformTransactionManager {
    //获得事务
    TransactionStatus getTransaction(@Nullable TransactionDefinition var1) throws TransactionException;
    //提交事务
    void commit(TransactionStatus var1) throws TransactionException;
    //回滚事务
    void rollback(TransactionStatus var1) throws TransactionException;
}
```

- `TransactionDefinition`：定义事务的信息，比如**传播方式、隔离级别**

- `TransactionStatus`：事务运行状态。

### 事务传播

在 Spring 框架中，事务传播（transaction propagation）**定义了当一个事务方法被另一个事务方法调用时，事务的行为方式**。

```java
@Transactional(propagation = Propagation.REQUIRED)
public void methodA() {
    // some logic
    serviceB.methodB();
    // some logic
}
```

有七种传播方式，他们总体来说定义了在原本有事务时，是选择加入、创建新事物、创建嵌套事物、还是报错；在原本没有事务时，是选择新建事务，还是直接以非事务的方式执行。

- **REQUIRED**：（**默认**）
  - **存在事务：加入**
  - 不存在事务：新建
- **REQUIRES_NEW**：
  - 存在事务：新建（原有的事务会被挂起，直到新事务完成）两个事务之间互不干扰，各自回滚各自的，但是如果事务抛出了异常且没有捕获，可能会导致两个事务都回滚
  - 不存在事务：新建
- **NESTED**：
- 存在事务：创建一个**嵌套事务**。（外部事物失败，内部也会回滚；内部事务失败，外部无影响）

- 不存在事务：新建。
- **SUPPORTS**：
  - 存在事务：加入
  - 不存在事务：以非事务方式执行
- **NOT_SUPPORTED**：
  - 存在事务：挂起原事务，以非事务执行
  - 不存在事务：以非事务执行
- **MANDATORY**：
  - 存在事务：加入
  - 不存在事务：抛出异常。
- **NEVER**：
  - 存在事务：抛出异常
  - 不存在事务：非事务方式执行

### 隔离级别

RU、RC、RR、Serial

```java
public interface TransactionDefinition {
    ......
    int ISOLATION_DEFAULT = -1; // 默认方式会使用对应数据库的默认方式，比如mysql RR、oracle RC
    int ISOLATION_READ_UNCOMMITTED = 1;
    int ISOLATION_READ_COMMITTED = 2;
    int ISOLATION_REPEATABLE_READ = 4;
    int ISOLATION_SERIALIZABLE = 8;
    ......
}
```

### 事务只读

```java
public interface TransactionDefinition {
    ......
    // 返回是否为只读事务，默认值为 false
    boolean isReadOnly();
}
```

事务只读属性，如果设置为true，表面当前事务只会涉及到一些查询操作

> 有什么作用？在开启后，数据库会有一些优化手段

比如mysql每个连接默认都是autocommit的，每一个sql语句都会开启一个事务，如果不开启事务去执行，那么每次都读取的是最新值，但是如果涉及到一些批量查询，可能就需要前后一致性，就可以开启事务，保证多个sql在一个事物内执行！

- 对于一次性执行单条查询语句来说，没必要开启
- 对于一次性执行多条查询语句，**例如统计查询，报表查询**，在这种场景下，多条查询 SQL 必须保证整体的读一致性，否则，在前条 SQL 查询之后，后条 SQL 查询之前，数据被其他用户改变，则该次整体的统计查询将会出现读数据不一致的状态，此时，应该启用事务支持

# 相关链接

1. [三级缓存很不错的blog](https://blog.csdn.net/a745233700/article/details/110914620)
2. Java EE 企业级开发教程



