---
title: 速转Golang
date: 2023-10-25 00:59:43
tags: 
- Golang
categories: 
- Golang

---

 <center>
引言：字节用的Go有点多呢，速转一下
</center>

<!--more-->

# 速转Golang

## Hello world

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello world!")
}
```

## Printf

打印类似C语言：

- `%d`：10进制
- `%b`：2进制
- `%o`：8进制
- `%x`：16进制（如果是大写`%X`，表示输出的字母为大写，比如15：输出`F`，否则输出`f`）

```go
fmt.Printf("%d \n", 42)
fmt.Printf("%d \t %b \t %o \t %x \n", 42, 42, 42, 42)
fmt.Printf("%#d \t %#b \t %#o \t %#x \n", 42, 42, 42, 42)
// 加#表示带前缀输出
```

输出为：

```go
42
42       101010          52      2a   
42       0b101010        052     0x2a
```

[有关printf的占位符看此篇](https://studygolang.com/articles/2644)

## 包

在go中，**每一个文件就是一个包**，一般一个Go项目的结果是：

- `bin`：存放可执行文件
- `pkg`：存放其他包
- `src`：存放源代码
- go.mod文件

```go
package stringutil

// 开头大写的变量或是函数，代表这个变量会被export
var MyName = "Todd"
func Reverse(s string) string {
	return reverseTwo(s)
}
```

```go
package main

import (
    // import导入包
	"GolangTraining/src/02_package/stringutil"
	"fmt"
)

func main() {
	fmt.Println(stringutil.Reverse("!oG ,olleH"))
	fmt.Println(stringutil.MyName)
}
```

## 变量

### 变量的声明

变量有三种声明方式：

- `:=`声明并且赋值
- `var valueName valueType`：声明变量及其类型，但是没有赋值（有默认值）
- `var valueName =  value`：直接给值，会自动推断类型

第一种方式：`:=`

```go
a := 10 // 推断为int
b := "golang" // 推断为string
c := 4.17 // 推断为float64
d := true // 推断为bool
e := "Hello" // 推断为string
f := `Do you like my hat?` // 反引号作用与双引号一样
g := 'M' // 推断为int32 单引号表示为字符
```

第二种方式：`var valueName valueType`

```go
var a int // 默认为0
var b string // 默认为空字符串""
var c float64 // 默认为0
var d bool // 默认为false
```

第三种方式：

```go
var message string
message = "Hello World."
```

注意：

- 打印变量时：可以直接打印，或是使用替代符`%v`代表值，`%T`表示类型

- 变量如果不给值会有默认值

声明多种变量：

```go
var a, b, c = 1, false, 3 // 自动推断
var a, b, c int = 1, 2, 3 // 这种情况必须全为int
```

### 作用域

- 声明在函数外，作用域属于包`package scope`；

- 声明函数内，作用域属于函数`function scope`
- 声明在代码块内，作用域就属于块`block scope`

1、包作用域：

```go
package vis

import "fmt"

// 这两个变量都属于vis包内可见的，其他包想使用这两个变量必须导入vis包
var MyName = "Todd" // 开头大写，代表导出，其他包可以直接使用
var yourName = "Future Rock Star Programmer" // 小写表示私有

func PrintVar() {
	fmt.Println(MyName)
	fmt.Println(yourName) 
}
```

```go
// 使用main打印两种变量
func main() {
	fmt.Println(vis.MyName) // 输出 Todd
	// fmt.Println(vis.yourName) 这个就不行了
	vis.PrintVar()
    // 输出
    // Todd
    // Future Rock Star Programmer
}
```

2、函数域：

有**函数**和**函数表达式**两种：

```go
var x = 0 // 包内可见

func increment() int {
	x++
	return x
}

