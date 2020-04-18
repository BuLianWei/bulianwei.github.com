# Golang

## 数据类型

### 数值类型

#### bool

#### byte
byte = uint8

#### int
有符号类型：int8，int16，int32，int64
无符号类型：uint8，uint16，uint32，uint64
rune = int32

#### float
float16，float32

#### string

#### 数组

数组类型声明赋值方式

- 1

  ```
  var arr [3]int
  
  arr[0] = 1
  arr[1] = 2
  arr[2] = 3
  
  ```

- 2
  ```
  var arr =[3]int{1,2,3}
  ```

- 3

  ```
  var arr =[...]int{1,2,3}
  ```

- 4 

  ```
   arr :=[3]int{1,2,3}
  ```

- 5 

  ```
   arr :=[...]int{1,2,3}
  ```



不管是使用...代替size还是有具体size 在声明赋值时必须有size

### 引用类型

#### slice

#### map



## 变量
go语言的变量声明了就必须使用，不用就别声明，否则编译不通过

### 函数变量声明

#### 单一变量声明
关键字	变量名称	变量类型
~~~go
var a	int	
~~~
声明的同时可以赋值变量，同时赋值可以省略变量类型
~~~go
var a	= 0	
~~~
#### 多变量同时声明1
关键字	变量名称，变量名称	变量类型
多变量同时声明时，必须为多个类型一样的变量
~~~go
var a,b,c int
~~~
声明的同时可以赋值变量，同时赋值时可以省略变量类型
~~~go
var a,b,c = 0,0,0
~~~
#### 多变量同时声明2
关键字	（
变量名称	变量类型	
变量名称	变量类型
）
带有括号的多变量赋值时，变量类型可以不同
~~~go
var (
  a int
  b string
)
~~~
其中并且可以进行赋值操作，变量类型可以直接省略
~~~go
var (
  a = 0
  b = "bb"
)
~~~

#### 简短声明赋值
变量名称  :=  变量值
~~~go
a := 0
~~~

### 全局变量声明

全局变量声明，包含以上**单一变量声明**，**多变量同时声明1**，**多变量同时声明2**，声明方式，但是只有**多变量同时声明2**声明方式可以同时进行赋值操作，其他方式不支持赋值操作

### 指针

&：取地址

\*：取值

#### 指针数组
\*[4]Type
首先是一个指针，存储的是数组地址



var arr [3]int{1,2,3}

var p1 *[3]int

p1=&arr

(*p1)[2]=100 简写成：p1[2]=100。p1操作的是arr数组

#### 数组指针
首先是一个数组，存储的是指针

[4]\*Type



>注意：
>	\*[4]int	//指针，4个长度的int类型数组的指针
>	[4]\*int	//数组，4个长度的int指针地址的数组
>	\*[4]\*int	//指针，4个长度int指针地址数组的指针
>	\*\*[4]\*int	//指针，4个长度int指针地址数组的指针的指针




## 条件语句

### if
- 简单形式
  ~~~go
  if 1==2 {
  
  }
  ~~~

- 复合形式
  - 1
    ~~~go
    if a := 0; a ==0 {
    
    }
    ~~~
  - 2
    ~~~go
    if  {
    
    }
    ~~~
    当不写条件时，当作true处理
### switch

- 简单形式
  ~~~go
  a := 1
  switch a {
  case 1:
    println(1111)
  default:
  }
  ~~~


- 复合形式
  - 1
    ~~~go

    switch a := 1; a {
    case 1:
      println(1111)
    case 2,3,4:
      println(222,333,444)
    default:
    }
    ~~~

  - 2

    ~~~go
    a := 4
    switch  {
    case a>=0:
      println(">=0")
    case a<=5:
      println("<=4")
    }
    ~~~
    当不写条件时，当作true处理。此处只会输出一种情况`>=0`，switch处理时只要满足一种情况就不会再进行判断
    
#### switch穿透 fallthrough
  ~~~go
  a := 1
  switch a {
  case 1:
    println(1111)
    fallthrough
 case 2:
    println(2222)
  default:
  }
  
  =======输出为=======
  1111
  2222
  ~~~
fallthrough 可以穿透case，链接上下两个case一起执行

> 注意：
> a := 1
> switch a {
>
> default:
>     println("default")
>
>   case 1:
>     println(1111)
>
> }
> =================输出为===========
> 1111


## 循环语句

go语言里面没有while，do while 等其他循环结构，只有for



- 简单形式

  ~~~go
  for i := 0; i < 10; i++ {

  }
  ~~~

