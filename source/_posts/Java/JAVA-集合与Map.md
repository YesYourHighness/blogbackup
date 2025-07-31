---
title: JAVA-集合与Map
date: 2019-08-14 17:17:04
tags: 
- Java
categories: 
- 后台
- Java
---

<center>
引言：Collection与Map
</center>

<!--more-->

---

# 集合
Java中实现了部分的数据结构（Data Structure）的内容，并把他们封装成了类

## 集合是什么
集合是一种可以存储多个数据的容器（此集合非彼集合）

优点：
- 集合的长度可以随意变化
- 集合存储的都是对象，而且对象的类型可以不同

## 将集合接口与实现分离

例如：队列
> 队列：先进先出(FIFO)，头的一端可以删除元素，尾的一端可以添加元素，并且可以查找队列中元素的个数

队列Queue接口可能是这么实现的
```java
public interface Queue<E>{
    void add(E element);
    E remove();
    int size();
}
//这里只是做个示例
```
队列有两种实现：
1. 循环数组
```java
public class CircularArrayQueue<E> implements Queue<E>{
    private int head;
    private int tail;
    CircularArrayQueue(int capcity){...}
    public void add(E e){...}
    public E remove(){...}
    public int size(){...}
    private E[] elements;
}
//这里只是做个示例
```
2. 链表
```java
public class LinkedListQueue<E> implements Queue<E> {
    private Link head;
    private Link tail;
    LinkedListQueue(){...}
    public void add(E element){...} 
    public E remove(){...}
    public int size(){...}
}
//这里只是做个示例
```

当在程序中使用队列时，一旦构建了集合就不需要知道究竟使用了哪种实现，**所以，只有在构建集合对象的时候，使用具体的类才有意义**
，例如：

```java
Queue<MyClass> myClass = new CircularArrayQueue<>(100);
myClass.add(new MyClass("Jack"));
/*
    一旦我们改变了想法，我们只需要改变调用构造方法的那一块，如下
*/
Queue<MyClass> myClass = new LinkedListQueue<>(100);
myClass.add(new MyClass("Jack"));
```

