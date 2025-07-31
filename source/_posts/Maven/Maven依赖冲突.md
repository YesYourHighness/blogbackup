---
title: Maven依赖冲突
date: 2023-07-23 10:52:14
tags: 
- Maven
categories:
- Maven
---

<center>
引言： Maven解决依赖冲突的两种方式。
</center>

<!-- more -->

# Maven依赖冲突

## Maven产生依赖冲突的原因

产生的原因：当使用Maven引入包时，包也会引入其他包，因此我们导入的包之间，可能其间接依赖的包之间版本不同，在启动后，可能会出现以下“**找不到**”的异常：

- `java.lang.NoSuchMethodError`
- `java.lang.ClassNotFoundException`
- `java.lang.NoClassDefFoundError`

## Maven的选择原则

Maven有两个选择原则：

- **路径最短原则**：假设`A->C1`，`B->D->C2`，Maven会选择依赖路径短的一个，即C1
- **优先声明原则**：假设`A->C1`，`B->C2`，Maven会选择优先声明的一个，即C1

## Maven依赖冲突的解决方式

IDEA安装插件`Maven Helper`，使用插件进行分析，我们自己手动排除`exclusion`那些需要的包。

打开`pom.xml`文件，点击左下角`Dependency Analyzer`

此时就可以进行选择了，对不需要的包我们可以直接`Exclude`

在Pom文件中我们可以在`dependency`中使用标签`exclusion`

```xml
<dependency>
    <groupId>xxx</groupId>
    <artifactId>xxx</artifactId>
    <version>1.0</version>
    <exclusions>
        <exclusion>
            <groupId>xxx</groupId>
            <artifactId>xxx</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```