- 复合形式

  - 1
    
    ~~~go
    for {
    
    }
    ~~~
    不写条件，当作true处理
    
  - 2 
  
    ~~~go
    for index,value := range []int{1,2,3,4} {
      fmt.Printf("index:%d,value:%d \n",index,value)
    }
    ~~~
    range关键字可以取出数组，切片，集合等容器里面的数据



## defer关键字

defer 延迟，推迟执行。遵循栈的执行原则先进后出，参数变量是已经传递，是在return语句前执行


## 函数


func name(a int,b int) (int){

return 0

}

### 函数返回值

~~~go
func name ()(int){

return 1

}



func name ()int{

return 1

}



func name ()(sum int){

sum:=1

return

}

~~~





## 结构体
深拷贝，值传递
- 简单
  ~~~go
  type	name	struct {
    a int
    b string
  }
  ~~~

- 匿名结构体

  ~~~go
  struct {
    a int
    b string
  }
  ~~~

- 匿名结构体字段

  ~~~go
  type	name	struct {
   int
   string
  }
  ~~~

  匿名结构体字段访问时，使用匿名字段类型`name.int`，`name.string`，匿名的字段类型不能重复

  ~~~go
  type	name	struct {
   int
   string
   int	//不可以
  }
  ~~~


### 提升字段
结构体可以嵌套，当一个结构体内嵌套另一个匿名结构体时，可以直接使用字段名称访问被嵌套结构体的字段，这叫做字段提升。
字段提升只有在被嵌套结构体为匿名的时候可用。
如果结构体中的字段与被嵌套结构体的字段一样，则访问的为本结构体字段


~~~go
type Person struct {
	name string
	age int
}

type Student struct {
	Person
	addre string
	name string
}

p:=Person{"nihao",10}
fmt.Println(p)

s:=Student{Person{"wohao",20},"beij","djh"}
fmt.Println(s)
fmt.Println(s.Person.name,s.Person.age,s.addre,s.name)
fmt.Println(s.name,s.age,s.addre)


======================输出为============================

{nihao 10}
{{wohao 20} beij djh}
wohao 20 beij djh
djh 20 beij
~~~



## 方法
方法是一个作用在struct结构体上的函数


~~~go
func (s Struct) name() {

}
~~~

实例

~~~go
type Person struct {
	name string
	age int
}

type Student struct {
	Person
	addre string
	name string
}
func (p Person) run(){
	fmt.Println("p run")
}

func (s Student) run()  {
	fmt.Println("s run")
}



p:=Person{"lisi",10}
fmt.Println(p)
p.run()
s:=Student{Person{"wangwu",20},"beij","xiaowangwu"}
fmt.Println(s)
s.Person.run()
s.run()





===============输出为================
{lisi 10}
p run
{{wangwu 20} beij xiaowangwu}
p run
s run
~~~

## 接口（interface）

接口可以看作是一些方法的集合。接口是为了解耦合，go语言的接口是非侵入式的。空接口可以看作是任意类型

~~~go
type name interface{
  start()
  end()
}
~~~

### 接口断言
利用接口来判断类型

- 1
  ```go
  instance := 接口对象.(实际类型)   //不安全，会panic（）
  instance,err := 接口对象.(实际类型)	//安全
  ```
- 2
  ```go
  switch instance := 接口对象.(type) {
    case 实际类型：
      xxx
    case 实际类型：
      xxxx
  }
  ```


## type 关键字

- 定义新类型
  type myint int
  定义一个int类型的myint类型，但是int和myint类型不一样不能通用
- 给类型其别名
  type myint = int 
  给一个int类型起一个别名，int和myint可以通用

## 创建对象

### new

### make

- slice
  - make([]int,10)	
    长度（len）和容量（cap）都为10，默认值为int的零值
  - make([]int,0,10)	
    长度（len）为0，和容量（cap）为10，因为0个长度所以没有值
- map
  - make(map[string]string)
    
  - make(map[string]string,10)
  
- chan
  - make(chan bool)
    没有缓存的chan
  - make(chan bool,10)
    长度为10的chan

## 访问权限
名称首字母大写时，可以在其他地方导入，小写其他地方访问不了


## IO

## 协程（Gorutine）

## 管道（chan）

## 反射（reflect）

### 基本数据类型反射使用

#### 修改值
- step1:
  获取反射对象Value。要修改对象，就要获取对象的地址，用对象的引用地址来更改里面的具体值
  ~~~go
  var a int = 0
  value := reflect.ValueOf(&a)
  ~~~
- step2:
  获取反射对象的指针对象
  ~~~go
  elem := value.Elem()
  ~~~
- step3:
  修改值
  ~~~go
  elem.SetInt(9)
  ~~~

