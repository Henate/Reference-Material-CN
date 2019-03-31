# Golang基础知识


## Golang包管理
## 工作环境
windows下查看go环境变量: 
```
#win+R -> cmd -> go env
```

windows下修改go环境变量:
```
#系统属性 -> 高级 -> 环境变量
#修改环境变量为当前工程文件的上一级目录

```

### 同级目录

> .go文件放在相同目录文件夹中

同级目录中的.go文件,包名需要一致:
```
#目录Example_file中所有.go文件开头第一行均为:
package Example_pack
```

同级目录中的.go文件可直接调用同级目录其他文件的function无需带包名:
```
#file_1.go中定义:
fun print_log(){
    fmt.Println("hello world")
}

#file_2.go中使用:
fun run(){
    print_log()
}
```
### 非同级目录
> .go文件放在不同目录文件夹中
> 包中成员以名称⾸字母⼤⼩写决定访问权限：
> 1. public: ⾸字母⼤写，可被包外访问 
> 2. private: ⾸字母⼩写，仅包内成员可以访问

非同级目录中的.go文件,包名不能相同,以防不知如何调用包内function

包声明及导入
```
#main.go中:         //main.go在目录在src/中
package main
import (
    "calc"          //导入需要调用函数所属的包
)
func main() {
    calc.Add(1,2) 
}

#src/cal/calc.go中      //calc.go在目录在src/cal/中
package calc
func Add(var_1, var_2 int)(result int) {     //注:首字母大写才能被包外调用
    result = var_1 + var_2
    return
}
```

导入包与init()函数

> 当一个包被导入时，如果该包还导入了其它的包，那么会先将其它包导入进来，然后再对这些包中的包级常量和变量进行初始化，接着执行init函数（如果有的话），依次类推。等所有被导入的包都加载完毕了，就会开始对main包中的包级常量和变量进行初始化，然后执行main包中的init函数（如果存在的话），最后执行main函数。

### 关键字import导入包

单独导入
```go
import "fmt"
import "os"
```

单行单独导入
```go
import "fmt"; import "os"
```

一次性导入多个包Ⅰ
```go
import (
   "fmt"
   "os"
)
```
一次性导入多个包Ⅱ
```go
import ("fmt"; "os")
```

总结：
使用关键字import导入某路径下的包，使用双引号""包含包名。import可用括号()导入多个包，若需要在一行导入多个包需要使用分号;

如果包名不是以 . 或 / 开头，如 "fmt" 或者 "container/list"，则 Go 会在全局文件进行查找；如果包名以 ./ 开头，则 Go 会在相对目录中查找；如果包名以 / 开头（在 Windows 下也可以这样使用），则会在系统的绝对路径中查找。

### 使用包中的变量、函数
规则：
当标识符（包括常量、变量、类型、函数名、结构字段等等）以一个大写字母开头，如：Group1，那么使用这种形式的标识符的对象就可以被外部包的代码所使用（客户端程序需要先导入这个包），这被称为导出（像面向对象语言中的 public）；标识符如果以小写字母开头，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的（像面向对象语言中的 private ）。

当使用某个包中的变量将使用`.`进行调用，例如`pack1.Thing` 和 `pack2.Thing`

## Golang变量定义格式

### 常量定义

#### iota
使用iota可以获得有自增形式的运行于公式下的常量。

```go

type BitFlag int
const ( 
    Active  BitFlag = (1 << iota) + 1 // 1 << 0 + 1 == 2
    Send                                            // 1 << 1  + 1 == 3 
    Receive                                        // 1 << 2  + 1 == 5
)
```

### 变量类型强制转换


由于 Go 语言不存在隐式类型转换，因此所有的转换都必须显式说明，就像调用一个函数一样（类型在这里的作用可以看作是一种函数）：

公式：类型 B 的值 = 类型 B(类型 A 的值)
```go
valueOfTypeB = typeB(valueOfTypeA)
```
示例
```go
a := 5.0
b := int(a)
```

### 定义变量的两类方法

两大种进行定义的方法
1. 普通定义
2. 由编译器自动推断自动推导变量类型(在编译时自动识别需要定义的变量是何种类型，不同于python等动态语言是在运行时判定变量类型)

小结:
 1. 普通定义需要带上变量类型type(如int string)在变量名var_name后.
 2. 自动推导使用 := 在变量名var_name及值value中间

### 1.普通定义格式 

 1. var [var_name] [var_type] = value

由于类型明确,常用于全局变量定义
```
//整形int 
var v1 int = 1      //方式1, 编译器根据等号左边定义的int型决定变量的类型
```


### 2.自动推导格式

1.  var_name := value 
2.  var [var_name] = value

 
 定义直接且简单,常用于：
 1. for循环的idx变量
 2. 接收函数的返回值(可忽略返回的类型具体要怎么定义)
 3. 方法的返回值

 
 等号右边输入任何格式的值,将被自动推导.
```go
var v2 = "haha"     // 方式2，编译器自动推导出v2的类型为string
v3 := 3.14          // 方式3，编译器自动推导出v3的类型为float64
```

自动推倒类型
```go
var (
	//变量名为default404Body ，值为("404 page not found") 使用直接推倒特性，变量类型为[]byte
	default404Body   = []byte("404 page not found")
	default405Body   = []byte("405 method not allowed")
	defaultAppEngine bool
)
```

并行赋值 ,可用于多个不同类型的变量并行赋值
```go
var intVar, stringVar = 4, "haha"
const beef, two, c = "eat", 2, "veg"
```

### 各类型变量定义
#### 指针变量定义 - var / := 

 1. 普通定义
 2. 自动推导
 3. 可使用`new()`指向临时空间

```go
#普通定义&自动推导
var p1 *string = &v2    //格式为*string, 使用前需要明确知道右值是什么类型
var p2 = &v2            //直接推导右值类型赋值到p2
p3 := &v2               //直接推导右值类型赋值到p2

#new()创建指针临时指向内存空间
#以上三种方法均可使用new()暂时指向一片值为零的空间
var p4 *string = new(string)    
var p5 = new(string)
p6 := new(string)

#注:使用new后一定需要指向真实的地址,以免别的func操作到空指针
p5 = &v2
p4 = &v2
p6 = &v2
	
```

注:
 1. 若使用普通定义需在类型名前加星号*
 2. 右值若是一个变量需要加取地址符&以赋值到指针变量

#### 数组变量定义 - var / := 
 1. 普通定义
 2. 自动推导
 3. 可使用`...`不指定数组第一维空间长度

>- 数组的长度在定义之后无法再次修改,数组是值类型.

普通定义,需要用for循环初始化
```
	var a1 [3]int
	for i := 0; i < len(a1); i++ {
		a1[i] = i
	}
```