func main() {
	fmt.Println(increment())
	fmt.Println(increment())
}
```

```go
func main() {
	x := 0
	increment := func() int { // increment是一个匿名函数
		x++ // 函数使用了外部变量num，这是一个闭包
		return x
	}
	fmt.Println(increment())
	fmt.Println(increment())
}
```

**关于什么是闭包？见下一节**

3、块域：

```go
func main() {
	x := 42
	fmt.Println(x)
	{
		fmt.Println(x)
		y := "The credit belongs with the one who is in the ring."
		fmt.Println(y)
	}
	// fmt.Println(y) 块外部无法打印y
}
```

> 关于**域出现的特性的本质**：
>
> 大括号定义了一个新的堆栈框架，因此定义了一个新的范围级别。
>
> 变量名称可以在新的大括号内重复使用。
>
> **当代码到达右大括号时，堆栈中的一小部分将被弹出**

### 闭包

闭包是一个特殊的匿名函数, 它是匿名函数和相关引用环境组成的一个整体

**闭包允许函数在其定义的作用域外部访问变量**，即使在函数执行完毕后，这些变量仍然可以被引用和操作。

下面是一个go的demo：

```java
func wrapper() func() int { // 这里函数的返回值为func()，表示返回一个函数
	x := 0
	return func() int {
		x++
		return x
	}
}

func main() {
	increment := wrapper()
	fmt.Println(increment()) // 1
	fmt.Println(increment()) // 2
}
```

在Java中，闭包一般由内部类完成（内部类操作外部类的数据）



### 声明顺序

函数内的变量必须要声明在使用之前

```go
package main

import "fmt"

func main() {
	// fmt.Println(x)
	fmt.Println(y)
	// x := 42 局部变量的声明必须优先于使用
}

var y = 42
```

### 变量覆盖

下面是一个，变量覆盖的demo：

```go
func max(x int) int {
   return 42 + x
}

func main() {
   max := max(7) // max和函数max()同名
   fmt.Println(max) // max现在是一个值，而不再是一个方法
}
```

这种是非常错误的习惯

### 同一个包下的不同的代码

同一个包下的代码可以写在两个文件内：

```go
package main

var x = 7
```

```go
package main

import "fmt"

func main() {
   fmt.Println(x)
}
```

但是执行时需要特别注意：

```go
go run *.go
```

或者

```go
go build
./xxx
```

### 空白标识符

go中，如果一个变量没有被使用，是会报错的

```go
func main() {
	a := "stored in a"
	b := "stored in b"
	fmt.Println("a - ", a)
	// b is not being used - invalid code
	// 如果一个变量没有使用到，编译器会报错：b declared but not used
}
```

因此如果函数返回了一个值，但是我们没有用到，就可以使用`_`来代替：

下面是一个http的demo：

```go
func main() {
	res, err := http.Get("http://www.geekwiseacademy.com/")
	if err != nil {
		log.Fatal(err)
	}
	page, err := ioutil.ReadAll(res.Body)
	res.Body.Close()
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%s", page)
}
```

上面的code使用了err，但是如果我们现在不想校验err，那么就可以像下面一样

```java
func main() {
	res, _ := http.Get("http://www.geekwiseacademy.com/")
	page, _ := ioutil.ReadAll(res.Body)
	res.Body.Close()
	fmt.Printf("%s", page)
}
```

## 常量

### 声明

与变量一样，有作用域：

```go
const p = "death & taxes"

func main() {
	const q = 42
}
```

声明多个常量可以使用括号：

```go
const (
	pi       = 3.14
	language = "Go"
)
```

### iota

（类似于Java的枚举类型）

有时候，我们只希望做一些区分（仅仅使用数字来区分，而不是字符串，没有实际的意义）就可以使用iota

```go
// 方式一
const (
	a = iota // 0
	b = iota // 1
	c = iota // 2
)
// 方式二：第一个之后的省略
const (
	a = iota // 0
	b        // 1
	c        // 2
)
// 方式三：可以用_省略第一个0，之后的可以给新的值
const (
	_  = iota             // 0
	KB = 1 << (iota * 10) // 1 << (1 * 10)
	MB = 1 << (iota * 10) // 1 << (2 * 10)
	GB = 1 << (iota * 10) // 1 << (3 * 10)
	TB = 1 << (iota * 10) // 1 << (4 * 10)
)
```

## 指针

与C语言一样，go语言使用指针来操作内存地址：

```go
a := 43