## 集合关系图
在Java中，集合类的基本接口是Collection接口
![Collection接口](http://img.yesmylord.cn//Collection01.png)

- Collection接口

    定义了所有单列集合中共性方法，
    所有单列集合都可以使用的方法，没有带索引的方法
    
    - List接口
        1. 有序的集合（存储和取出元素的顺序相同）
        2. 允许存储重复的元素
        3. 有索引，可以使用普通的for循环遍历
    - Set接口
        1. 不允许存储重复元素
        2. 没有索引，不能使用普通的for循环遍历
        3. 无序
    - LinkedHashSet
      
        例外，是有序的集合

学习集合的方法：
1. 学习顶层：学习顶层接口/抽象类的共性方法，所有的子类都可以使用
2. 使用底层：顶层无法创建对象使用，需要使用底层来创建对象

# Collection接口

```java
public interface Collection<E>{
    boolean add(E element);
    Iterator<E> iterator();
    ...
}
```
`iterator()`实现了Iterator接口，首先了解一下迭代器

## 迭代器

Iterator迭代器接口有四个方法：
    1.  `next()`：逐个访问集合中的每个元素，但是如果到带了集合的末尾会抛出一个NoSuchElementException异常
            Java的迭代器可以看做是位于两个元素之间的，当调用`next()`的方法时，迭代器就越过下一个元素，并返回刚刚越过的元素的引用
    2.  `hasNext()`：判断是否还有下一个元素
    3.  `remove()`：删除一个刚刚越过的元素
    4.  `forEachRemaining`：流式编程

```java
public interface Iterator<E>{
    E next();
    boolean hasNext();
    void remove();
    default void forEachRemaining(Consumer<? super E>action)
}
```
![迭代器](http://img.yesmylord.cn//iterator_01.png)

### 迭代器的使用
迭代器的使用
```java
Collection<String> coll = ...;
Iterator<String> iter = coll.iterator();
//调用集合的iterator()方法
while(iter.hasNext()){
    String e = iter.next();
    //do something
}
```
`for each`也可以简单的表示同样的循环操作
```java
for(String e : c){
    //do something
}
```
JavaSE8后，可以调用Iterator最后一个方法`foreachRemaining`方法，这个方法会提供一个Lambda表达式
```java
iter.forEachRemaining(element -> doSomething)
```

## Collection的常用方法
常用方法
```java
int size() 
//返回当前存储在集合中的元素个数

boolean isEmpty()
//如果此 collection 不包含元素，则返回 true。

boolean contains(Object o)
//判断当前集合是否包含给定的对象，包含返回true，不包含返回false

boolean containsAll(Collection<?> other)
//如果这个集合包含other中所有的元素，返回true

boolean add(E e)
/*如果此 collection 由于调用而发生更改，则返回 true。
如果此 collection 不允许有重复元素，并且已经包含了指定的元素，则返回 false。
*/
boolean addAll(Collection<? extends E> other)
//将other中所有的元素添加到这个集合

boolean remove(Object o)
//从此 collection 中移除指定元素的单个实例,不存在此元素返回false

boolean removeAll(Collection<?> other)
//从这个集合中删除other集合中存在的所有元素

boolean retainAll(Collection<?> other) 
//从这个集合中删除所有与 other 集合中的元素不同的元素。如果由于这个调用改变了集合， 返回 true

void clear()
//移除此 collection 中的所有元素,集合是还存在的

Object[] toArray()
//返回包含此 collection 中所有元素的数组。
```
## 代码示例
```java
Collection<String> coll = new ArrayList<>();//接口的构造
//add方法
boolean b1 = coll.add("张三");
//把给定的对象添加到集合当中，返回值一般都是true，可以不用接受
System.out.println(coll);//[张三] 不是地址,说明重写了toString方法
System.out.println(b1);//true

Collection<String> coll = new ArrayList<>();//接口的构造

coll.remove("张三");//删除方法

coll.contains("张三");//是否含有方法

coll.isEmpty(); //是否为空方法

coll.size();//获取长度

coll.toArray();//把集合变为数组

coll.clear();//清空元素方法
```

# Iterator

迭代器： 一种通用的取出集合元素的方法

`Iterator`是一个接口，我们无法直接使用

需要使用实现类对象，获取它的实现类比较特殊

迭代： 在取元素之前判断集合中有没有该元素，如果有就取出这个元素，继续判断，直到取出集合中所有需要的元素

## 实现类

获取实现类的方法

Collection接口有一个`iterator()`方法，此方法直接返回实现类对象

## 常用方法

```java
boolean hasNext()
//如果仍有元素可以迭代，则返回 true。（换句话说，如果 next 返回了元素而不是抛出异常，则返回 true）。 

E next()
//返回迭代的下一个元素。 
```

## 常用方法

```java
Collection<String> coll = new ArrayList<>();
coll.add("姚明");
coll.add("科比");
coll.add("麦迪");
coll.add("詹姆斯");
coll.add("艾弗森");
//iterator<>也有泛型
Iterator<String> it = coll.iterator();//Iterator有泛型<>
//通过Collection的Iterator()方法获得实现类
while (it.hasNext())//判断是否含有下个元素
    System.out.println(it.next());//循环获得元素
```

# foreach

JDK1.5之后的一个高级for循环，专门用来遍历数组和集合的，它的内部其实是个iterator迭代器

**所以在遍历的过程中，不能对集合中的元素进行增删操作**

所有的单列集合都可以使用增强for

```java
for(元素的数据类型 变量 : Collection集合/数组){
    //操作代码，不能对集合中的元素进行增删操作
}
```

```java
Collection<String> coll = new ArrayList<>();
coll.add("姚明");
coll.add("科比");
coll.add("麦迪");
coll.add("詹姆斯");
coll.add("艾弗森");

for (String s : coll) {
    System.out.println(s);
}
```

# 泛型

泛型是一种未知的数据类型，当我们不知道使用声明数据类型的时候，我们可以使用泛型

可以看成一个变量，可以用来接收数据类型

`ArrayList`集合在定义的时候，不知道集合中都会存储声明元素的数据，所以类型使用泛型

```java
public class ArrayList<E>{
    public boolean add(E e){}
    public E get(int index){}
}
```

泛型数据类型的确定时间在创建对象的那一刻

## 优缺点

- 优点

  1. 避免了类型转换的麻烦，存储的是什么类型，取出的就是什么类型
  2. 把运行期异常(代码运行之后会抛出的异常)，提升到了编译器(写代码的时候会报错)

- 弊端

  泛型是什么类型只能存储声明类型

如果不加泛型，默认`Object`类，可以存储任何对象，**但不安全，但容易引发异常**

（例如：当存入`Integer`对象和`String`对象，遍历时想调用`toString`方法，而这样就必须向下转型，在运行中也可能会报错）

## 泛型类

### 泛型类

```java
public class father<E> {
    private E name;

    public void setName(E name) {
        this.name = name;
    }

    public E getName() {
        return name;
    }
}
```

调用

```java
father f = new father();
f.setName("字符串");
f.setName(123);
f.setName(true);
//不管是什么类型都可以使用
```

### 泛型方法

```java
public <E> void method (E e){
    System.out.println(e);
}
public static <S> void methodStatic(S s){
    System.out.println(s);
}
```

### 泛型接口

```java
public interface api<E> {
    public void out(E e);
}
```

接口实现类

```java
public class apiImpl <E> implements api<E>{
    @Override
    public void  out(E e){
        System.out.println(e);
    }
}
```

### 高级用法

泛型通配符`？`

不知道使用什么类型时可以使用

但是此时只能接受数据，而不能存储数据

```java
public static void print(ArrayList<?> list){
    Iterator<?> it = list.iterator();
    while (it.hasNext()){
        System.out.println(it.next());
    }
}
```

高级用法：

- 泛型的上限限定：`? extends E` 代表使用的泛型只能是E类型的子类/本身

- 泛型的下限限定：`?  super  E`代表使用的泛型只能是E类型的父类/本身

# 数据结构简述

## 栈(stack)

先进后出

只有一个口，最先进去的，最后才能出去

类似于手枪弹夹

## 队列(quene)

先进先出

有一个入口一个出口，先进去的先出去


## 数组(array)

查询快，增删慢

查询快：

数组的地址是连续的，我们通过数组的首地址可以找到数组，通过数组的索引值可以快速的找到某一个元素

增删慢：

数组的长度是固定的，我们增删一个元素，必须创建一个新的数组，把原数组的数据复制过来，而原数组会被垃圾回收


## 链表(linked list)

查询慢，增删快

查询慢: 链表的地址不是连续的，每次查询元素必须从头开始

增删快：链表结构增删一个元素，对整体没有影响

链表的每一个元素称之为一个**节点**,

一个节点包括三个部分(自己的地址+数据+下一个节点的地址)

### 两种列表

- 单向列表

  链表中只有一条链子，不能保证元素的顺序（存储元素和去除元素的顺序有可能不一致）

- 双向列表

  链表中有两条链子，一条专门记录元素的顺序，一条是一个有序的集合


## 红黑树

特点：趋于平衡的树，查询的速度非常快

约束：

1. 节点可以是红色也可以是黑色
2. 根节点是黑色
3. 叶子结点（空节点）是黑色
4. 每个红色的节点的子节点都是黑色的

5. 任何一个节点到其每一个叶子结点的路径上黑色节点相同

# List集合

## 特点

- 有序：存储与取出的顺序是一致的
- 有索引：包含常用的索引方法
- 允许重复

## 常用方法

```java
public void add(int index,E element)
//将指定的元素添加到指定的位置上
public E remove (int index)
//移除表中指定位置的元素，返回被移除的元素
public E set(int index,E element)
//用指定元素替换几何中指定位置的元素，返回更新前的元素
```

示例

```java
List<String>list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");
list.add("d");

list.add(3,"啦啦啦");// 添加方法
System.out.println(list);
list.remove("啦啦啦");// 移除方法
System.out.println(list);
list.set(2,"A");// 替换方法
System.out.println(list);
```

遍历方法

```java
//普通for循环
for(int i = 0;i<list.size();i++){
    String s = list.get(i);
    System.out.println(s);
}
//foreach遍历
for (String s : list) {
    System.out.println(s);
}
//迭代器遍历
Iterator<String> it = list.iterator();
while (it.hasNext()){
    String s = it.next();
    System.out.println(s);
}
```

## 常见报错

```java
IndexOutofBoundsException//索引越界异常，集合会报
ArrayIndexOutofBoundsException//数组索引越界异常
StringIndexOutOfBoundsException//字符串索引越界异常
```

# ArrayList<E>

List 接口的大小可变数组的实现。

实现了所有可选列表操作，**并允许包括 `null`在内的所有元素。**

除了实现 List接口外，此类还提供一些方法来操作内部用来存储列表的数组的大小

尖括号内的E叫做**泛型**，装在该集合当中的所有元素都必须是统一的类型

泛型**只能是引用类型，不能是基本类型**

包路径：

```
java.util
```

构造方法：

```java
public ArrayList(Collection<? extends E> c)
//构造一个包含指定 collection 的元素的列表，
//这些元素是按照该 collection的迭代器返回它们的顺序排列的。 
```

常用方法

```java
public E set(int index,E element);  //输入索引值和相同的泛型内容替换原有的内容

public E get(int index)             //返回索引值对应的内容

public boolean add(E e)             //把指定元素添加到集合的尾部

public void add(int index,E element)//添加指定的元素到索引值处，后面的内容索引值加一

public E remove(int index)          //移除集合上指定位置上的元素

public boolean remove(Object o)     
//移除此列表中首次出现的指定元素（如果存在）。如果列表不包含此元素，则列表不做改动

public void clear()                 //移除集合中的所有元素

public boolean addAll(Collection<? extends E> c)
//将另一个集合并入此集合

public boolean addAll(int index,Collection<? extends E> c)
//从指定位置并入另一个集合

public void ensureCapacity(int minCapacity)
//增加此ArrayList的容量,以确保至少能容纳参数所指定的元素数

public boolean isEmpty()            //判断集合是否为空

public boolean contains(Object o)   //判断集合是否含有指定的元素

public int indexOf(Object o)
//返回此列表中首次出现的指定元素的索引，或如果此列表不包含元素，则返回 -1。

public int lastIndexOf(Object o)
//返回此列表中最后一次出现的指定元素的索引，或如果此列表不包含索引，则返回 -1。

```

示例代码：

```java
ArrayList<String> list = new ArrayList<>();
// 创建了一个ArrayList的集合，
// 泛型为String类型，代表这个集合内只能含有String类型的数据
System.out.println(list);//返回[]
//直接打印空集合，返回的并不是地址，而是一个空数组

list.add("小明");//添加方法
list.add(1,"小白");//指定添加

String str = list.get(0);//得值方法

list.set(0, "小红");//替换方法

list.remove(0);//移除方法
list.remove("小白");//移除第一个指定值

list.ensureCapacity(9);//最小兼容方法

list.isEmpty();//是否空方法

list.contains("小白");//是否包含方法

list.indexOf("小白");//求首次出现索引方法

list.lastIndexOf("小白");//求末次出现索引方法


ArrayList<String> list2 = new ArrayList<>();
list2.addAll(list);//合并另一个ArrayList

list.size();//返回长度

for (int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}//输入list.fori+TAB快速得到遍历集合的循环
```

泛型只能导入引用类型，那怎么存入基本类型呢？

使用基本类型的对应的包装类

从JDK1.5开始，支持自动装箱拆箱

```java
ArrayList<Integer> list1 = new ArrayList<>();
list1.add(100);//完成了自动装箱
list1.add(new Integer(100));//手动装箱，同上
```

# LinkedList

底层是一个双向列表，

查询慢，增删快，多线程

也有`list`的三个特点(有序，可索引，可重复)

## 包路径

```
java.util
```

## 常用方法

添加方法

```java
public void addFirst(E e)
//将指定元素插入此列表的开头

public void addLast(E e)
//将指定元素添加到此列表的结尾

public void push(E e)
//将元素推入此列表所表示的堆栈。换句话说，将该元素插入此列表的开头,同addFirst
```

获取方法

```java
public E getFirst()
//返回此列表的第一个元素

public E getLast()
返回此列表的最后一个元素
```

移除方法

```java
public E removeFirst()
//移除并返回此列表的第一个元素。 

public E removeLast()
//移除并返回此列表的最后一个元素。 

public E pop()
//从此列表所表示的堆栈处弹出一个元素。换句话说，移除并返回此列表的第一个元素。 
//此方法等效于 removeFirst()。 

```

## 示例代码

```java
LinkedList<String> linked = new LinkedList<>();
linked.add("a");
linked.add("b");
linked.add("c");
//向集合添加一些内容

//添加内容
linked.addFirst("www");
linked.push("aaa");//与addFirst相同
linked.addLast("com");
System.out.println(linked);
//获取内容
System.out.println(linked.getFirst());
System.out.println(linked.getLast());
//移除方法
linked.removeFirst();
linked.removeLast();
System.out.println(linked);
```

# Vector

是一个同步的集合（单线程，速度慢）

了解即可，最早期的一个集合

# HashSet

特点:

- 继承set接口
  1. 没有重复元素
  2. 没有索引，也不能使用普通的for循环
- HashSet特点
  1. 不同步（多线程）
  2. 无序的集合
  3. 没有索引
  4. 底层是一个哈希表结构（查询速度非常快）

## 包路径

```java
java.util
```

## 构造函数

```java
public HashSet()
//构造一个新的空 set，其底层 HashMap 实例的默认初始容量是 16，加载因子是 0.75。
```

```java
Set<Integer> set = new HashSet<>();//构造一个HashSet集合
set.add(1);
set.add(12);
set.add(12);//填入两个12
set.add(3);
System.out.println(set);//[1, 3, 12]只有一个12
```

## 存储自定义类型的元素

对于自定义元素

```java
Set<father> set = new HashSet<>();//构造一个HashSet集合
father f1 = new father("一号小弟",15);
father f2 = new father("二号小弟",15);
father f3 = new father("一号小弟",15);
//有三个变量，其实只有两个人，对于f1和f3是同一个人
set.add(f1);
set.add(f2);
set.add(f3);
System.out.println(set);
//但是发现，三个人都存进去了，这显然不是我们想要的
```

我们必须重写`hashCode`和`equals`方法，来保证不重复

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    father father = (father) o;
    return age == father.age &&
            Objects.equals(name, father.name);
}
@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```

再次运行，发现重复的人就被去掉了

# 哈希值

哈希值是一个**十进制的整数**，由系统随机给出

就是对象的地址值，是一个逻辑地址，是模拟出来的地址，不是数据实际存储的物理地址

在Obejct类中有一个方法可以返回对象的哈希值

```java
father f1 = new father();
int h1 = f1.hashCode();
System.out.println(h1);//961
```

```java
String s1 = "重地";
String s2 = "通话";
System.out.println(s1.hashCode());//1179395
System.out.println(s2.hashCode());//1179395
//通话和重地有一个巧合，哈希值相同
```

## 哈希表

HashSet集合存储数据的结构（哈希表）

把元素进行了分组，相同哈希值的元素分为一组，每一组都是一个链表，这些元素的哈希值相同

如果链表超过了8位，那么就会把链表转化为红黑树，提高查询速度


- jdk1.8之前：哈希表 = 数组+链表

- jdk1.8之后：

  哈希表 = 数组 +链表

  哈希表 = 数组 + 红黑树（提高查询的速度）

  哈希表的特点：速度快 

## set不重复原理

add方法会计算字符串的哈希值，得到哈希值会去寻找是否存在此哈希值。

如果没有就会存储到集合当中

如果有(哈希冲突)就会调用`equals`方法对这两个相同的字符进行比较，比较如果不同才会存储这个元素，挂在同一个位置

# LinkedHashSet

继承了HashSet集合，底层是一个哈希表(数组+链表/红黑树)+链表

多了一条链表（记录元素的顺序），可以保证元素有序

## 包路径

```
java.util
```

**HashSet无序,LinkedHashSet有序**

# 可变参数

JDK1.5之后，如果我们需要接受多个参数，并且多个参数的类型一致，我们就可以简化成如下的格式：

```java
public static void sum(int... a){//这样接收
}
```

```java
//调用
sum(1,3,5,545,8,4);//可以接收任意个相同类型的参数
```

注意事项：
一个方法的参数列表，只能有一个 可变参数
如果方法的参数有多个，那么可变参数必须写在最右边

小实例：计算任意项的和

```java
public static void main(String[] args) throws ParseException {
    sum(1,3,5,545,8,4);
}
public static void sum(int... a){
    int sum = 0;
    for (int i : a) {
        sum+=i;
    }
    System.out.println(sum);//566
}
```

# Collections

此类完全由在 `collection` 上进行操作或返回 `collection` 的静态方法组成。

它包含在 `collection` 上操作的多态算法，即“包装器”，包装器返回由指定 `collection` 支持的新 `collection`，以及少数其他内容。

## 常用方法

```java
Collections.addAll()//参数第一个是一个集合，后面是任意个元素

Collections.shuffle//参数：一个集合,打乱集合

//1.
Collections.sort()
//参数只能是一个List集合，不能是set集合！
//将集合的元素按默认排序，默认是升序，从小到大
//自定义类型的sort必须重写Comparable接口中CompareTo这个方法，否则默认报错

//2.
Collections.sort()
//两个参数，第一个是集合，第二个是方法new 
//区别: 
//Comparator相当于找一个第三方的裁判，比较两个
//Comparable自己和参数比较，自己需要实现Comparable接口，重写比较的规则compareTO方法
```

## 示例代码

```java
ArrayList<String> list = new ArrayList<>();
Collections.addAll(list,"a","v","w","6","g");
Collections.shuffle(list);//打乱
System.out.println(list);//[g, 6, v, a, w]
Collections.sort(list);//默认升序排序
System.out.println(list);//[6, a, g, v, w]
```

自定义元素`sort`,需要在自定义类中重写`compareTo`方法

引入`Comparable<>`接口

```java
public class father implements Comparable<father>{
    //这里要引入一个接口Comparable,注意写入泛型
    @Override
    public int compareTo(father f){
        return this.getAge() - f.getAge();
        //return 0;默认返回一个零，认为是相同的
    }
...}
```

```java
father f1 = new father("一号",10);
father f2 = new father("二号",18);
father f3 = new father("三号",40);
ArrayList<father> list = new ArrayList<>();
Collections.addAll(list,f1,f2,f3);
Collections.sort(list);
```

`Comparable<>`接口的排序规则:

this - 参数 -> 升序

参数 - this -> 降序


# Map

Map<K,V>

- K键：此映射所维护的键的类型
- V键：映射值的类型

Map是一个双列集合，与Collection不同

将键映射到值的对象。一个映射不能包含重复的键；每个键最多只能映射到一个值。 

特点：

1. Map集合是一个双列集合，包含一个键一个值
2. 键和值的数据类型不相关
3. key不可以重复，但是value可以重复
4. key和value一一对应

## 包路径

```
java.util
```

## 实现类

- HashMap(底层是一个哈希表)
  1. 查询速度快
  2. 底层是哈希表
  3. 无序集合
- LinkedHashMap
  1. 底层是一个哈希表+链表
  2. 有序集合

## 常用方法

```java
V put(K key, V value)
//将指定的值与此映射中的指定键关联（可选操作）
//如果此映射以前包含一个该键的映射关系，则用指定值替换旧值

//存储键值对的时候
//key不重复，返回值v是null
//key重复，会使用新的v替换map中重复的v，返回被替换的value值

V remove(Object key)
//key存在，返回被删除的值
//key不存在，返回null

V get(Object key)
//返回指定键所映射的值
//如果此映射不包含该键的映射关系，则返回 null

boolean containsKey(Object key)
//如果此映射包含指定键的映射关系，则返回 true。
//更确切地讲，当且仅当此映射包含针对满足 (key==null ? k==null : key.equals(k)) 的键 k 的映射关系时，返回 true。

Set<K> keySet()
//把Map集合中所有的key取出，存入到Set集合当中

```

## 示例代码

```java
Map<String,String> map = new HashMap<>();
map.put("李晨","范围");//不重复返回null
map.put("李晨","范冰冰");//重复返回被替换去掉的值
map.put("杨过","小龙女");
System.out.println(map);//{杨过=小龙女, 李晨=范冰冰}
map.remove("李晨");
System.out.println(map);//{杨过=小龙女}
System.out.println(map.get("杨过"));//小龙女
System.out.println(map.containsKey("李晨"));//false
Set<String> set = map.keySet();//转化为set集合
System.out.println(set);//[杨过]
```

## Map遍历

- 法一：利用keySet方法

```java
Map<String,String> map = new HashMap<>();
map.put("李晨","范围");
map.put("阿q","abc");
map.put("杨过","小龙女");
Set<String> set = map.keySet();
for (String s : set) {
    System.out.println(s + "," + map.get(s));
}
```

- 法二：使用entrySet方法

```java
Map<String,String> map = new HashMap<>();
map.put("李晨","范围");
map.put("阿q","abc");
map.put("杨过","小龙女");
//通过Map名来找到Entry对象
for(Map.Entry<String, String> s : map.entrySet()){
    System.out.println(s.getKey()+","+s.getValue());
}
```

存储自定义类型，记得要重写hashCode方法，去重

## Map.Entry

Map接口中有一个内部接口Entry

作用：当Map集合一旦创建，那么就会在Map集合中创建一个Entry对象，用来记录**键与值**的关系

他有两个方法:

- getKey
- getValue
  用来获取V和K

# JDK9优化

jdk9中，list，set，map都存入了一个静态of方法，
**只适用于这三个接口，不适用于这三个接口的实现类**

1. 可以用来  一次性 添加  多个元素
2. of方法的返回值是一个不能改变的集合，该集合不能再用add，put存入元素
3. Set和Map接口在调用of方法时不能有重复的元素
   所以要在数量确定的时候再用

```java
List<String> list = List.of("a","b","c");
```