直接推导,可以用大括号{}赋值初始化数组
```
a2 := [3]int{111, 222}
a4 := [...]int{111, 222, 333, 444} //通过初始化值确定数组长度为4

a5 := [4][2]int{{10, 11}, {20, 21}, {30, 31}, {40, 41}}   //全部赋值
a6 := [...][2]int{{10, 11}, {20, 21}, {30, 31}, {40, 41}} //第一维可以填...第二维必须是确定值
a7 := [4][2]int{1: {10, 11}}                              //第一行赋值
```

#### 切片变量定义 - var / := / make 

 1. 切片的基本定义初始化方法与数组一样 
 2. 中括号内无需加任何东西
 3. 可用make创建`(最常用)`

```
a2 := []int{111, 222}
a3 := []int{111, 222, 333}      
a4 := []int{111, 222, 333, 444} 

a5 := [][]int{{10, 11}, {20, 21}, {30, 31}, {40, 41}} //全部赋值
a6 := [][]int{{10, 11}, {20, 21}, {30, 31}, {40, 41}} //第一维可以填...第二维必须是确定值
a7 := [][]int{1: {10, 11}}                            //第一行赋值

//最常用使用make创建切片 make(type, len, cap)
s2 := make([]int, 5, 10)
```



#### 结构体变量定义 - type / var / := 

 1. 普通定义赋值 - var
 2. 自动推导赋值 struct_name := struct_type{}
 3. 使用type作定义
 4. 可包含匿名字段作成员变量
 5. 可继承匿名字段方法


定义 (type):
```
//定义结构体
type Example_struct struct{
    notype_struct
    Example_int     int
    Example_string  string
}
```

赋值格式:
 1. 普通初始化 var
 2. 直接推导赋值 := 
```
//普通初始化
var s1 Example_struct
s1.name = "pis"
s1.id = 9527

//直接推导
s2 := Example_struct{"pis2", 9528}
s3 := Example_struct{name: "pis3"}
```

#### 接口变量定义 - type

 1. 接口名一般以er结尾
 2. 接口需要匿名方法为类型定义一个func同名方法以实现

```
//接口定义
type Humaner interface {
    SayHi()
}
```

```
//接口实现
func (s *Student) SayHi() {
    fmt.Printf("Student[%s, %f] say hi!!\n", s.name, s.score)
}
```

#### 映射变量定义 - var / make

map[keyType]valueType

```
//普通定义
var m1 map[string]int
```

使用make定义
``` 
//make(map, cap)
m2 := make(map[string]interface{}, 4)
```



### 变量名重命名格式
格式: `type [new_type] [old_type]`    
重命名后可直接用new_type作为格式去定义变量,与old_type使用方法完全等效



## 控制结构

### switch
switch支持所有相等判断的类型都可以作为测试表达式的条件，包括 int、string、指针等。即switch后面的判断条件可以不仅仅为整形、字符串。还可以是指针。

swtich不需要使用break跳出条件


switch不会自动地去执行下一个分支的代码。
```go
switch i {
        case 0: // 空分支，只有当 i == 0 时才会进入分支
        case 1:
                f() // 当 i == 0 时函数不会被调用
}
```

如果在执行完每个分支的代码后，还希望继续执行后续分支的代码，可以使用 `fallthrough` 关键字来达到目的。
```go
switch i {
        case 0: fallthrough
        case 1:
                f() // 当 i == 0 时函数也会被调用
}
```

### for

#### 与C语言不同点

* 关键字`for`后的条件无需使用括号

```go

   for i := 0; i < 5; i++ {
                fmt.Printf("This is the %d iteration\n", i)
   }
```

* 条件具有平行赋值特性

```go
for i, j := 0, N; i < j; i, j = i+1, j-1 {} //i与j平行赋值
```

* 无while关键字，使用for进行无限循环
```go
for {
}
```

#### for-range 结构

一般形式为：`for idx, val := range Var { }`
可以通过迭代的方式获取变量Var中的索引idx与值val
```go

for pos, char := range str {
...
}
```


## 函数
    
### 特性

* Go是编译型语言，所以函数编写的顺序是无关紧要的。顺序不影响互相调用。

* 可返回多个返回值

* 多个返回值作另一函数实参
假设` f1 `需要 3 个参数` f1(a, b, c int)`，同时` f2` 返回 3 个参数 `f2(a, b int) (int, int, int)`，就可以这样调用 f1：`f1(f2(a, b))`。
```go
func threeArgReturn() (i, j, k int) { 
    return 1, 2, 3
}

func threeArgInput(x, y, z int) { 
    fmt.Println("result:", x+y+z)
}

func main() {
    threeArgInput(threeArgReturn())
}       
```

* 不支持函数重载（function overloading）因此每个函数名字需要独立。
* 没有形参的名字，只有形参类型的函数通常被称为 niladic 函数（niladic function），就像 func f(int, int, float64)
```go
func getX2AndX3(input int) (int, int) {     //返回值不用明确具体名字
    return 2 * input, 3 * input                 //直接返回形参值  
}
```


### 按值传递（call by value） 按引用传递（call by reference）
默认情况下GO传递参数方式为按值传递(即传递副本)。但传递指针（一个32位或者64位的值）的消耗都比传递副本来得少


在函数调用时，像切片（slice）、字典（map）、接口（interface）、通道（channel）这样的引用类型都是默认使用引用传递（即使没有显式的指出指针）。

### 命名作返回值（named return variables）

返回值在函数中已有初始化，当调用return时可直接返回值。
PS：需要有确定的返回形参名
```go
func getX2AndX3_2(input int) (x2 int, x3 int) {
    x2 = 2 * input
    x3 = 3 * input
    // return x2, x3
    return
}
```

### 函数通过指针修改变量值

当指针作函数的参数时，在函数内部可以通过修改指针指向的值修改某变量值。
```go
// this function changes reply:
func Multiply(a, b int, reply *int) {
    *reply = a * b
}
```

### 传递变长参数

#### 变长参数简析
* WHAT
函数接收 0~N 个参数
* WHY？
当一个函数不确定需要接收多少个参数时，可使用变长参数。
* HOW?
使用 `...ArgType` 作为变长参数的类型
```go
func myFunc(a, b, arg ...int) {}
```

#### 往形参为变长参数的函数传值

* 若变长参数都为同一个类型，使用切片slice作为参数类型

如：`func(slice...)`
```go

func main() {
        x := min(1, 3, 2, 0)
        fmt.Printf("The minimum is: %d\n", x)
        slice := []int{7,9,3,5,1}
        x = min(slice...)
        fmt.Printf("The minimum in the slice is: %d", x)
}

func min(s ...int) int {
        if len(s)==0 {
                return 0
        }
        min := s[0]
        for _, v := range s {
                if v < min {
                        min = v
                }
        }
        return min
}
```