fmt.Println(a) // 43
fmt.Println(&a) // 0xc000016098

var b = &a  // b的类型是一个int型的指针

fmt.Println(b) // 0xc000016098
fmt.Printf("%T \n", b) // *int
fmt.Println(*b) // 43
```

取地址符`&`和取内容符`*`（或者成为引用`reference`和逆向引用`dereference`）

指针最大的作用就是实现**地址传递**，而不是**值传递**

在Java中，如果一个方法传递了一个数组，默认是引用传递；在Go中除非你传递指针，否则是一个值传递。

（也可以这么理解：**go中的一切都是值传递，传递地址也是传递了一个值**）

下面是一个地址传递的demo：

```go
func zero(z *int) {
	*z = 0
}

func main() {
	x := 5
	zero(&x)
	fmt.Println(x) // x is 0
}
```

## for循环

在go中，只有for循环

```go
// 方式一：使用分号隔开，同Java
for i := 0; i <= 100; i++ {
    fmt.Println(i)
}
// 方式二：只有一个判断条件，这种很像while
i := 0
for i < 10 {
    fmt.Println(i)
    i++
}
// 死循环，直接for即可
i := 0
for {
    fmt.Println(i)
    i++
}
```

`break`与`continue`与其他语言一样

除此外，还有类似foreach的遍历方式：`for range`的遍历方式

```go
func main() {
	n := average(43, 56, 87, 12, 45, 57)
	fmt.Println(n)
}

