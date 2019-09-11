---
title: Golang 回调函数和闭包
date: 2019-09-11 17:02
tags: 
- Golang
---

# 回调函数和闭包

## 高阶函数

> golang中的高阶函数有以下特性：

>* 函数可以作为另一个函数的参数（常用于回调函数）

>* 函数可以作为另一个函数的返回值（常用于闭包）

>* 函数可以被赋值给一个变量

### 1.函数作为参数

```go
package main

import "fmt"

func added(msg string, a func(a, b int) int) {
    fmt.Println(msg, ":", a(33, 44))
}

func main() {
    // 函数内部不能嵌套命名函数
    // 所以main()中只能定义匿名函数
    f := func(a, b int) int {
        return a + b
    }
    added("a+b", f)
}
```

* 此例中将在main函数中定一个匿名函数，将该函数赋值给变量f，作为参数传入added函数中，added函数会接收一个func(a, b int) int类型的函数作为参数。

* 这种将函数b作为参数传入函数a的使用方式可以提高函数a的灵活性，只要函数b符合函数a定义的参数类型就可以通过b完成很多不同的功能，因此常用于回调

### 2.函数作为返回值

```go
func squares() func() int {

var x int

// f := func() int {

// x++

// return x * x

// }

// return f

return func() int {

x++

return x * x

}

}



f2 := squares()

fmt.Println(f2()) //1

fmt.Println(f2()) //4

fmt.Println(f2()) //9

fmt.Println(f2()) //16
```



* 这里在函数体内部定义一个匿名函数，并且将该匿名函数作为另一个函数的返回值。main函数中将squares函数赋值给变量f2。

* 每一次打印都会调用一次squares函数，生成一个新的局部变量x，但是匿名函数会记录x的状态，每一次x都是在之前的基础上加1。

* 函数值就被称为闭包



## 回调函数

go语言中sort包内的SliceStable()就是一个回调函数，它用于给slice中的元素进行排序。该函数定义如下：

```go
func SliceStable(slice interface{}, less func(i, j int) bool)

s := []int{112, 22, 52, 32, 12}
	less := func(i, j int) bool {
	//按照降序进行排序，可以修改less的实现来进行不同方式的排序
		return s[i] > s[j]
	}
	sort.SliceStable(s, less)
```

* 该函数的第二个参数就是一个func(i, j int) bool类型的名为less的函数。

* less方法对已知的slice中的i，j索引对应的元素进行排序。

* 我们可以通过不同的less函数的实现对slice进行不同的排序方式。



## 闭包

函数a返回函数b的经典场景就是闭包。

> 闭包 -> 函数+作用域环境

闭包就是包含以上两个要素的特殊函数，它可以访问不是在它内部声明的变量，这个变量位于其他的作用域并且是未赋值的。

```go
package main

import "fmt"

func f(x int) func(int) int{
    g := func(y int) int{
        return x+y
    }
    // 返回闭包
    return g
}

func main() {
    // 将函数的返回结果"闭包"赋值给变量a
    a := f(3)
    // 调用存储在变量中的闭包函数
    res := a(5)
    fmt.Println(res)
    // 可以直接调用闭包
    // 因为闭包没有赋值给变量，所以它称为匿名闭包
    fmt.Println(f(3)(5))
}
```

* x不在g的作用域内，并且g引用了x，所以g是一个闭包。

* 如果x在传给g之前就被赋值了，g就不能称为闭包了

> 闭包不一定要通过另一个函数返回，可以直接自定义一个闭包,只要它访问了一个不在自己作用域并且尚未被赋值的变量即可。

```go
func main() {
    // 自由变量x
    var x int
    // 闭包函数g
    g := func(i int) int {
        return x + i
    }
    x = 5
    // 调用闭包函数
    fmt.Println(g(5))
    x = 10
    // 调用闭包函数
    fmt.Println(g(3))
```

闭包的作用：让多个闭包函数共享一个自由变量。

例如在第一个例子中，a := f(3)，给外层的变量x赋值为3，那么a(3),a(5),a(8)....所有闭包都共享这个值

什么时候使用闭包？

> 一般来说，可以将过程分成两部分或更多部分都进行工厂化的时候，就适合闭包(实际上，有些地方直接将闭包称呼为工厂函数)。第一个部分是可以给自由变量批量绑定不同的值，第二部分是多个闭包函数可以共享第一步绑定后的自由变量。