* 若变长参数为非同一个类型，可使用结构体/空接口作为参数类型

1. 使用结构体：定义一个结构类型，假设它叫 Options，用以存储所有可能的参数：
```go
type Options struct {

        par1 type1,
        par2 type2,
        ...
}
```
函数 F1 可以使用正常的参数 a 和 b，以及一个没有任何初始化的 Options 结构： `F1(a, b, Options {})`。如果需要对选项进行初始化，则可以使用 `F1(a, b, Options {par1:val1, par2:val2})`。

2. 使用空接口：如果一个变长参数的类型没有被指定，则可以使用默认的空接口 interface{}，这样就可以接受任何类型的参数（详见第 11.9 节）。该方案不仅可以用于长度未知的参数，还可以用于任何不确定类型的参数。一般而言我们会使用一个 `for-range` 循环以及 switch 结构对每个参数的类型进行判断：
```go
func typecheck(..,..,values … interface{}) {

        for _, value := range values {
                switch v := value.(type) {
                        case int: …
                        case float: …
                        case string: …
                        case bool: …
                        default: …
                }
        }
}
```


### 内置函数

| 名称 | 说明 |
| --- | --- |
| close | 用于关闭管道通信 |
| len、cap | len 用于返回某个类型的长度或数量（字符串、数组、切片、map 和管道）；cap 是容量的意思，用于返回某个类型的最大容量（只能用于切片和 map） |
| new、make |  new 和 make 均是用于分配内存：new 用于值类型和用户定义的类型，如自定义结构，make 用于内置引用类型（切片、map 和管道）。它们的用法就像是函数，但是将类型作为参数：new(type)、make(type)。new(T) 分配类型 T 的零值并返回其地址，也就是指向类型 T 的指针（详见第 10.1 节）。它也可以被用于基本类型：v := new(int)。make(T) 返回类型 T 的初始化之后的值，因此它比 new 进行更多的工作。new() 是一个函数，不要忘记它的括号|
| copy、append |用于复制和连接切片  |
| panic、recover |均用于错误处理机制  |
| print、println | 底层打印函数，在部署环境中建议使用 fmt 包 |
| complex、real imag | 用于创建和操作复数 |


### 递归函数
在一个函数中调用自己，称为函数的递归。

经典递归例子：斐波那契数列
```go

package main

import "fmt"

func main() {
        result := 0
        for i := 0; i <= 10; i++ {
                result = fibonacci(i)
                fmt.Printf("fibonacci(%d) is: %d\n", i, result)
        }
}

func fibonacci(n int) (res int) {
        if n <= 1 {
                res = 1
        } else {
                res = fibonacci(n-1) + fibonacci(n-2)
        }
        return
}    
```

### 闭包
#### 匿名函数
定义一个函数但不定义函数名，称为匿名函数。

有两种定义匿名函数的方式：
使用变量接收匿名函数的地址，通过调用变量使用匿名函数。

1. 往变量传值使用匿名函数
```go
func main() { 
    AnonyAdd := func(x, y int) int { return x + y } 
    fmt.Println(AnonyAdd(3, 4))
    }
```

2. 定义匿名函数后在括号()中赋值
```go
func main() { 
    AnonyMin := func(j, k int) int {         
        return j - k 
    }(6, 3) 
    fmt.Println(AnonyMin)
}
```

#### 高阶函数-函数返回值为一个函数

* 特性：闭包函数内的变量值不变

闭包函数保存并积累其中的变量的值，不管外部函数退出与否，它都能够继续操作外部函数中的局部变量。
```go
func main() {
        var f = Adder()
        fmt.Print(f(1), " - ")
        fmt.Print(f(20), " - ")
        fmt.Print(f(300))
}

func Adder() func(int) int {
        var x int
        return func(delta int) int {
                x += delta
                return x
        }
}
```

---

## 复合类型

### 指针

1. 默认值 nil，没有 NULL 常量 
2. 操作符 "&" 取变量地址， "*" 通过指针访问目标对象 
3. 不支持指针运算，不支持 "->"运算符，直接⽤ "." 访问目标成员
4. 申请空间以及赋值,go语言中new后无需释放内存

>定义指针类型

在类型前加上*可以定义该类型的指针
```go
var intP *int   //定义指向int类型的指针
```

>使用new创建内存空间
```
func ptr_c(){
    p := new(int)   //自动推导new一片int型空间,返回值为指针变量
    *p = 666        //使用指针复制int空间
}
```
>使用指针传递变量地址,在函数内科直接对变量内容进行操作
```
func swap02(x, y *int) {
    *x, *y = *y, *x
}

func main() {
    a := 10
    b := 20
    //swap01(a, b) //值传递
    swap02(&a, &b) //变量地址传递
    fmt.Printf("a = %d, b = %d\n", a, b)
}

```

### 数组

#### 数组声明
一维数组声明
```
func array() {
	var a [10]int       //int类型数组
	var b [10]string    //string类型数组  
	for i := 0; i < 10; i++ {
		a[i] = i
	}
	for i := 0; i < 10; i++ {
		b[i] = "haha"
	}

	fmt.Println("Arraya: ", a)
	fmt.Println("Arrayb: ", b)
}
```

直接推导声明二维数组数组, 使用不同方法赋值
```
    //一般定义二维数组方法
	array_a := [3][4]int{{1, 2, 3, 4}, {4, 5, 6, 7}, {8, 9, 10, 11}}
	fmt.Println("array_a: ", array_a)

    //第1行及第2行有某位不初始化,则默认值为0
	array_b := [3][4]int{{1, 2, 3, 4}, {4, 5, 6}, {8, 9, 10}}
	fmt.Println("array_a: ", array_b)

    //第0行初始化为{},即为全0
	array_c := [3][4]int{{}, {4, 5, 6, 7}, {8, 9, 10, 11}}
	fmt.Println("array_a: ", array_c)

    //制定第1行数组为{4, 5, 6, 7}
	array_d := [3][4]int{1: {4, 5, 6, 7}, {8, 9, 10, 11}}
	fmt.Println("array_a: ", array_d)
```
#### 数组比较
只能使用 == 和 != 比较,返回值:true/false
```
	array_a := [3][4]int{{1, 2, 3, 4}, {4, 5, 6, 7}, {8, 9, 10, 11}}
	array_a_compare := [3][4]int{{1, 2, 3, 4}, {4, 5, 6, 7}, {8, 9, 10, 11}}
	fmt.Println("array_a ==  array_a_compare", array_a == array_a_compare)
```