func average(sf ...float64) float64 {
	fmt.Println(sf)
	fmt.Printf("%T \n", sf)
	var total float64
    // 第一个参数是序号，第二个参数是值
	for _, v := range sf {
		total += v
	}
	return total / float64(len(sf))
}
```

## switch语句

switch语句和Java之类的语言的区别在于，`case`不会顺序执行，会自动`break`；

额外提供了`fallthrough`关键字来提供向下执行的功能。

demo1：

```go
state := "Mhi"
switch state {
case "Daniel":
    fmt.Println("Wassup Daniel")
case "Medhi":
    fmt.Println("Wassup Medhi")
case "Jenny":
    fmt.Println("Wassup Jenny")
default:
    fmt.Println("Have you no friends?")
}
```

执行结果是输出：`Have you no friends?`

可见：

- `case`执行完成后会自动`break`，不会向下顺序执行（如果还想继续执行，那么使用`fallthrough`）
- 没有匹配到相应的语句就会进入`default`分支

demo2：

```go
switch state {
case "Tim":
	fmt.Println("Wassup Tim")
case "Jenny":
	fmt.Println("Wassup Jenny")
case "Marcus":
	fmt.Println("Wassup Marcus")
	fallthrough
case "Medhi":
	fmt.Println("Wassup Medhi")
	fallthrough
case "Julian":
	fmt.Println("Wassup Julian")
case "Sushant":
	fmt.Println("Wassup Sushant")
```

输出：

```
Wassup Marcus
Wassup Medhi
Wassup Julian
```

可见：

- `fallthrough`只会继续向下执行一个（一般能不使用这个关键字就不要使用，会使得逻辑混乱）

## if语句

基本的结构是`if elseif else`：

```go
if false {
    fmt.Println("first print statement")
} else if true {
    fmt.Println("second print statement")
} else {
    fmt.Println("third print statement")
}
```

除了基本的使用方法外，go中还支持`if`初始化一个属于if域的值

```go
b := true

if food := "Chocolate"; b { 
    // 这里初始化了一个food变量，只能在这个块域中使用
    fmt.Println(food)
}
// 这里不能使用food
```

## 函数

### 关于函数和方法的区别

函数一般指执行特定功能的一段代码；

方法特定于面向对象语言中，一个类执行的一些函数；

所以在Java中，函数等于方法；

但是在Go中，函数是函数，方法是类中的函数。

### 函数的结构

- main函数是程序的入口
- 普通的函数的结构是`func functionName(param1 typ1, ...) returnValue{}`

```go
func main() {
	fmt.Println(greet("Jane ", "Doe"))
}
// 参数可以传入多个，和声明多个变量一样；返回值在后面写
func greet(fname, lname string) string {
    //Sprint方法会拼接两个参数，当参数不是字符串时会添加空格
	return fmt.Sprint(fname, lname)
}
```

- 函数可以有多个返回值（Java只能有一个返回值）

```go
func greet(fname, lname string) (string, string) {
	return fmt.Sprint(fname, lname), fmt.Sprint(lname, fname)
}
```

一般使用多返回值的特性来返回结果值和错误情况，下面这个demo就是一个多返回值且使用了**命名返回**

```go
func ReturnId() (id int, err error) {
   id = 10
   return
}
```

- 传入多个参数使用`...`

下面这个例子计算平均值

```go
func main() {
	n := average(43, 56, 87, 12, 45, 57)
	fmt.Println(n)
}

func average(sf ...float64) float64 {
	fmt.Println(sf)
	fmt.Printf("%T \n", sf)
	var total float64
	for _, v := range sf {
		total += v
	}
	return total / float64(len(sf))
}
```

- `...`可以用来拆解数组

```go
func main() {
	data := []float64{43, 56, 87, 12, 45, 57}
	n := average(data...) // 将数组拆散输入
	fmt.Println(n)
}

func average(sf ...float64) float64 {
	total := 0.0
	for _, v := range sf {
		total += v
	}
	return total / float64(len(sf))
}
```

### 命名返回

可以给返回值指定一个变量名，返回时会自动返回

```go
func greet(fname string, lname string) (s string) {
	s = fmt.Sprint(fname, lname)
	return
}
```

**注意**：要避免使用命名返回

要注意命名覆盖的情况：

```go
func ReturnId() (id int, err error) {
   id = 10
   if id == 10 {
      err := fmt.Errorf("无效的 Id\n")
      // 这里会编译错误，因为你重新声明并赋值了err
      return
   }
   return
}
```

这种情况与下面的情况一样，go中不允许重复声明一个变量并且给其赋值

```go
func main() {
   id := 10
   id := 20

   fmt.Printf("Id: %d\n", id)
}
```

> 大括号定义了一个新的堆栈框架，因此定义了一个新的范围级别。变量名称可以在新的大括号内重复使用。当代码到达右大括号时，堆栈中的一小部分将被弹出。

因此一个大括号内，不要出现两个相同命名的变量

### 函数表达式

变量可以赋值为一个函数

```go
greeting := func() {
    fmt.Println("Hello world!")
}

greeting() // 变量名() 就可以调用函数
fmt.Printf("%T\n", greeting) // 输出 func()
```

甚至可以不给名字：

```go
func main() {
	func() {
		fmt.Println("I'm driving!")
	}()
}
```

- 函数的返回值可以是一个函数

```go
func makeGreeter() func() string {
	return func() string {
		return "Hello world!"
	}
}

func main() {
	greet := makeGreeter()
	fmt.Println(greet())
}
```

- 参数也可以传入函数

下面这个例子过滤掉数组中所有小于等于1的数字

```go
func filter(numbers []int, callback func(int) bool) []int {
	var xs []int
	for _, n := range numbers {
		if callback(n) {// callback返回一个bool
			xs = append(xs, n)
		}
	}
	return xs
}

func main() {
    // 传入一个数组和一个函数
	xs := filter([]int{1, 2, 3, 4}, func(n int) bool {
		return n > 1
	})
	fmt.Println(xs) // [2 3 4]
}
```

### defer关键字

defer关键字可以延迟一个函数的执行直到本方法执行结束：

```go
func hello() {
	fmt.Println("hello ")
}

func world() {
	fmt.Println("world")
}

func main() {
	fmt.Println(1)
	defer world() // 延迟world的执行
	fmt.Println(2)
	hello()
	fmt.Println(3)
}
```

输出的结果是：省去了换行

`1 2 hello 3 world`

注意：

- `defer`和`return`谁先谁后？`return`先，`defer`后

- 当多个defer出现的时候，是以栈的方式调用的，先进后出

![defer的执行顺序](http://img.yesmylord.cn//image-20231004185739256.png)





## 数组

### 声明方式

- 使用中括号，中间的数字表示数组的大小

```go
var x [58]int
fmt.Println(x)
// 会把数组的每一位都打印出来：[0 0 0 0 00 0 0 0 0 0 0 
// 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
// 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
fmt.Println(len(x)) // 58
fmt.Println(x[42]) // 0
x[42] = 777
fmt.Println(x[42]) // 777
fmt.Printf("%T", x)// [58]int
```

这种方式**限制了数组的大小**，如果需要可以扩容的数组，使用切片`slice`

获取数组的长度可以使用`len(数组)`

### 数组的遍历

数组的遍历，除了普通的for循环，可以使用foreach

```go
var x [256]int

fmt.Println(len(x))
fmt.Println(x[42])
// 普通方式
for i := 0; i < 256; i++ {
    x[i] = i
}
// foreach
for i, v := range x {
    fmt.Printf("%v - %T - %b\n", v, v, v)
    if i > 50 {
        break
    }
}
```

## 切片

### int切片

- 声明有两种方式：

1、当明确数组的内容时，可以不写大小，进行推断：

```go
mySlice := []int{1, 3, 5, 7, 9, 11}
// 或者使用var
var student []string
student := []int{}
// 注意使用:=时，要加{}
```

2、使用`make`进行分配()

```go
mySlice := make([]int, 0, 3)
```

- **注意**：`make`方式传入三个参数：

1、数组的类型；

2、数组的长度，可以使用`len()`获取；

3、数组的容量，可以使用`cap()`获取（**如果没有输入第三个参数，那么容量与长度相等**）

- 切片是可以进行自动扩容的，每次扩容为2倍原始的大小

- 添加元素：

1、切片添加元素可以使用下标进行赋值（这种方式**不能越过容量大小**！）

2、使用`append()`，按顺序进行添加，会自动进行扩容

```go
mySlice := make([]int, 0, 3)

fmt.Println(mySlice) // []
fmt.Println(len(mySlice)) // 0
fmt.Println(cap(mySlice)) // 3

for i := 0; i < 80; i++ {
    mySlice = append(mySlice, i)
    fmt.Println("Len:", len(mySlice), "Capacity:", cap(mySlice), "Value: ", mySlice[i])
}
/*
输出结果为：
Len: 1 Capacity: 3 Value:  0   
Len: 2 Capacity: 3 Value:  1   
Len: 3 Capacity: 3 Value:  2   
Len: 4 Capacity: 6 Value:  3 在这里可以看到容量翻倍了
....
*/
```

- 与python一样，切片可以使用`[:]`进行切片

```go
mySlice := []int{1, 3, 5, 7, 9, 11}
// 1、一个参数，表示按index访问对应的数字
fmt.Println(mySlice[1]) // 3
// 2、两个参数，左闭右开区间，返回的也是一个切片
fmt.Println(mySlice[2:3]) // [5] 左闭右开区间
fmt.Println(mySlice[2:5]) // [5 7 9]
fmt.Println(mySlice[0:6]) // [1 3 5 7 9 11]
// fmt.Println(mySlice[2:99]) //执行报错 panic: runtime error: slice bounds out of range [:99] with capacity 6
// fmt.Println(mySlice[-1:]) // 不允许是负数，编译报错
// 3、两个参数前后可以省略，表示从头开始或者结束
fmt.Println(mySlice[:3]) //[1 3 5]
fmt.Println(mySlice[3:]) //[7 9 11]
fmt.Println(mySlice[:]) // [1 3 5 7 9 11]
```

注意：与python不同的是，go的切片只支持两个参数，没有第三个参数步进

- 切片使用append添加切片，使用`...`

```go
mySlice := []int{1, 2, 3, 4, 5}
myOtherSlice := []int{6, 7, 8, 9}

mySlice = append(mySlice, myOtherSlice...)
// 如果你没有加...，代表着添加这个切片，而不是添加切片的元素！！
mySlice = append(mySlice, myOtherSlice) // 与上面的区分
```

- 切片也可以删除元素

```go
mySlice := []string{"Monday", "Tuesday"}
myOtherSlice := []string{"Wednesday", "Thursday", "Friday"}

mySlice = append(mySlice, myOtherSlice...)
fmt.Println(mySlice) //[Monday Tuesday Thursday Friday]ay Friday]

mySlice = append(mySlice[:2], mySlice[3:]...)
fmt.Println(mySlice) // 输出是空
```

### string切片

与int切片基本一致，除此外可以使用string访问对应元素

```go
"AaBb"[0] // 得到的是一个值 97
```

### 声明方式的区别

使用`var`和使用`:=`的区别是什么？

```go
var student []string
student := []int{}
```

1. `var student []string`：
   - 这是一个声明切片类型的变量 `student`，但没有分配内存空间给它。
   - 此时，`student` 的值是 `nil`，表示它不引用任何底层数组，也没有分配内存，因此不能进行读取或写入操作。
2. `student := []int{}`：
   - 这是使用切片字面量创建一个空切片，并分配了内存空间。
   - 这个切片引用了一个底层数组，虽然它是空的，但是可以安全地进行读取和写入操作。这个切片的长度为0。

## 切片与数组的区别

> 如何区分切片和数组？

声明时是否限制了大小？

- 如果`student := [3]int`，这就是一个数组
- 如果是`student := []int{}`或是使用了`make`，就是一个切片

> 切片与数组的区别是什么？

1. **内存方面**，数组是一个连续的内存地址；而切片不是
2. **底层数组**：数组是一个固定长度的内存块；而切片是一个地址，指向底层数组
3. **是否可变**：数组长度不可变；切片长度可变
4. **参数传递**：数组传递会copy一个数组（值传递）；切片直接传递地址（地址传递）

## map

声明方式与变量一样

```go
// 方式一：var声明 map[key]value，这种方式下还没分配内存 myGreeting = nil
var myGreeting map[string]string

// 方式二：:=
myGreeting := make(map[string]string)
myGreeting := map[string]string{
    "Tim":   "Good morning!",
    "Jenny": "Bonjour!",
}

// 方式三：make
var myGreeting = make(map[string]string)
myGreeting := map[string]string{}
```

- 获取长度使用`len`

```go
len
```

- 添加、删除、判断元素是否存在

```go
myGreeting := map[int]string{
    0: "Good morning!",
    1: "Bonjour!",
}
// 添加元素
myGreeting[2] = "Buenos dias!"
// 删除元素
delete(myGreeting, 1)
// 检查元素是否存在，使用第二个参数
if val, exists := myGreeting[2]; exists {
    fmt.Println("That value exists.")
    fmt.Println("val: ", val)
    fmt.Println("exists: ", exists)
}
```

- 遍历操作

```go
myGreeting := map[int]string{
    0: "Good morning!",
    1: "Bonjour!",
    2: "Buenos dias!",
    3: "Bongiorno!",
}

for key, val := range myGreeting {
    fmt.Println(key, " - ", val)
}
```

## Go结构体

### Go的面向对象

1. **封装**：类和结构体

   - 在Java中，面向对象编程的基本单位是类（class）
   - 在Go中，没有类的概念，而是使用结构体（struct）来定义自定义数据类型。Go通过在结构体上定义方法来实现面向对象编程，而不是使用类。

2. **继承**：Go中的继承更像是组合

   - Java支持继承，您可以创建一个子类，继承父类的属性和方法，并在子类中添加或覆盖它们。
   - Go不支持显式的继承。相反，Go鼓励使用组合来重用代码，通过将一个结构体嵌入到另一个结构体中来实现组合。这被称为嵌入式类型。

3. **多态**：接口

   - Java使用接口（interface）来定义抽象方法，类可以实现一个或多个接口。

   - Go也支持接口，但与Java不同，**Go的接口是隐式的**，即类型实现接口的方式不需要明确声明。**只要一个类型实现了接口所定义的方法，它就被认为实现了该接口。**

### 封装

使用`type Name struct`来声明一个结构体：

```go
// 结构体、结构体的值、方法，开头大写表示公有，小写表示私有
type person struct {
	name string
	age  int
}
// 方法前面使用()指定属于哪一个架构体
// 加*是地址传递，不加是值传递
func (this *person) SetName(newName string) {
	this.name = newName
}
func (this *person) GetName() string {
	return this.name
}
```

---

注意：当只有一个数据类型时，是一个**别名**

```go
type myInt int
```

此时`myInt`就是`int`的一个别名，没有别的作用

### 继承

- 在Go中使用组合的方式去实现“继承”

```go
type person struct {
	First string
	Last  string
	Age   int
}

type doubleZero struct {
	person // doubleZero继承了person
	LicenseToKill bool
}
```

- 子类可以重新定义属性或方法区覆盖父类的属性或方法

```go
type person struct {
	Name string
	Age  int
}

func (p person) Greeting() {
	fmt.Println("父类方法")
}

type doubleZero struct {
	person
	LicenseToKill bool
}

// 这里重写了父类的Greeting方法
func (dz doubleZero) Greeting() {
	fmt.Println("子类方法")
}

func main() {
    // 定义一个父类
	p1 := person{
		Name: "A",
		Age:  44,
	}
	// 定义一个子类
	p2 := doubleZero{
		person: person{
			Name: "B",
			Age:  23,
		},
		LicenseToKill: true,
	}
	p1.Greeting()        // 父类方法
	p2.Greeting()        // 子类方法
	p2.person.Greeting() // 父类方法
}
```

### 多态

Go使用接口来实现多态

```go
// 使用关键字interface来声明一个接口
type shape interface {
	area() float64
}

type square struct {
	side float64
}

func (this *square) area() float64 {
	return this.side * this.side
}

type circle struct {
	radius float64
}

func (this *circle) area() float64 {
	return math.Pi * math.Pow(this.radius, 2)
}

func main() {
	// 声明一个接口
	var shape1 shape
	// 接口的本质是一个指针，所以要用取地址符&
	shape1 = &square{10}
	fmt.Println(shape1.area())
	
	shape1 = &circle{4}
	fmt.Println(shape1.area())
}
```

> square和circle什么时候实现了shape接口呢？

Go中，只要一个类型实现了接口所定义的方法，它就被认为实现了该接口。

### 结构体标签

结构体还可以添加标签来对属性进行说明：

```go
type User struct {
	Nickname string `info:"登录ID" doc:"nickname不可以重复"`
	Password string `info:"登录密码" doc:"不可以设置过于简单"`
}
```

标签可以使用反射获取

不过主要的用途还是用来进行orm映射或是json序列化使用：

```go
type LoginUser struct {
    // 这里指定了json化的映射关系
	Nickname string `json:"nickname"`
	Password string `json:"passwd"`
}

func main() {
	user1 := LoginUser{"小李", "123"}
	res, err := json.Marshal(user1)
	if err != nil {
		fmt.Println("json marshal error", err)
		return
	}

	fmt.Println(string(res))
	// {"nickname":"小李","passwd":"123"}

	user2 := LoginUser{}
	err = json.Unmarshal(res, &user2)
	if err != nil {
		fmt.Println("json marshal error", err)
		return
	}
	fmt.Println(user2)
	// {小李 123}
}
```



## 空接口`interface{}`

### 类型断言

与Java中的Object类似，Go中的一切，甚至包括基础类型int、bool都是空接口的实现。

```go
interface{}
```

空接口提供了“**类型断言**”的机制来判断是哪一种类型：

```go
func main() {
    // 向上转型
	var name interface{} = "Sydney"
	// 注意：只有空接口才有这个方法
    // 两个返回值：值与是与不是
	str, ok := name.(string) // name是不是string类型？
	if ok {
		fmt.Printf("%T\n", str)
	} else {
		fmt.Printf("value is not a string\n")
	}
}
```

### 类型转换

空接口向下转型使用断言

```go
var val interface{} = 7
fmt.Printf("%T\n", val.(int))
```

普通类型转型直接转即可

```go
rem := 7.24
fmt.Printf("%T\n", int(rem))
```

这里列出一些常见的类型转换：

```go
// int与float相互转换
float64(x)
int(x)
// int转为string
string(x)
// byte数组与string
string([]byte{'h', 'e', 'l', 'l', 'o'})
[]byte("hello")
// go还有strconv包来帮助我们转换
strconv.Atoi(x) // A表示string、i表示int，表示string转为int
strconv.Itoa(x)	// 与上面相反
strconv.ParseBool("true") // 解析string为bool
strconv.ParseFloat("3.1415", 64) // 解析string为float
strconv.ParseInt("-42", 10, 64) // 解析string为int
strconv.ParseUint("42", 10, 64) // 解析string为int
```

## 反射

### pair结构

Go中的**变量包括（type, value）两部分**

- `type`：`type`有两种
  - `static type`：编码时的类型，静态类型，比如`int`、`string`
  - `concrete type`：`runtime`系统看见的类型
- `value`：变量的值

空接口`interface{}`有两个指针：一个指针指向值的类型【对应`concrete type`】，另外一个指针指向实际的值【对应`value`】

（只有运行时的类型才有反射一说，基本类型没有反射这个概念）

```go
func main() {
    // tty: pair<type: *os.File, value: "/dev/tty"的文件描述符>
	tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)
	if err != nil {
		fmt.Println("open file error", err)
		return
	}
    // r: pair<type: , value: >
	var r io.Reader
    // r: pair<type: *os.File, value: "/dev/tty"的文件描述符>
	r = tty

    // w: pair<type: ,value: >
	var w io.Writer
    
    // w: pair<type: *os.File, value: "/dev/tty"的文件描述符>
	w = r.(io.Writer)// 这里的断言为什么能成功呢？
	w.Write([]byte("HELLO THIS IS A TEST!!!\n"))
}
```

`w = r.(io.Writer)`这里的断言为什么能成功呢？

是因为`r`变量的`concrete type`（即`*os.File`）也实现了`w`的方法！因此可以成功。

**类型断言的本质**：变量的`concrete type`实现了要转换类型的方法

### 反射获取类型和方法

go提供了reflect包，主要有两个方法

```go
//ValueOf用来获取输入参数接口中的数据的值，如果接口为空则返回0
func ValueOf(i interface{}) Value {...}