完整步骤：
~~~go
var a int = 0
value := reflect.ValueOf(&a)
elem := value.Elem()
if !elem.CanSet {  //判断反射对象是否可以修改，不可以直接返回不做修改
  return
}

if elem.Kind() ==reflect.Int{ //判断对象是否是int类型
  elem.SetInt(9)
}

~~~

### 函数调用

~~~go
func star()  {
  fmt.Println("call this func")
}

valueOf := reflect.ValueOf(star)  //这里star不能加（）

if valueOf.Kind() == reflect.Func {
  valueOf.Call(nil)
}
~~~

### struct 的反射使用

#### 获取字段名称和具体值

- step1:
  获取反射valueOf和typeOf实例
  ~~~go
  type Person struct {
    Name string
    Age  int
  }
  p:=Person{"zhangshan",30}

  valueOf := reflect.ValueOf(p)
  typeOf := reflect.TypeOf(p)
  ~~~


- step2:
  通过实例获取字段名称和字段实际值
  
  ~~~go
  for i := 0; i < typeOf.NumField(); i++ {
    fmt.Println("fieldname:  ",typeOf.Field(i).Name)
    fmt.Println("value:  ",valueOf.Field(i))
  }
  ~~~


完整代码：
~~~go
type Person struct {
  Name string
  Age  int
}

p:=Person{"zhangshan",30}

valueOf := reflect.ValueOf(p)
typeOf := reflect.TypeOf(p)


for i := 0; i < typeOf.NumField(); i++ {
  fmt.Println("fieldname:  ",typeOf.Field(i).Name)
  fmt.Println("value:  ",valueOf.Field(i))
}
~~~
#### 修改字段值

- step1:
  获取反射valueOf实例
  ~~~go
  type Person struct {
    Name string
    Age  int
  }
  p:=Person{"zhangshan",30}

  valueOf := reflect.ValueOf(&p)  //这里要填写p的地址
  ~~~
  
- step2:
  修改相应的字段值
  ~~~go
  valueOf.Field(0).SetString("wangwu")
  ~~~
  

完整代码：
~~~go
type Person struct {
	Name string
	Age  int
}
p:=Person{"zhangshan",30}

valueOf := reflect.ValueOf(&p)  //这里要填写p的地址

if valueOf.Kind() == reflect.Ptr {
  if valueOf.Elem().CanSet(){
    elem := valueOf.Elem()
  //  field := elem.Field(0)  //通过字段序号获取字段
    field := elem.FieldByName("Name")  //通过字段名称来获取字段
    field.SetString("wangwu")
  }
}
~~~

> 注意：
> Person的字段为大写开头，为可访问字段，如果小写，则会发生程序panic
> `reflect: reflect.flag.mustBeAssignable using value obtained using unexported field`

#### 调用方法

##### 调用无参数方法

- step1:
  获取反射valueOf实例
  ~~~go
  type Person struct {
    Name string
    Age  int
  }
  p:=Person{"zhangshan",30}
  
  func (p Person) Run() {
    fmt.Println("p run")
  }
  
  valueOf := reflect.ValueOf(p)
  ~~~
  
- step2:
  通过反射实例获取方法实例
  ~~~go
  method := valueOf.MethodByName("Run")
  ~~~
- step3:
  调用方法
  ~~~go
  method.Call(nil)
  ~~~

完整代码：
~~~go
type Person struct {
  Name string
  Age  int
}
p:=Person{"zhangshan",30}
  
func (p Person) Run() {  //方法名称一定要大写
  fmt.Println("p run")
}
  
valueOf := reflect.ValueOf(p)

// method := valueOf.Method(0)  //通过序号获取方法实例
method := valueOf.MethodByName("Run")

if method.Kind() == reflect.Func {
  // method.Call(make([]reflect.Value,0))  //传入一个空切片
  method.Call(nil)  //直接传入nil空值
}
~~~

##### 调用多参数方法

调用多参数方法和无参数方法执行步骤一样，只是在调用时传入参数的切片即可`method.Call([]reflect.Value{reflect.ValueOf("lii"),reflect.ValueOf("nihao"),reflect.ValueOf(5)})`



~~~go
func (s Person) Write(name string,msg string, length int) {
  fmt.Println(name,"写了",msg,"一共",length)
}
~~~



## RPC

### gRPC

## Protobuf











fallthrough   在switch 语句中进行穿透，链接两个case 



函数：



当参数类型一致时，前一个参数可以省略类型

函数参数中如果有可变参数，可变参数放到最后，参数列表中只能有一个可变参数

函数返回值





指针