#### 算法
生成随机数
```
import (
	"fmt"
	"math/rand"     //还有随机数生成包
	"time"
)

func main() {
	rand.Seed(time.Now().UnixNano())                //以时间为rand seed
	fmt.Printf("Randint:%d\n", rand.Int())          //生成随机数
	fmt.Printf("Randint:%d\n", rand.Intn(10000))    //生成0-9999随机数
}

```

### 切片slice
>slice和数组的区别：声明数组时，方括号内写明了数组的长度或使用...自动计算长度，而声明slice时，方括号内没有任何字符。

#### 切片的创建
> 格式: slice := array[type, len, cap]

与数组相似,但中括号[]内无需填长度。
```
	s := []int{1, 2, 3, 4, 88}
	fmt.Println("s:", s, "len:", len(s), "cap:", cap(s))
	
	//使用make创建切片 make(type, len, cap)
	s2 := make([]int, 5, 10)
	fmt.Println("s2:", s2, "len:", len(s2), "cap:", cap(s2))
```

#### 切片截取
> 格式: slice_cut := array[low:high:max]

|操作           | 描述| 
| --------      | -----  |
| s[n]	        | 切片s中索引位置为n的项 | 
|s[:]	        |从切片s的索引位置0到len(s)-1处所获得的切片| 
|s[low:]	    |从切片s的索引位置low到len(s)-1处所获得的切片| 
|s[:high]	    |从切片s的索引位置0到high处所获得的切片，len=high| 
|s[low:high]	|从切片s的索引位置low到high处所获得的切片，len=high-low| 
|s[low:high:max]|从切片s的索引位置low到high处所获得的切片| 
|len(s)	        |切片s的长度，总是<=cap(s)| 
|cap(s)	        |切片s的容量，总是>=len(s)| 
|注:            |len=high-low，cap=max-low| 

直接截取数组或切片
```
	s := []int{11, 22, 33, 44, 88, 55, 66, 98}
	array := [8]int{11, 22, 33, 44, 88, 55, 66, 98}
	
	//可从切片或数组中截取,被截取的切片和数组不会有任何影响
	s2 := s[1:4:4]
	s3 := array[1:4]
```

修改切片截取即修改原值
```
    //当修改切片截取中的值,原array会一同修改
	s := []int{11, 22, 33, 44, 88, 55, 66, 98}

    //切片截取
	slice_modify_var := s[2:5:5]
	//使用切片可修改原s中的值.
	slice_modify_var[2] = 666
```

#### 向slice末尾追加参数
>格式: append(slice, var_1, var_XX)
```
    //使用append向切片末尾追加
	slice_array := []int{11}
	slice_array = append(slice_array, 24)
```
>append函数会智能地底层数组的容量增长，一旦超过原底层数组容量，通常以2倍容量重新分配底层数组，并复制原来的数据：

#### 复制slice
> 格式: copy(dst_slice, src_slice)
> 函数 copy 在两个 slice 间复制数据，复制⻓度以 len 小的为准，两个 slice 可指向同⼀底层数组。

####silce作函数参数
slice可直接传索引值,只需要把slice名字作参,即可修改值(相当于不带星号*的指针)
```
//函数调用以下func后,slice的值已被改变
func modify_slice(sl []int) {
	sl[0]++
}
```

### map
>格式 map[keyType]valueType

在函数间传递映射并不会制造出该映射的一个副本，不是值传递，而是**引用传递**

 1. 在一个map里所有的键都是唯一的，而且必须是支持==和!=操作符的类型，切片、函数以及包含切片的结构类型这些类型由于具有引用语义，不能作为映射的键，使用这些类型会造成编译错误.
 2. map值可以是任意类型，没有限制。map里所有键的数据类型必须是相同的，值也必须如何，但键和值的数据类型可以不相同。
 3. 注意：map是无序的，我们无法决定它的返回顺序，所以，每次打印结果的顺利有可能不同。
```
 var m1 map[int]string  //只是声明一个map，没有初始化, 此为空(nil)map
 fmt.Println(m1 == nil) //true
 
 

```
 

### 结构体
>定义格式: 
>type struct_name struct{}


注意:与函数一样,若结构体或结构体成员需要被外部文件调用,定义时首字母需要大写
```
type sample_struct struct {         //定义为Sample_struct则可被外部调用
	id   int                        //定义为Id则可被外部调用
	name string
	sex  byte
}

func struct_modify(p *sample_struct) {
	p.id = 666
	p.name = "hack"
}

func struct_init() {
	//顺序初始化
type part_struct struct {
	name string
	addr string
}

type p_part_struct struct {
	name string
	addr string
}

type sample_struct struct {
	part_struct
	*p_part_struct
	id   int
	name string
	sex  byte
}

func struct_init() {
	//顺序赋值
	s := sample_struct{part_struct{"pis2", "gz"}, &p_part_struct{"pPis", "pGz"}, 9527, "pis", 'm'}
	fmt.Println("Struct:", s)
	fmt.Printf("pname:%s, paddr:%s\n", s.p_part_struct.name, s.p_part_struct.addr)

	//部分成员赋值
	s1 := sample_struct{part_struct: part_struct{addr: "gz"}, p_part_struct: &p_part_struct{name: "pPis2"}, sex: 'm'}
	fmt.Println("Struct:", s1)
	fmt.Printf("pname:%s, paddr:%s\n", s1.p_part_struct.name, s1.p_part_struct.addr)

	//匿名字段成员与结构体成员同名, 赋值采用就近法则
	s2 := sample_struct{}
	s2.p_part_struct = new(p_part_struct)   //保持默认值需要用new申请空间,否则报错
	s2.name = "pis3"
	fmt.Println("Struct:", s2)
	fmt.Printf("pname:%s, paddr:%s\n", s2.p_part_struct.name, s2.p_part_struct.addr)
}

}
```

## 面向对象编程

>  1. 封装：通过方法实现 
>  2. 继承：通过匿名字段实现 
>  3. 多态：通过接口实现

### 一、继承

> Golang使用匿名字段实现继承
匿名字段可有:结构体,方法等

```go
//匿名字段为结构体:
type part_struct struct {           //匿名字段
	name string
	addr string
}

type sample_struct struct {         //继承匿名字段的结构体
	part_struct
	id   int
	name string
	sex  byte
}

func struct_init() {
	//顺序赋值
	s := sample_struct{part_struct{"pis2", "gz"}, 9527, "pis", 'm'}
	fmt.Println("Struct:", s)

	//部分成员赋值
	s1 := sample_struct{part_struct: part_struct{addr: "gz"}, sex: 'm'}
	fmt.Println("Struct:", s1)

	//匿名字段成员与结构体成员同名, 赋值采用就近法则
	s2 := sample_struct{}
	s2.name = "pis3"
	fmt.Println("Struct:", s2)

}
```

### 二、封装(方法)