//TypeOf用来动态获取输入参数接口中的值的类型，如果接口为空则返回nil
func TypeOf(i interface{}) Type {...}
```

对于基本的类型：

```go
x := 15
fmt.Println(reflect.TypeOf(x))  // int
fmt.Println(reflect.ValueOf(x)) // 15
```

对于复杂类型，比如说结构体：

结构体结构如下：

```go
type User struct {
	// 反射只能获取到公有的变量
	Nickname string `info:"登录ID" doc:"nickname不可以重复"`
	Password string `info:"登录密码" doc:"不可以设置过于简单"`
}

// 反射只能获取到公有的方法
func (this *User) Login() {
	fmt.Println(this.Nickname)
	fmt.Println(this.Password)
}
```

**反射的`NumField()`和`NumMethod()`只能获取公有的属性和方法**

```go
// 对于复杂类型：结构体
user := User{Nickname: "hynis", Password: "123"}

// 1、获取Type
userType := reflect.TypeOf(user)
fmt.Println(userType)

// 2、获取value
userValue := reflect.ValueOf(user)
fmt.Println(userValue)

// 3、通过type获取字段
for i := 0; i < userType.NumField(); i++ {
    field := userType.Field(i)
    value := userValue.Field(i).Interface()
    fmt.Printf("%s: %v=%v\n", field.Name, field.Type, value)
}

// 4、通过type获取方法
for i := 0; i < userType.NumMethod(); i++ {
    method := userType.Method(i)
    fmt.Printf("%s = %v", method.Name, method.Type)
}
// 5、获取标签的内容
for i := 0; i < userType.NumField(); i++ {
    tagInfo := userType.Field(i).Tag.Get("info")
    tagDoc := userType.Field(i).Tag.Get("doc")
    fmt.Println("info:", tagInfo, "doc", tagDoc)
}
```

















