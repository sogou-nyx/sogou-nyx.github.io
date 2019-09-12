---
title: Golang中的方法
date: 2019-09-12 
tags:
- Golang
---

# Golang中的方法

Golang并非是面向对象的语言，但是它可以模拟面向对象。Golang中的struct就类似于面向对象语言中的类，那么既然有了类，就要有对应的类的方法。Golang中以接收器（reciever）的形式实现struct的方法。

##	1.方法的声明

方法声明的形式如下

```go
type mytype struct {}

func (m mytype) method(para) return_type {}
func (m *mytype) method(para) return_type {}
```

> 这就表示方法method是绑定在mytype上的，就类似于面向对象语言中类的成员方法

下面是一个实例

```go
type Point struct {
	X, Y float64
}

func Distance(p, q Point) float64 {
	return math.Hypot(q.X-p.X, q.Y-p.Y)
}

//类型Point的方法
func (p Point) Distance(q Point) float64 {
	return math.Hypot(q.X-p.X, q.Y-p.Y)
}

func main() {
	p := Point{1, 2}
	q := Point{4, 6}

	fmt.Println(Distance(p, q))
	//p调用属于类型Point的方法Distance
	fmt.Println(p.Distance(q))
}
```

> * 以上例子中，虽然声明了两个同名的Distance函数，但是不会报错。因为第一个是属于当前包的包级别函数，第二个是属于Point类的方法。
> * Go语言中，不同类型可以有相同的方法名，只要定义方法时用接收器区分开就可以。



```go
type myslice []int

func (s myslice) sum() int {
	sum := 0
	for _, v := range s {
		sum += v
	}
	return sum
}

func main() {
	s := myslice{1, 2, 3}
	fmt.Println(s.sum())
}
```

> * Go语言中，与面向对象语言不同的是它不仅仅可以为struct定义方法，也可以为type定义的类型别名、slice、map、channel、func类型定义方法。但内置简单数据类型(int、float等)不行，interface类型不行。
> * 上述不可以为简单类型定义方法，可以通过使用type a int这种形式为简单类型定义方法。
> * 面向对象语言中，类的方法要定义在类的内部，Go语言中struct可以与方法的定义分开在不同的文件中，但一定要在一个包内。
> * 方法就是函数，所以Go语言中没有重载的概念，同一类型中所有方法名都必须唯一



## 2.方法和函数的区别

本质上方法就是函数，方法是绑定了类型的特殊函数，需要指定类型的实例才可以调用。



## 3.值类型和指针类型的接受者

如果接收者是一个指针类型，那么同实例化类型一样会自动解除引用

```go
//实例化类型
type Person struct {name, age int}

p := Person{}//值类型的Person实例
p1 := new(Person)//指针类型的实例
//在需要访问或调用mytype实例属性时候，如果发现它是一个指针类型的变量，Go会自动将其解除引用
p.name = "xxs"
p1.name = "sdad"//不需要(*p1).name

//接收者为指针类型
func (a *mytype1) add() ret_type {}
a.add()//不需要(*a).add()
```

方法的接受者有两种类型：值类型和指针类型。它们在使用上有很大的区别

```go
func (p person) setname(name string) { p.name = name }
func (p *person) setage(age int) { p.age = age }		
```

* 对于setname方法，它的reciever是值类型的，那么在接收参数时是拷贝完整的Person的数据结构，无论实例是值类型还是指针类型，方法内部使用和修改的都是实例的副本，不会影响外部原始副本的值。
* 对于setage方法，它的reciever是指针类型的，那么在接收参数时是拷贝指针，无论实例是值类型还是指针类型，方法内部使用和修改的都是原始实例的数据。

如下例所示

```go
type Person struct {
	name string
	age  int
}

func (p *Person) getname() string {
	return p.name
}

func (p *Person) getage() int {
	return p.age
}

//这里比较reciever是值类型和指针类型的区别
func (p Person) setname(name string) {
	p.name = name
}

func (p *Person) setage(age int) {
	p.age = age
}

func main() {
	p1 := Person{}
	p1.setname("nyx")
	p1.setage(10)
	fmt.Println(p1.getname())//输出""
	fmt.Println(p1.getage())//输出10

	p2 := new(Person)
	p2.setname("nyx")
	p2.setage(10)
	fmt.Println(p2.getname())//输出""
	fmt.Println(p2.getage())//输出10
}
```



## 4.嵌套struct中的方法

与属性类似，内部struct的方法也会被嵌套，外部struct可以直接调用内部struct的方法。如果外部struct有方法与内部strcut方法同名，那么直接调用的话就是调用外部strcut的方法，想要调用内部struct的对应方法需要使用间接调用的方式

> Go语言的struct支持嵌套多个匿名字段，也就是说一个struct可以有多个内部struct，相当于支持多重继承



## 5.重写String()方法