> Golang使用函数(方法)实现封装

格式:
一般函数: func XXX{}
匿名函数: func {}
方法:     func (XXX type)XXX{}


实现方法:
>   格式：func (receiver ReceiverType) funcName(parameters) (results){}

- 可以给任意自定义类型（包括内置类型，但不包括指针类型）添加相应的方法。
- 参数 receiver 可任意命名。如⽅法中未曾使⽤，可省略参数名。
- 参数 receiver 类型可以是 T 或 *T。基类型 T 不能是接⼝或指针。
- 不支持重载方法，也就是说，不能定义名字相同但是不同参数的方法。


#### 1.为类型添加方法
```
//自定义类型UINT_8作为ReceiverType
type UINT_8 int

func (obj_var UINT_8) Add_sum(input UINT_8) UINT_8 {
	Add_result := obj_var + input
	return Add_result
}

func call_Add_sum() {
    //调用前定义UINT_8的变量
	var a UINT_8 = 3
	a = a.Add_sum(2)                        //遵循receiver.func()的格式
	fmt.Println(a)
}
```

#### 2.为结构体添加方法
结构体通常会使用type重定义结构体名字，可使用作ReceiverType
```
type sample_struct struct {
	id   int
	name string
	sex  byte
}

func (obj_struct sample_struct) PrintStructInfo() {
	fmt.Println("Struct info:", obj_struct)
}

func call_Modify_struct() {
	s := sample_struct{9527, "pis", 'm'}
	s.PrintStructInfo()                     //遵循receiver.func()的格式
}
```

#### 3.值语义:值作receiver / 引用语义:指针作receiver
值语义即传参为值,不会修改参数的内容,而使用引用语义并修改后,参数的内容会一并更改
```
//ReceiverType为值类型
func (obj_struct sample_struct) PrintStructInfo() {
	fmt.Println("Struct info:", obj_struct)
}

//ReceiverType为指针类型
func (obj_struct *sample_struct) ModifyStructInfo(id int, name string, sex byte) {
	obj_struct.id = id
	obj_struct.name = name
	obj_struct.sex = sex
}
```

#### 4.继承来自匿名字段的方法
如果匿名字段实现了一个方法，那么包含这个匿名字段的struct也能调用该方法。
```
type sample_struct struct {
	id   int
	name string
	sex  byte
}

func (obj_struct sample_struct) PrintStructInfo() {     //匿名字段定义了方法
	fmt.Println("Struct info:", obj_struct)
}

type student struct {
	sample_struct       // 匿名字段，那么student包含了sample_struct的所有字段
	name string
	addr string
}

func student_init() {
	s1 := student{sample_struct{9528, "pis", 'm'}, "pis2", "gz"}
	s1.PrintStructInfo()                       //调用继承自匿名字段的方法          
}
```

#### 5.方法的重写

当ReceiverType使用同名方法,会调用重写后的方法。而通过显式调用则可调用重写前的方法
```
type sample_struct struct {
	id   int
	name string
	sex  byte
}

type student struct {
	sample_struct       //结构体继承匿名字段
	name string
	addr string
}

//匿名字段的方法
func (obj_struct sample_struct) PrintStructInfo() {     
	fmt.Println("Struct info:", obj_struct)
}

//重写从匿名字段中继承的方法
func (obj_struct student) PrintStructInfo() {           
	fmt.Println("student:", obj_struct)
	
func student_init() {
	s1 := student{sample_struct{9528, "pis", 'm'}, "pis2", "gz"}
	s1.PrintStructInfo()                    //调用重写方法
	s1.sample_struct.PrintStructInfo()      //显式调用
```

#### 6.方法值&方法表达式
方法值:隐式传参
方法表达式:显式传参

```
	vfunc := s1.PrintStructInfo
	vfunc() //不传receiver,调用方法值

	pfunc := s1.pModifyStructInfo
	pfunc(s1)
	s1.PrintStructInfo()
```


### 三、接口 interface
> 格式: type inter interface {methos()}

>- 接⼝命名习惯以 er 结尾
>- 接口只有方法声明，没有实现，没有数据字段
>- 接口可以匿名嵌入其它接口，或嵌入到结构中


#### 1.创建接口
```
type sample_struct_1 struct {
	name string
	id   int
}

type sample_var string

//定义接口
type sampler interface { //和定义struct一样不用加()
	sayhi() //在此定义方法, 方法在别处实现
}

//实现接口中sayhi()方法
func (obj_name *sample_struct_1) sayhi() {
	fmt.Println("struct_1 sayhi!")
}

func call_func() {
	//定义接口变量
	var i sampler
	s2 := &sample_struct_2{"gz", 'm'}   //以结构体赋值的格式向接口传递参数
	i = s2
	i.sayhi()
}
```

> - 多态：调用同一个函数,表现出不同的功能,即多种形态。

#### 2.切片多态
使用切片有效管理方法
```
//创建切片指向结构体以使用方法
slic := make([]sampler, 3, 3)
slic[0] = s1
slic[1] = s2
slic[2] = v1

//顺序调用方法
//第一个值:下标(丢弃), 第二个值:内容(调用方法)
for _, i := range slic {
	i.sayhi()
}
```

#### 3.函数多态
使用函数的方法灵活调用方法
```
func call_interface(obj sampler) {
	obj.sayhi()
}

func call_func() {
	s1 := sample_struct_1{"pis", 9527}
	s2 := &sample_struct_2{"gz", 'm'}
	var v1 sample_var = "hello"
	call_interface(s1)
	call_interface(s2)
	call_interface(v1)
}
```

#### 空接口
> 格式: type inter interface {} (大括号内不加方法)

 > - 空接口可指向任何类型变量
空接口(interface{})不包含任何的方法，正因为如此，所有的类型都实现了空接口，因此空接口可以存储任意类型的数值。它有点类似于C语言的void *类型。

```
var v1 interface{} = 1     // 将int类型赋值给interface{}
var v2 interface{} = "abc" // 将string类型赋值给interface{}
var v3 interface{} = &v2   // 将*interface{}类型赋值给interface{}
var v4 interface{} = struct{ X int }{1}
var v5 interface{} = &struct{ X int }{1}

```


## 异常处理 

### 一、errors.go
>- Go语言引入了一个关于错误处理的标准模式，即error接口，它是Go语言内建的接口类型
>- 处理非致命错误

函数的异常返回值
函数中定义一个返回值为异常返回,可根据异常返回是否nil决定后续动作。
```
package main

import (
	"errors"    //异常处理包
	"fmt"
)
//两个返回值,一个为异常返回.
func Divc(a, b int) (err error, ret int) {
	err = nil
	if b == 0 {
		err = errors.New(" should not zero!")
	} else {
		ret = a / b
	}
	return
}

func main() {
	Div_err, Div_data := Divc(10, 0)
	if Div_err != nil {                 //若函数调用出现异常,在此处理.
		fmt.Println("err:", Div_err)
	} else {
		fmt.Println("result:", Div_data)
	}
```

### 二、panic

>- 格式: func panic(v interface{})
>- 处理致命错误

在通常情况下，向程序使用方报告错误状态的方式可以是返回一个额外的error类型值。

但是，当遇到不可恢复的错误状态的时候，如数组访问越界、空指针引用等，这些运行时错误会引起painc异常。这时，上述错误处理方式显然就不适合了。反过来讲，在一般情况下，我们不应通过调用panic函数来报告普通的错误，而应该只把它作为报告致命错误的一种方式。当某些不应该发生的场景发生时，我们就应该调用panic。

一般而言，当panic异常发生时，程序会中断运行，并立即执行在该goroutine（可以先理解成线程，在中被延迟的函数（defer 机制）。随后，程序崩溃并输出日志信息。日志信息包括panic value和函数调用的堆栈跟踪信息。

不是所有的panic异常都来自运行时，直接调用内置的panic函数也会引发panic异常；panic函数接受任何值作为参数。

```
//自定方法中用panic防止发生错误
func panic_example(){
    ret, error := rungo()
    if error != nill{
        panic("This is a error panic!!!")
    }
}
```

### 三、recover
> - 格式: func recover() interface{}
> - recover使当前的程序从运行时panic的状态中恢复并重新获得流程控制权。


```
//该代码片段放在可能出现panic的func内,可拦截panic,让程序继续往下走
defer func() {
	if err := recover(); err != nil {
		fmt.Println("err", err)
	}
}()
```





## 字符串处理

### 一、字符串操作

#### 1.Contains - 查包含
func Contains(s, substr string) bool
功能：字符串s中是否包含substr，返回bool值

```
fmt.Println(strings.Contains("seafood", "foo"))
```


#### 2.Join - 拼接字符串
func Join(a []string, sep string) string
功能：字符串链接，把slice a通过sep链接起来

```
s := []string{"foo", "bar", "baz"}
fmt.Println(strings.Join(s, ", "))
//运行结果:foo, bar, baz
```

#### 3.Index - 查字符串中特定字符位置
func Index(s, sep string) int
功能：在字符串s中查找sep所在的位置，返回位置值，找不到返回-1

```
fmt.Println(strings.Index("chicken", "ken"))
```

#### 4.Repeat - 重复打印字符串
func Repeat(s string, count int) string
功能：重复s字符串count次，最后返回重复的字符串

```
fmt.Println("ba" + strings.Repeat("na", 2))
//运行结果:banana
```

#### 5.Replace - 替换字符串中特定字符
func Replace(s, old, new string, n int) string
功能：在s字符串中，把old字符串替换为new字符串，n表示替换的次数，小于0表示全部替换

```
fmt.Println(strings.Replace("oink oink oink", "k", "ky", 2))
fmt.Println(strings.Replace("oink oink oink", "oink", "moo", -1))
//运行结果:
//oinky oinky oink
//moo moo moo
```

#### 6.Fields - 去除字符串中所有空格
func Fields(s string) []string
功能：去除s字符串的空格符，并且按照空格分割返回slice

```
fmt.Printf("Fields are: %q", strings.Fields("  foo bar  baz   "))
//运行结果:Fields are: ["foo" "bar" "baz"]
```

#### 7.Split - 以特定字符分割字符串
func Split(s, sep string) []string
功能：把s字符串按照sep分割，返回slice

```
    fmt.Printf("%q\n", strings.Split("a,b,c", ","))
```

#### 8.Trim - 去除字符串头尾空格
func Trim(s string, cutset string) string
功能：在s字符串的头部和尾部去除cutset指定的字符串

```
    fmt.Printf("[%q]", strings.Trim(" !!! Achtung !!! ", "! "))
    //运行结果:["Achtung"]
```

### 二、字符串转换

转换方法在包"strconv"中

#### 1.Append - 向数组追加各种类型变量
Append 系列函数将整数等转换为字符串后，添加到现有的字节数组中。

```
str := make([]byte, 0, 100)                 //make切片
str = strconv.AppendInt(str, 4567, 10)      //以10进制方式追加
str = strconv.AppendBool(str, false)        //以布尔类型方式追加
str = strconv.AppendQuote(str, "abcdefg")   //以string类型方式追加
str = strconv.AppendQuoteRune(str, '单')    //以byte类型方式追加
fmt.Println(string(str))                    //4567false"abcdefg"'单'
```

#### 2. Format
Format 系列函数把其他类型的转换为字符串。

```
a := strconv.FormatBool(false)          //布尔类型转字符串
b := strconv.FormatInt(1234, 10)        //把有符号整形转为字符串
c := strconv.FormatUint(12345, 10)      //把无符号整形转为字符串
d := strconv.Itoa(1023)                 //整形转字符串
fmt.Println(a, b, c, d)                 //false 1234 12345 1023
```

#### 3.Parse 
Parse 系列函数把字符串转换为其他类型。

```
func checkError(e error) {
    if e != nil {
        fmt.Println(e)
    }
}
func Parse_sth(){
    a, err := strconv.ParseBool("false")            //string转bool
    checkError(err)
    b, err := strconv.ParseFloat("123.23", 64)      //string转float64
    checkError(err)
    c, err := strconv.ParseInt("1234", 10, 64)      //string转int64
    checkError(err)
    d, err := strconv.ParseUint("12345", 10, 64)    //string转UINT64
    checkError(err)
    e, err := strconv.Atoi("1023")                  //string转整形
    checkError(err)
    fmt.Println(a, b, c, d, e) //false 123.23 1234 12345 1023
}
```

#### 4.正则表达式
包:"regexp"

注: 字符串处理我们可以使用strings包来进行搜索(Contains、Index)、替换(Replace)和解析(Split、Join)等操作，但是这些都是简单的字符串操作，他们的搜索都是大小写敏感，而且固定的字符串，如果需要匹配可变的那种就没办法实现
尽量使用strings包能解决问题，效率比正则好。

可参考: https://studygolang.com/pkgdoc 中所有使用regexp的方法



## Json

包:"encoding/json"

参考网站:
JSON官方网站：http://www.json.org/
在线检查格式：http://www.json.cn/

注:不能用含JSON名字定义.go文件

使用json.Marshal()函数可以对一组数据进行JSON格式的编码。 json.Marshal()函数的声明如下：
```
func Marshal(v interface{}) ([]byte, error)
```
格式化输出：
```
// MarshalIndent 很像 Marshal，只是用缩进对输出进行格式化
func MarshalIndent(v interface{}, prefix, indent string) ([]byte, error
```

### 一、生成Json
#### ①结构体生成json
```
//结构体变量首字母必须大写
type Json_struct struct {
	Company  string   `json:"company"` //struct tag 改变名字
	Subjects []string `json:"-"`       //不输出
	IsOk     bool
	Price    float64
}

func main() {
	j1 := Json_struct{"itcast", []string{"Go", "C++", "Python", "Test"}, true, 666.666}

	//方法一:JSON文件一行打印
	j_buf, err := json.Marshal(j1)
	if err != nil {
		fmt.Println("err: ", err)
	}

	//格式化打印 json.MarshalIndent(结构体变量, 首行字符, 占位字符)
	j_buf2, err := json.MarshalIndent(j1, "", " ")
	if err != nil {
		fmt.Println("err: ", err)
	}

	fmt.Println("j_buf: ", string(j_buf))   //需要把json.Marshal返回值转为string打印
	fmt.Println("j_buf2: ", string(j_buf2)) //需要把json.Marshal返回值转为string打印
}

```

#### ②字典map生成json

```
m1 := make(map[string]interface{}, 4)

m1["Company"] = "Itcast"
m1["Subjects"] = []string{"Go", "C++", "Python", "Test"}
m1["IsOk"] = true
m1["Price"] = 666.666

j_buf, err := json.MarshalIndent(m1, "", "	")
if err != nil {
	fmt.Println("err: ", err)
	return
}
fmt.Println("json:", string(j_buf))
```

### 二、解码JSON
使用json.Unmarshal()函数将JSON格式的文本解码为Go里面预期的数据结构。

json.Unmarshal()函数的原型：
```
    func Unmarshal(data []byte, v interface{}) error
```
该函数的第一个参数是输入，即JSON格式的文本（比特序列），第二个参数表示目标输出容器，用于存放解码后的值。

#### ①.结构体接收解码

```
	decode_json := []byte(`{
    "company": "itcast",
    "subjects": [
        "Go",
        "C++",
        "Python",
        "Test"
    ],
    "isok": true,
    "price": 666.666
}`)

	var uj_buf Json_struct

	err := json.Unmarshal(decode_json, &uj_buf)	//参数:json文件, 接收结构体
	if err != nil {
		fmt.Println("err: ", err)
	}
	fmt.Println("uj_buf: ", uj_buf)
```

#### 字段map接收解码

不及结构体输出简单,但解码方便.

```
//创建容量为4的[string]interface{}类型
m := make(map[string]interface{}, 4)        

err := json.Unmarshal(decode_json, &m)
if err != nil {
	fmt.Println("err: ", err)
	return
}

//需要使用类型断言逐个输出
for _, value := range m {
	switch m_type := value.(type) {
	case string:
		ret := m_type
		fmt.Printf("value:%+v, type:%T\n", value, ret)
	case bool:
		ret2 := m_type
		fmt.Printf("value:%+v, type:%T\n", value, ret2)
	case float64:
		ret3 := m_type
		fmt.Printf("value:%+v, type:%T\n", value, ret3)
	case []interface{}:
		ret4 := m_type
		fmt.Printf("value:%+v, type:%T\n", value, ret4)
	}
}
```


## 文件操作

### 一、创建文件

#### 新建文件两个方法
```
//根据提供的文件名创建新的文件，返回一个文件对象，默认权限是0666的文件，返回的文件对象是可读写的。
func Create(name string) (file *File, err Error)

//根据文件描述符创建相应的文件，返回一个文件对象
func NewFile(fd uintptr, name string) *File
```
#### 打开文件两个方法
```
//该方法打开一个名称为name的文件，但是是只读方式，内部实现其实调用了OpenFile。
func Open(name string) (file *File, err Error)

//打开名称为name的文件，flag是打开的方式，只读、读写等，perm是权限
func OpenFile(name string, flag int, perm uint32) (file *File, err Error)
```

### 二、写文件

```
//写入byte类型的信息到文件
func (file *File) Write(b []byte) (n int, err Error)

//在指定位置开始写入byte类型的信息
func (file *File) WriteAt(b []byte, off int64) (n int, err Error)

//写入string信息到文件
func (file *File) WriteString(s string) (ret int, err Error)
```

### 三、读文件

```
//读取数据到b中
func (file *File) Read(b []byte) (n int, err Error)

//从off开始读取数据到b中
func (file *File) ReadAt(b []byte, off int64) (n int, err Error)


```

### 四、实例
```
func main() {
    args := os.Args //获取用户输入的所有参数

    //如果用户没有输入,或参数个数不够,则调用该函数提示用户
    if args == nil || len(args) != 3 {
        fmt.Println("useage : xxx srcFile dstFile")
        return
    }

    srcPath := args[1] //获取输入的第一个参数
    dstPath := args[2] //获取输入的第二个参数
    fmt.Printf("srcPath = %s, dstPath = %s\n", srcPath, dstPath)

    if srcPath == dstPath {
        fmt.Println("源文件和目的文件名字不能相同")
        return
    }

    srcFile, err1 := os.Open(srcPath) //打开源文件
    if err1 != nil {
        fmt.Println(err1)
        return
    }

    dstFile, err2 := os.Create(dstPath) //创建目的文件
    if err2 != nil {
        fmt.Println(err2)
        return
    }

    buf := make([]byte, 1024) //切片缓冲区
    for {
        //从源文件读取内容，n为读取文件内容的长度
        n, err := srcFile.Read(buf)
        if err != nil && err != io.EOF {
            fmt.Println(err)
            break
        }

        if n == 0 {
            fmt.Println("文件处理完毕")
            break
        }

        //切片截取
        tmp := buf[:n]
        //把读取的内容写入到目的文件
        dstFile.Write(tmp)
    }

    //关闭文件
    srcFile.Close()
    dstFile.Close()
}

```

## 并发编程

并行与并发:

 - 并行(parallel)：指在同一时刻，有多条指令在多个处理器上同时执行。
 - 并发(concurrency)：指在同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，使得在宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的，只是把时间分成若干段，使多个进程快速交替的执行。
 - 主goroutine(即main函数)退出后，其它的工作goroutine也会自动退出
 
 
### 一、goroutine
包 "runtime"
一般方式
```
func co_fun() {
	fmt.Println("biubiu")   //在主协程后执行
}

func main() {
	go co_fun()
	fmt.Println("haha")     //主协程，需要最先执行
}
```
 


#### ①.runtime.Gosched()
用于让出CPU时间片，让出当前goroutine的执行权限，调度器安排其他等待的任务运行，并在下次某个时候从该位置恢复执行。

```
func co_fun() {
	fmt.Println("biubiu")
}

func main() {
	go co_fun()
	runtime.Gosched()       //有runtime.Gosched, 该协程(主协程)将在子协程后运行
	fmt.Println("haha")
}
```

#### ②.runtime.Goexit
```
func co_fun() {
	defer fmt.Println("tyj")
	runtime.Goexit()            //把defer执行完后,退出协程
	fmt.Println("biubiu")       //由于Goexit,无法执行
}

func main() {
	go co_fun()
	runtime.Gosched()
	fmt.Println("haha")
}
```


#### ③.GOMAXPROCS
 设置并行计算的CPU核数的最大值，并返回之前的值。

```
runtime.GOMAXPROCS(4) //设置4核运行
```


### 二、channel

 1. 由make创建
 2. 作参时为引用类型
 3. 零值为nil

#### ①.声明格式
 1. capacity = 0 时 channel 为无缓冲阻塞读写
 2. capacity > 0 时 channel 有缓冲、非阻塞的，直到写满capacity个元素才阻塞写入
```
make(chan Type) //等价于make(chan Type, 0)
make(chan Type, capacity)
```

#### ②.使用 <- 赋值
channel通过**操作符<-**来接收和发送数据，发送和接收数据语法：

    channel <- value      //发送value到channel
    <-channel             //接收并将其丢弃
    x := <-channel        //从channel中接收数据，并赋值给x
    x, ok := <-channel    //功能同上，同时检查通道是否已关闭或者是否为空

使用<-发送value到channel后,若value没有被接收,则当前goruntine不能往下执行
即:**value被接收前阻塞留在当前Goruntine中**

#### ③.无缓冲channel:
> - 无缓冲的通道（unbuffered channel）是指在接收前没有能力保存任何值的通道。

```
var ch_1 = make(chan string)    //全局变量使用var声明

func cl_printer(str string) {
	for _, data := range str { //string从第0个字符开始遍历
		fmt.Printf("%c", data)  //打印一个字符
		time.Sleep(time.Second) //一秒睡眠
	}
}

func co_fun() {
	tem := <-ch_1
	fmt.Printf("%s", tem)
	cl_printer("biubiu")
}

func co_fun2() {
	cl_printer("bababa")
	ch_1 <- " co_fun2 is done! "
}

func main() {
    //由于cl_printer中有时间延时,该两个子协程将轮番运行(不用channel时)
	go co_fun()    
	go co_fun2()
	for { //死循环
	}
}

```


#### ④有缓冲channel
> - 有缓冲的通道（buffered channel）是一种在被接收前能存储一个或者多个值的通道。
> - 当channel中有缓冲value时,可一边输入一边输入,保持value持续传递.

```
var ch = make(chan int, 3) //有缓存channel

func main() {
	go func() {
		for i := 0; i < 3; i++ {        //预先缓冲3个int型value
			ch <- i
			fmt.Println("ch input: ", i)
		}
	}()

	time.Sleep(time.Second * 3) //3s睡眠

	for i := 0; i < 3; i++ {
		ret := <-ch                     //连续接收3个缓冲中的int型value
		fmt.Println("ch output: ", ret)
	}
}
```


#### ⑤close channel

 1. channel不需要文件一样需要经常去关闭，当后续无数据需要发送了，或想显式的结束range循环之类的，才去关闭channel；
 2. 关闭channel后，无法向channel再发送数据(引发panic错误后导致接收立即返回零值)
 3. 关闭channel后，可以继续向channel接收数据； 对于nil channel，无论收发都会被阻塞。

接收缓冲channel方法一:
```
//check到channel关闭后退出接收的for循环
for {
	if data, ok := <-ch; ok {
		fmt.Println(data)
	} else {
		break
	}
}
```
接收缓冲channel方法二:
```
//range 遍历接收channel内所有data
for data := range ch {
	fmt.Println("ch output: ", data)
}
```


#### ⑥单向channel

 1. channel默认为双向 
 2. 可声明单项chanel

单向channel变量的声明:
```
var ch1 chan int       // ch1是一个正常的channel，不是单向的
var ch2 chan<- float64 // ch2是单向channel，只用于写float64数据
var ch3 <-chan int     // ch3是单向channel，只用于读取int数据
```

 1. chan<- 表示数据进入管道，要把数据写进管道，对于调用者就是输出。 
 2. <-chan 表示数据从管道出来，对于调用者就是得到管道的数据，当然就是输入。

双向channel 可隐式转换为单向队列，只收或只发，不能将单向channel转换为普通双向channel


### 三、定时器缓冲channel
包:"time"

#### ①.Timer - 定时器
 1. 创建定时器: time.time.NewTimer(seconds)
 2. 定时器阻塞: <- timer.C
 3. 定时器停止: timer.Stop()
 4. 定时器重置: timer.Reset()

应用实例:
```
//倒计时2秒
timer1 := time.NewTimer(time.Second * 1)
//倒计时结束前阻塞
t1 := <-timer1.C
fmt.Println("t1:", t1)

//倒计时阻塞1秒
t2 := <-time.After(time.Second * 2)
fmt.Println("t2:", t2)

t3 := time.NewTimer(time.Second)
go func() {
	<-t3.C
	fmt.Println("Timer 3 expired")
}()

//timer.Stop() 停止定时器
stop := t3.Stop() 
if stop {
	fmt.Println("Timer 3 stopped")
}

fmt.Println("before")
t4 := time.NewTimer(time.Second * 5) 

//timer.Reset(second) 重新设置时间
t4.Reset(time.Second * 1)   
<-t4.C
fmt.Println("after")

for {

}
```

#### ②.Ticker - 定时触发的计时器
以一个间隔(interval)往channel发送一个事件(当前时间)，而channel的接收者可以以固定的时间间隔从channel中读取事件。
```
for {
	//间隔1s
	timer1 := time.NewTicker(time.Second * 1)
	//倒计时结束前阻塞
	t1 := <-timer1.C
	fmt.Println("t1:", t1)
}
```

### 四、select - 监听channel

 1. select中每个case语句里必须是一个IO操作
 2. 在一个select中，Go语言会按顺序从头至尾评估每一个发送和接收的语句。
 3. 若其中的任意一语句可以继续执行(即没有被阻塞)，随机挑选执行。

```
select {
case <-chan1:
    // 如果chan1成功读到数据，则进行该case处理语句
case chan2 <- 1:
    // 如果成功向chan2写入数据，则进行该case处理语句
default:
    // 如果上面都没有成功，则进入default处理流程
}
```

如果没有任意一条语句可以执行(即所有的通道都被阻塞)，那么有两种可能的情况：
 1. 如果给出了default语句，那么就会执行default语句，同时程序的执行会从select语句后的语句中恢复。
 2. 如果没有default语句，那么select语句将被阻塞，直到至少有一个通信可以进行下去。

