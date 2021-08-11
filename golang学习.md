## 基础

go语言不需要使用逗号作为语句的结束，出发一行代码中包括多条语句

go run、build、install的区别：

- go run 编译并直接运行程序，需要在main包下执行go run 

- go build 用于测试编译包，主要检查是否会有编译错误，如果是一个可执行文件的源码（即是 main 包），就会在当前目录直接生成一个可执行文件。

- go install 的作用有两步：

    ​      第一步是编译导入的包文件，所有导入的包文件编译完才会编译主程序；

    　  第二步是将编译后生成的可执行文件放到 bin 目录下（$GOPATH/bin），编译后的包文件放到 pkg 目录下（$GOPATH/pkg）   	（$GOPATH为Go的工作目录）



### 可见性

如果一个名字是在函数内部定义，那么它就只在函数内部有效。

如果是在函数外部定义，那么将在当前包的所有文件中都可以访问。

名字的开头字母的大小写决定了名字在包外的可见性（变量和函数都一样）。如果一个名字是大写字母开头的（译注：必须是在函数外部定义的包级名字；包级函数名本身也是包级名字），那么它将是导出的，也就是说可以被外部的包访问，例如fmt包的Printf函数就是导出的，可以在fmt包外部访问。包本身的名字一般总是用小写字母。

### 变量

```Go
---单变量形式---
var 变量名字 类型 = 表达式 //完成表达式式
var 变量名字 = 表达式 //赋初值，编译器根据变量值自动推到变量类型
var 变量名字 类型 //不显式赋初值，编译器自动赋初值，不同变量类型的初值不同，所有必须要指明变量类型

---多变量形式---
var 变量1, 变量2, 变量3 string //这种形式只能同时声明多个同样类型的变量，不能声明不同的类型
var 变量1, 变量2, 变量3 = 1, "ab", 1.1 //如果省略每个变量的类型，将可以声明多个类型不同的变量（类型由初始化表达式推导）

---函数赋值---
var f, err = os.Open(name) //多个变量也可以由函数返回的多个值初始化
```

数值类型变量对应的零值是0，
布尔类型变量对应的零值是false，
字符串类型对应的零值是空字符串，
接口或引用类型（包括slice、指针、map、chan和函数）变量对应的零值是nil。
数组或结构体等聚合类型对应的零值是每个元素或字段都是对应该类型的零值。

### 短变量声明

在函数内部，有一种称为简短变量声明语句的形式可用于声明和初始化局部变量。它以“名字 := 表达式”或者”变量1, 变量2 := 值1,值2“形式声明变量，变量的类型根据表达式来自动推导。

**简短变量声明被广泛用于大部分的局部变量的声明和初始化。var形式的声明语句往往是用于需要显式指定变量类型的地方，或者因为变量稍后会被重新赋值而初始值无关紧要的地方。**

注意：**“:=”是一个变量声明语句，而“=”是一个变量赋值操作**。

go语言支持多重赋值

```Go
i, j := 12, 13 //声明局部变量i,j并赋初始值
i, j = j, i // 交换 i 和 j 的值
```

同var形式的变量声明语句一样，简短变量声明语句也可以用函数的返回值来声明和初始化变量，像下面的os.Open函数调用将返回两个值：

```Go
f, err := os.Open(name)
```

短变量声明在同时声明多个变量时，要求左侧至少要有一个全新的变量，否则无法编译通过

```Go
in, err := os.Open(infile) //in，err都是全新变量，编译通过
out, err := os.Create(outfile) //out是新变量，err是上一行代码声明的变量，此时声明新的变量out并赋值，err则为纯赋值操作
in, err := os.Create(outfile) //in，err都不是新的变量，不能编译通过，将“:=”改为“=”即可
```

### 指针

不管是普通定义的变量还是短变量，都可以进行指针访问，声明一个变量的指针方式如下：

```go
var pt *int //指针变量的声明

//指针的对比
var x, y int
fmt.Println(&x == &x, &x == &y, &x == nil) // "true false false"
```

下面是通过指针访问变量的一个例子：

```go
func main() {
   var a, b = 12, 12
   var pa, pb *int
   pa, pb = &a, &b
   fmt.Println(*pa + *pb)
   inc(pa, pb)
   fmt.Println(*pa + *pb)
}

func inc(pa *int, pb *int)  {
   *pa++
   *pb++
}
```

在Go语言中，函数之后返回指针在函数执行完毕之后依然有效。例如下面的代码，调用f函数时创建局部变量v，在局部变量地址被返回之后依然有效，因为指针p依然引用这个变量。**编译器会自动选择在栈上还是在堆上分配局部变量的存储空间，推测被返回指针的变量应该被保存到栈上**

```Go
var p = f()
func f() *int {
    v := 1
    return &v
}
```

### new和make

Go有两个数据结构创建函数：new和make。两者的区别在学习Go语言的初期是一个常见的混淆点。基本的区别是`new(T)`返回一个指针类型`*T`，。而`make(T, args)`返回一个普通的T。

另一个创建变量的方法是调用内建的new函数。表达式new(T)将创建一个T类型的匿名变量，初始化为T类型的零值，然后返回变量地址，返回的指针类型为`*T`。

```Go
p := new(int)   // p, *int 类型, 指向匿名的 int 变量
fmt.Println(*p) // "0"
*p = 2          // 设置 int 匿名变量的值为 2
fmt.Println(*p) // "2"
```

一下代码具有相同的行为

```go
func main() {
   var p1 = new(int32) //得到一个int32指针的地址
   var a int32
   var p2 = &a //得到一个int32指针的地址
}
```

**new函数使用通常相对比较少**，因为对于结构体来说，直接用字面量语法创建新变量的方法会更灵活

### 包初始化

包的初始化首先是解决包级变量的依赖顺序，然后按照包级变量声明出现的顺序依次初始化：

```Go
var a = b + c // a 第三个初始化, 为 3
var b = f()   // b 第二个初始化, 为 2, 通过调用 f (依赖c)
var c = 1     // c 第一个初始化, 为 1

func f() int { return c + 1 }
```

如果包中含有多个.go源文件，它们将按照发给编译器的顺序进行初始化，Go语言的构建工具首先会将.go文件根据文件名排序，然后依次调用编译器编译。

对于在包级别声明的变量，如果有初始化表达式则用表达式初始化，还有一些没有初始化表达式的，例如某些表格数据初始化并不是一个简单的赋值过程。在这种情况下，我们可以用一个特殊的init初始化函数来简化初始化工作。每个文件都可以包含多个init初始化函数

```Go
func init() { /* ... */ }
```

这样的init初始化函数除了不能被调用或引用外，其他行为和普通函数类似。在每个文件中的init初始化函数，在程序开始执行时按照它们声明的顺序被自动调用。

每个包在解决依赖的前提下，以导入声明的顺序初始化，每个包只会被初始化一次。因此，如果一个p包导入了q包，那么在p包初始化的时候可以认为q包必然已经初始化过了。初始化工作是自下而上进行的，main包最后被初始化。以这种方式，可以确保在main函数执行之前，所有依赖的包都已经完成初始化工作了。



## 数据类型

### 整型

Go语言同时提供了有符号和无符号类型的整数运算。这里有int8、int16、int32和int64四种截然不同大小的有符号整数类型，分别对应8、16、32、64bit大小的有符号整数，与此对应的是uint8、uint16、uint32和uint64四种无符号整数类型。

这里还有两种一般对应特定CPU平台机器字大小的有符号和无符号整数int和uint；其中int是应用最广泛的数值类型。这两种类型都有同样的大小，32或64bit，但是我们不能对此做任何的假设；因为不同的编译器即使在相同的硬件平台上可能产生不同的大小。

### 浮点数

浮点数有float32与float64



**浮点数与浮点数，整数与整数，浮点数与整数之间的数值运算都需要做显式的类型转换，否则编译报错**

### 字符串

**一个字符串是一个不可改变的字节序列**。字符串可以包含任意的数据，包括byte值0，但是通常是用来包含人类可读的文本。文本字符串通常被解释为采用UTF8编码的Unicode码点（rune）序列，我们稍后会详细讨论这个问题。



标准库中有四个包对字符串处理尤为重要：bytes、strings、strconv和unicode包。

1. strings包提供了许多如字符串的查询、替换、比较、截断、拆分和合并等功能。

2. bytes包也提供了很多类似功能的函数，但是针对和字符串有着相同结构的[]byte类型。因为字符串是只读的，因此逐步构建字符串会导致很多分配和复制。在这种情况下，使用bytes.Buffer类型将会更有效，稍后我们将展示。

3. strconv包提供了布尔型、整型数、浮点数和对应字符串的相互转换，还提供了双引号转义相关的转换。

4. unicode包提供了IsDigit、IsLetter、IsUpper和IsLower等类似功能，它们用于给字符分类。每个函数有一个单一的rune类型的参数，然后返回一个布尔值。而像ToUpper和ToLower之类的转换函数将用于rune字符的大小写转换。



### 常量

常量使用“**const**”修饰，常量表达式的值在编译期计算，而不是在运行期。每种常量的潜在类型都是基础类型：boolean、string或数字。

一个常量的声明也可以包含一个类型和一个值，但是如果没有显式指明类型，那么将从右边的表达式推断类型。

```go
const a = 10
const b = "afsdaf"
const pi float64 = 3.14159
```

批量定义常量，同一行使用分号分开，不同行直接写

```go
//多个常量定义在同一行
const (
   e  = 2.71828182; pi = 3.1415926535
)
//多个常量在不同的行
const (
    e  = 2.71828182845904523536028747135266249775724709369995957496696763
    pi = 3.14159265358979323846264338327950288419716939937510582097494459
)
```

如果是批量声明的常量，除了第一个外其它的常量右边的初始化表达式都可以省略，如果省略初始化表达式则表示使用前面常量的初始化表达式写法，对应的常量类型也一样的。例如：

```Go
const (
    a = 1
    b
    c = 2
    d
)
//或者
const (
	a = 1; b; c = 2; d
)

fmt.Println(a, b, c, d) // "1 1 2 2"
```

#### itoa常量生成器

常量声明可以使用iota常量生成器初始化，它用于生成一组以相似规则初始化的常量，但是不用每行都写一遍初始化表达式。在一个const声明语句中，在第一个声明的常量所在的行，iota将会被置为0，然后在每一个有常量声明的行加一。

```Go
const (
    zero = iota
    one
    two
    three
    four
)
fmt.Println(zero, one, two, three, four) // "0 1 2 3 4"
```

#### 无类型常量

虽然一个常量可以有任意一个确定的基础类型，但是许多常量并没有一个明确的基础类型。编译器为这些没有明确基础类型的数字常量提供比基础类型更高精度的算术运算，通过延迟明确常量的具体类型，无类型的常量不仅可以提供更高的运算精度，而且可以直接用于更多的表达式而不需要显式的类型转换。

有六种未明确类型的常量类型，分别是无类型的布尔型、无类型的整数、无类型的字符、无类型的浮点数、无类型的复数、无类型的字符串。

一个例子，math.Pi无类型的浮点数常量，可以直接用于任意需要浮点数或复数的地方：

```Go
var x float32 = math.Pi
var y float64 = math.Pi
var z complex128 = math.Pi
```

如果math.Pi被确定为特定类型，比如float64，那么结果精度可能会不一样，同时对于需要float32或complex128类型值的地方则会强制需要一个明确的类型转换：

```Go
const Pi64 float64 = math.Pi

var x float32 = float32(Pi64)
var y float64 = Pi64
var z complex128 = complex128(Pi64)
```

前面说过除法运算符/会根据操作数的类型生成对应类型的结果。因此，不同写法的常量除法表达式可能对应不同的结果：

```Go
var f float64 = 212
fmt.Println((f - 32) * 5 / 9)     // "100"; (f - 32) * 5 is a float64
fmt.Println(5 / 9 * (f - 32))     // "0";   5/9 is an untyped integer, 0
fmt.Println(5.0 / 9.0 * (f - 32)) // "100"; 5.0/9.0 is an untyped float
```

只有常量可以是无类型的。当一个无类型的常量被赋值给一个变量的时候，就像下面的第一行语句，或者出现在有明确类型的变量声明的右边，如下面的其余三行语句，无类型的常量将会被隐式转换为对应的类型，如果转换合法的话。

```Go
f = 2                  // untyped integer -> float64
f = 1e123              // untyped floating-point -> float64
f = 'a'                // untyped rune -> float64
```

上面的语句相当于:

```Go
f = float64(2)
f = float64(1e123)
f = float64('a')
```



类型推导中：**go将整数默认使用int类型，该类型的字节数平台相关；浮点数统一使用float64，复数统一使用complex128，内存大小是确定的**



## 复合类型

### 数组

**数组的长度是数组类型的一个组成部分，因此[3]int和[4]int是两种不同的数组类型。数组的长度必须是常量表达式，因为数组的长度需要在编译阶段确定。**

数组定义与初始化

```Go
//定义一个长度为3的数组
var a [3]int             // array of 3 integers
//定义一个长度为0的数组
var a [0]int

//注意在让编译器自动推导元素个数时，中括号中必须填入“...”，否则被视为切片
var a = [...]int{} //定义一个长度为0的数组，自动推导

//定义一个数组并初始化，数组长度根据元素个数确定
var ary = [...]int{1,2,3,4} //也可以写为：var ary = [...]int{1,2,3,4}
//或者
var ary [4]int = [...]int{1,2,3,4}
//或者
var ary [4]int = [4]int{1,2,3,4} //这种形式前后中括号中的长度值要么都为空，否则要求前后一行

//定义一个长度为100的数组，只初始化前3个
var ary = [100]int{1,2,3}
```

按具体下标对数据进行定义和初始化。在这种形式的数组字面值形式中，初始化索引的顺序是无关紧要的，而且没用到的索引可以省略，和前面提到的规则一样，未指定初始值的元素将用零值初始化。

```go
//显式地按下标初始化数组元素
var a = []string{0: "zero", 2: "two", 1:"one", 3: "three"}


var b = []string{99: "nighnigh"} //初始化使用了下标99， 长度变为100：len(b) is 100
```



### 数组定义的错误写法

go中数组在定义、使用和传参过程中，要求维度值严格匹配，否则编译失败，灵活性很低。

```go
/////////错误写法/////////
var a [...]int //这种写法无法通过编译，"..."表示自动推导，但是没有指明初始化元素
var a [...]int = [...]int{1,2,3,4}// 这样的写法也是错误的，等号右边推导出来是长度为4的数组，与等号左边无法匹配
var a []int = [...]int{1,2,3,4} //错误同上
/////////////////////////
```



### 数组比较

如果一个数组的元素类型是可以相互比较的，那么数组类型也是可以相互比较的，这时候我们可以直接通过==比较运算符来比较两个数组，只有当两个数组的所有元素都是相等的时候数组才是相等的。不相等比较运算符!=遵循同样的规则。

```Go
a := [2]int{1, 2}
b := [...]int{1, 2}
c := [2]int{1, 3}
fmt.Println(a == b, a == c, b == c) // "true false false"
d := [3]int{1, 2}
fmt.Println(a == d) // compile error: cannot compare [2]int == [3]int
```



### 二维数组

数据的定义与初始化

```go
var ary [10][10]int //定义10*10的数组
var ary [2][2]int = [2][2]int{{1, 2}, {3, 4}} //定义并初始化2*2的数组
var ary = [2][2]int{{1, 2}, {3, 4}} //定义并初始化2*2的数组，不在变量名后显式指定类型

//不指定维度，让编译器根据元素个数自动推断
var ary [2][2]int = [...][2]int{{1,2},{3,4}}
var ary = [...][2]int{{1,2},{3,4}}

//编译器只能推导一个维度，可以是第一个维度也可以是第二个维度，所以这样的写法是错误
var ary [2][2]int = [...][...]int{{1,2},{3,4}}
//但是以下写法可以通过编译
var ary  = [2][...]int{{1,2},{3,4}} //相当于切片[2][]int
var ary  = [...][2]int{{1,2},{3,4}} //相当于切片[][2]int

//如果改为以下方式，可以通过编译，但实际的形式为[2][]
var ary = [...][...]int{{1,2},{3,4}} //等价于[2][]int, 数组存放了两个切片
var ary = [...][...]int{{1},{1,2}, {1,2,3,4}} //等价于[3][]int, 一维数组存放了三个切片
```

### 数组指针与函数传递

当调用一个函数的时候，函数的每个调用参数将会被赋值给函数内部的参数变量，所以函数参数变量接收的是一个复制的副本，并不是原始调用的变量。因为函数参数传递的机制导致传递大的数组类型将是低效的，并且对数组参数的任何的修改都是发生在复制的数组上，并不能直接修改调用时原始的数组变量。

数组指针定义

```go
var ary = []int{1,2,3}
var aryptr *[]int = &ary
```

以下函数传入一个数组，并将传入的数组置为零

```Go
func zero(ptr *[32]byte) {
    for i := range ptr { //对数组指针的访问可以不用符号*
        ptr[i] = 0
    }
}
//或者写为
func zero(ptr *[32]byte) {
    *ptr = [32]byte{} //但是修改指针指向的内容时，必须使用符号*
}
```

**当数组指针传入函数时，函数对数组数据的访问可以直接把指针本身当成数组名操作，但是对数组内容的修改必须使用*修饰符拿到数组本身**



### 切片

Slice（切片）代表变长的序列，序列中每个元素都有相同的类型。一个slice类型一般写作[]T，其中T代表slice中元素的类型；slice的语法和数组很像，只是没有固定长度而已。

#### 切片的定义与初始化

切片的定义与初始化很相似，最大的区别是切片不需要指定长度。

```go
var months = [...]string{1: "January", /* ... */, 12: "December"}
var s1 []int  等价于 var s1 = []int{} //定义一个切片
var s2 = []int{1,2,3} //定义一个切片，且初始化3个元素

//使用make来创建切片
//使用make创建一个切片，当前的长度为0
var s3 []int = make([]int,0)
s4 := make([]int,0,0)
```

使用make来创建

```go
//make([]T, len, cap)
var s1 = make([]int, 10)    //创建一个slice，容量为10
var s2 = make([]int, 3, 10) //创建一个slice，容量为10，填充前3个元素为0
var s3 = make([]int, 6, 10) //创建一个slice，容量为10，填充前6个元素为0

fmt.Println(s1)
fmt.Println(s2)
fmt.Println(s3)
fmt.Println(len(s1), cap(s1))
fmt.Println(len(s2), cap(s2))
fmt.Println(len(s3), cap(s3))

//代码执行输出内容如下
[0 0 0 0 0 0 0 0 0 0]
[0 0 0]
[0 0 0 0 0 0]
10 10
3 10
6 10
```

#### 切片的访问与操作

对切片的append操作将返回全新的，独立与原有的切片。**切片只能通过append向后扩展，不能向前扩展**

```go
//使用append追加元素,可以追加一个或者多个，函数末尾是个可变长参数
var s1 = []int{0,0}
s2 := append(s1, 1)
s3 := append(s1, 1, 2)
s4 := append(s1, 1, 2, 3)
s1[0] = 111
fmt.Println(s1) //[111, 0]
fmt.Println(s2) //[0, 0, 1]
```



对切片进行切割之后得到的各个子切片是引用，以下代码展示的对来自同一个切片的任何一个引用进行修改都会影响该数据的所有引用

![img](assets/ch4-01.png)

下面Q2和summer两个slice都包含了六月份
```Go
months := [...]string{1: "January", /* ... */, 12: "December"}
Q2 := months[4:7]
summer := months[6:9]
fmt.Println(Q2)     // ["April" "May" "June"]
fmt.Println(summer) // ["June" "July" "August"]
```

如果切片操作超出cap(s)的上限将导致一个panic异常，但是超出len(s)则是意味着扩展了slice，因为新slice的长度会变大：

```Go
fmt.Println(summer[:20]) // panic: out of range

endlessSummer := summer[:5] // extend a slice (within capacity)
fmt.Println(endlessSummer)  // "[June July August September October]"
```




```go
var s1 = []int{1, 2, 3, 4, 5, 6, 7, 8, 9}
var s2 = s1[0:4]
var s3 = s2[0:2]
s1[0] = 1000
fmt.Println(s1, s2, s3)
s3[1] = 1111
fmt.Println(s1, s2, s3)

//程序输出
[1000 2 3 4 5 6 7 8 9]    [1000 2 3 4]    [1000 2]    //修改s1，导致了全部引用值发生变化
[1000 1111 3 4 5 6 7 8 9] [1000 1111 3 4] [1000 1111] //修改s3，同样导致了全部引用值发生变化

```

切割出来的slice，长度（len）就是可见数据的长度，容量（cap）是从起点到原始数据的终点。

```go
var s1 = []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
var s2 = s1[2:6]
fmt.Println(len(s1), cap(s1)) //output: 10 10
fmt.Println(len(s2), cap(s2)) //output: 4 8
```



另外，字符串的切片操作和[]byte字节类型切片的切片操作是类似的。都写作x[m:n]，并且都是返回一个原始字节序列的子序列，底层都是共享之前的底层数组，因此这种操作都是常量时间复杂度。

**x[m:n]切片操作对于字符串则生成一个新字符串，如果x是[]byte的话则生成一个新的[]byte。**



#### 切片的copy

copy函数所产生的效果类似于C语言中的memcpy函数，`copy(dest, src)`，复制的长度为`min(len(dest), len(src))`，dest和src可能是同一个数组中不同下标的位置，也可能是两个完全不同的数组。

如果要将一个切片独立地、完整的复制一份，则需要使用如下形式的代码：

```go
var src = []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
var desc = make([]int, len(src)) //必须创建一个新的切片，且保证len(dest) == len(src)
copy(desc, src)

//使用以下方式是不能实现数组复制的
var src = []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
var desc []int //没有实际的存储空间，因此len(desc) == 0, min(len(dest), len(src)) == 0, 复制0个数据
copy(desc, src)
```



### 数组与切片的差别

要注意的是slice类型的变量s和数组类型的变量a的初始化语法的差异。slice和数组的字面值语法很类似，它们都是用花括弧包含一系列的初始化元素，但是对于**slice并没有指明序列的长度**。这会隐式地创建一个合适大小的数组，然后slice的指针指向底层的数组。就像数组字面值一样，slice的字面值也可以按顺序指定初始化值序列，或者是通过索引和元素值指定，或者用两种风格的混合语法初始化。

和数组不同的是，**slice之间不能比较，因此我们不能使用==操作符来判断两个slice是否含有全部相等元素**。不过标准库提供了高度优化的bytes.Equal函数来判断两个字节型slice是否相等（[]byte），但是对于其他类型的slice，我们必须自己展开每个元素进行比较。

**一个零值的slice等于nil。一个nil值的slice并没有底层数组。一个nil值的slice的长度和容量都是0**。因此，以下表达式永远成立：

```go
var s []int
fmt.Println(s == nil) // always true
```

**如果你需要测试一个slice是否是空的，使用len(s) == 0来判断，而不应该用s == nil来判断**。除了和nil相等比较外，一个nil值的slice的行为和其它任意0长度的slice一样；



### map

map的底层实现为哈希表，它是一个无序的key/value对的集合，其中所有的key都是不同的，且要求 key必须是支持==比较运算符的数据类型，所以map可以通过测试key是否相等来判断是否已经存在。虽然浮点数类型也是支持相等运算符比较的，但是不建议使用浮点作为key，因为最坏的情况是可能出现的NaN和任何浮点数都不相等。map对于V对应的value数据类型则没有任何的限制。

#### map的创建方式

```Go
numbers := make(map[int]string) // mapping from int to string
numbers := map[int]string{} //末尾必须要有{}，以设置为一个空哈希表

//一下代码声明了一个map类型，但没有实际容量
var numbers map[int]string
numbers==nil  //true
numbers[12] //对值为nil的map可以使用key访问到默认值，不会报错
numbers[1]="one" //但是直接对nil的map赋值会报错
```

我们也可以用map字面值的语法创建map，同时还可以指定一些最初的key/value：

```Go
numbers := map[int]string {
    0: "zero",
    1: "one",
}

//以上相当于
numbers := make(map[int]string)
numbers[0] = "zero"
numbers[1] = "one"

创建空的map的表达式是
numbers = map[string]int{}
numbers := make(map[int]string)
```



#### map的访问修改、删除

对map中的元素的访问与修改与数组一样，将key作为map的下标即可（操作方法与Python/C++一样）：

```Go
ages["alice"] = 32
fmt.Println(ages["alice"]) // "32"
```

使用内置的delete函数可以删除元素：

```Go
delete(ages, "alice") // remove element ages["alice"]
```

所有这些操作是安全的，即使这些元素不在map中也没有关系；如果一个查找失败将返回value类型对应的零值，例如，即使map中不存在“bob”下面的代码也可以正常工作，因为ages["bob"]失败时将返回0。

```Go
ages["bob"] = ages["bob"] + 1 // happy birthday!
```

而且`x += y`和`x++`等简短赋值语法也可以用在map上，所以上面的代码可以改写成

```Go
ages["bob"] += 1
```

更简单的写法

```Go
ages["bob"]++
```

但是map中的元素并不是一个变量，因此我们不能对map的元素进行取址操作：

```Go
_ = &ages["bob"] // compile error: cannot take address of map element
```

禁止对map元素取址的原因是map可能随着元素数量的增长而重新分配更大的内存空间，从而可能导致之前的地址无效。

#### map的迭代

由于map底层的实现为hash，所以其迭代的顺序是不确定的。

```go
var nums = map[string]int{}
nums["one"] = 1
nums["two"] = 2
for k, v := range nums {
    fmt.Println(k, v)
}
```

如果要按key的顺序遍历map，这需要对map进行两次方法，第一次拿到map的所有键，对键排序后再依次访问值。

```go
var nums = map[string]int{}
var keySet []string
sort.Strings(keySet)
for k := range nums {
    keySet = append(keySet, k)
}

for _, k := range keySet {
    fmt.Println(nums[k])
}
```



#### map判断元素是否存在

对map的访问可以返回多值，其中一个值表示访问的目标key是不是存在。

```go
age = student["key"]        //单变量仅返回存在的值或者默认值0/""
age, exist = student["key"] //两个变量时，第一个变量为value，第二个变量为bool值，标识变量是否存在

//完整的判断方式如下：
if value, ok = mmap["key"]; ok { //如果key存在，则访问key对应的value
    //对value进行操作
} else {
    //其他处理
}
```



### 结构体

go的结构体与C语言的类型，区别在于多了type关键词修饰，另外类型在变量名之前。**如果结构体成员名字是以大写字母开头的，那么该成员就是导出的。未导出的变量仅限包内访问**

下面是一个结构体的定义：

```Go
type Employee struct { //定义一个结构体
    ID        int
    Name      string
    Address   string
}

var alice Employee //声明一个结构体变量
```

#### 结构体的初始化

**结构体中的所有成员变量都会被初始化为默认初始值**。成员变量的初始化方法有三种：

定义结构体，单独为结构体中的每个变量赋值

```go
var alice Employee;
alice.Name = "Alice"
alice.ID = 123
alice.Address = "china"
```

直接填充每个字段

```go
var alice = Employee {
   12,
   "Alice",
   "China",
}
```

通过kv的方式填充部分字段，为填充的字段使用默认初始值

```go
var Alice = Employee{
   ID:      10,
   Name:    "Alice",
   Address: "China",
}
```





结构体变量的访问同样通过点`.`操作。**并且在go语言中，无论是结构体本身还是结构体指针，对数据的访问都通过点`.`实现**

```go
var age = alice.age
alice.ID = 123

//结构体的指针
var alicePtr *Employee = &alice
alicePtr.Name = "alice";
fmt.Print(alicePtr.Name, alicePtr.ID, "\n") //Alice 123
fmt.Print(alice.Name, alice.ID, "\n") //Alice 123
```



**如果通过函数返回一个结构体，函数外部要对该结构体中的成员变量进行修改的话，需要返回该结构体的指针才能实现，否则返回的结构体对象是只读的常量。**



如果结构体没有任何成员的话就是空结构体，写作struct{}。它的大小为0，也不包含任何信息，但是有时候依然是有价值的。有些Go语言程序员用map来模拟set数据结构时，用它来代替map中布尔类型的value，只是强调key的重要性，但是因为节约的空间有限，而且语法比较复杂，所以我们通常会避免这样的用法。

```Go
seen := make(map[string]struct{}) // set of strings
// ...
if _, ok := seen[s]; !ok {
    seen[s] = struct{}{}
    // ...first time seeing s...
}
```



#### 结构体的比较

如果结构体的全部成员都是可以比较的，那么结构体也是可以比较的，那样的话两个结构体将可以使用`==`或`!=`运算符进行比较。相等比较运算符==将比较两个结构体的每个成员，因此下面两个比较的表达式是等价的：

```Go
type Point struct{ X, Y int }

p := Point{1, 2}
q := Point{2, 1}
fmt.Println(p.X == q.X && p.Y == q.Y) // "false"
fmt.Println(p == q)                   // "false"
```

可比较的结构体类型和其他可比较的类型一样，可以用于map的key类型。

```Go
type address struct {
    hostname string
    port     int
}

hits := make(map[address]int)
hits[address{"golang.org", 443}]++
```



#### 结构体嵌入和匿名成员

go语言中没有类，没有继承的概念，如果要复用某些对象，只能通过组合的方式。

以下代码展示了人类与学生类的组合使用关系

```go
type Person struct {
    name string
    age int
}

type Student struct {
    personal Person
    score int
}

var stu Student
//在访问Student对象时，使用多层嵌套访问
var name = stu.personal.name
var score = stu.score
```

通过上述组合的方式，如果A组合到B，B组合到C，C组合到D，那么通过D访问A中的属性时就要以`D.C.B.A.filed`的方式去访问。代码冗余繁琐，因此go语言提供的匿名的方式访问组合的对象。具体而言就是丢弃组合对象的变量名，只给类型。示例代码如下：

```go
type Person struct {
    name string
    age int
}

type Student struct {
    Person
    score int
}

var stu Student
//在访问Student对象时，使用多层嵌套访问
var name = stu.name //这里没有使用Person变量名做中间过渡，等价于stu.personal.name
var score = stu.score
```

但是变量匿名之后，不能通过前文的方式对变量进行初始化，需要通过一下方式：

```go
var stu = Student{{"123", 12}, 12} //错误的初始化方式

//正确的初始化方式1
var stu Student
stu.name = "af"
stu.age = 12
stu.score = 100

//正确的初始化方式2
var stu = Student{
   Person{"123", 12}, 
   12,
}

//正确的初始化方式3
var stu = Student{
    Person: Person{"123", 12},
    score: 12,
}
```

因为匿名成员也有一个隐式的名字，因此不能同时包含两个类型相同的匿名成员，这会导致名字冲突。同时，因为成员的名字是由其类型隐式地决定的，所以匿名成员也有可见性的规则约束。**如果要限制Person类的作用域在包内，需要结构体定义处的名称Person改为小写person**



### 函数

在Go语言中，所有的函数参数都是值拷贝传入的，函数参数将不再是函数调用时的原始变量。

#### 函数定义

函数声明包括函数名、形式参数列表、返回值列表（可省略）以及函数体。其中返回值也可以是多值。**如果只有一个返回参数，可以不用括号**。

返回值列表中只给定类型

```Go
func name(var1 type1, var2 type2 /*...*/) (type1, type2 /*...*/) {
    body
    return v1, v2 /*...*/
}
```

返回值中同时给定变量名与其变量类型，这种情况下，返回值列表中的变量名可以直接在函数中使用，同时，可以不用显示的返回每个变量，只需要一个空`return`语句即可。

```go
//返回值列表中直接给定变量名字
func name(var1 type1, var2 type2 /*...*/) (var1 type1, var2 type2 /*...*/) {
    body
    return v1, v2 /*...*/
}
```

也可以将多个类型相同的变量写在一起，只用一个类型符做声明：

```Go
func HourMinSec(t time.Time) (hour int, minute int, second int)
//等价于下面的代码
func HourMinSec(t time.Time) (hour, minute, second int)
```

调用多返回值函数时，返回给调用者的是一组值，调用者必须显式的将这些值分配给变量，如果某个值不被使用，则使用`_`标识即可。

```Go
links, err := findLinks(url)
```





下面代码是一个将秒数转为时分秒的函数，传入一个值，返回多个值

```go
func HourMinSec(time int) (hour, minute, second int) {
   hour = time / 3600
   minute = (time / 60) % 60
   second = time % 60
   //return hour, minute, second
   return //这种方式不宜过度使用，尽量给定完整的返回参数
}

func HourMinSec(time int) (int, int, int) {
   var hour = time / 3600
   var minute = (time / 60) % 60
   var second = time % 60
   return hour, minute, second
}
```



#### 错误处理

对于那些将运行失败看作是预期结果的函数，它们会返回一个额外的返回值，通常是最后一个，来传递错误信息。如果导致失败的原因只有一个，额外的返回值可以是一个布尔值，通常被命名为ok。通常，导致失败的原因不止一种，因此，额外的返回值不再是简单的布尔类型，而是error类型，内置的error是接口类型，error类型可能是nil或者non-nil。nil意味着函数运行成功，non-nil表示失败。对于non-nil的error类型，我们可以通过调用error的Error函数或者输出函数获得字符串类型的错误信息。

通常，当函数返回non-nil的error时，其他的返回值是未定义的（undefined），这些未定义的返回值应该被忽略。然而，有少部分函数在发生错误时，仍然会返回一些有用的返回值。比如，当读取文件发生错误时，Read函数会返回可以读取的字节数以及错误信息。对于这种情况，正确的处理方式应该是先处理这些不完整的数据



#### 匿名函数

```go
//定义一个匿名函数
var square = func(x int) int { return x * x }
var res = square(14) //196

//定义并执行匿名函数
var s = func(x int) int { return x * x }(4)
print(s) //16
```

通过匿名函数这种方式可以访问完整的词法环境（lexical environment），这意味着在函数中定义的内部函数可以引用该函数的变量，如下例所示：

```Go
// squares返回一个匿名函数。
// 该匿名函数每次被调用时都会返回下一个数的平方。
func squares() func() int {
    var x int
    return func() int {
        x++
        return x * x
    }
}
func main() {
    f := squares()
    fmt.Println(f()) // "1"
    fmt.Println(f()) // "4"
    fmt.Println(f()) // "9"
    fmt.Println(f()) // "16"
}
```

当匿名函数需要被递归调用时，我们必须首先声明一个变量（在上面的例子中，我们首先声明了 visitAll），再将匿名函数赋值给这个变量。如果不分成两步，函数字面量无法与visitAll绑定，我们也无法递归调用该匿名函数。

```Go
var visitAll func(items []string)
visitAll = func(items []string) {
    visitAll(m[item])
}
```

### 捕获迭代变量

本节，将介绍Go词法作用域的一个陷阱。请务必仔细的阅读，弄清楚发生问题的原因。即使是经验丰富的程序员也会在这个问题上犯错误。

考虑这样一个问题：你被要求首先创建一些目录，再将目录删除。在下面的例子中我们用函数值来完成删除操作。下面的示例代码需要引入os包。为了使代码简单，我们忽略了所有的异常处理。

```Go
var rmdirs []func()
for _, d := range tempDirs() {
    dir := d // NOTE: necessary!
    os.MkdirAll(dir, 0755) // creates parent directories too
    rmdirs = append(rmdirs, func() {
        os.RemoveAll(dir)
    })
}
// ...do some work…
for _, rmdir := range rmdirs {
    rmdir() // clean up
}
```

你可能会感到困惑，为什么要在循环体中用循环变量d赋值一个新的局部变量，而不是像下面的代码一样直接使用循环变量dir。需要注意，下面的代码是错误的。

```go
var rmdirs []func()
for _, dir := range tempDirs() {
    os.MkdirAll(dir, 0755)
    rmdirs = append(rmdirs, func() {
        os.RemoveAll(dir) // NOTE: incorrect!
    })
}
```

问题的原因在于循环变量的作用域。在上面的程序中，for循环语句引入了新的词法块，循环变量dir在这个词法块中被声明。在该循环中生成的所有函数值都共享相同的循环变量。需要注意，函数值中记录的是循环变量的内存地址，而不是循环变量某一时刻的值。以dir为例，后续的迭代会不断更新dir的值，当删除操作执行时，for循环已完成，dir中存储的值等于最后一次迭代的值。这意味着，每次对os.RemoveAll的调用删除的都是相同的目录。

通常，为了解决这个问题，我们会引入一个与循环变量同名的局部变量，作为循环变量的副本。比如下面的变量dir，虽然这看起来很奇怪，但却很有用。

```Go
for _, dir := range tempDirs() {
    dir := dir // declares inner dir, initialized to outer dir
    // ...
}
```

这个问题不仅存在基于range的循环，在下面的例子中，对循环变量i的使用也存在同样的问题：

```Go
var rmdirs []func()
dirs := tempDirs()
for i := 0; i < len(dirs); i++ {
    os.MkdirAll(dirs[i], 0755) // OK
    rmdirs = append(rmdirs, func() {
        os.RemoveAll(dirs[i]) // NOTE: incorrect!
    })
}
```

如果你使用go语句（第八章）或者defer语句（5.8节）会经常遇到此类问题。这不是go或defer本身导致的，而是因为它们都会等待循环结束后，再执行函数值。



#### 可变参数

可变参函数定义

```go
//可变参数使用写在类型前面的...表示
func sum(vals ...int) int { //在函数体中，vals被看作是类型为[] int的切片
    /*do something*/
}

//如果既有可变长参数又有固定参数，那么可变参数应该写在参数列表的末尾才能确定正确性
func foo(str string, vals ...int) string {
   /*do something*/
}
```

可变参数函数调用
```Go
//调用者隐式的创建一个数组，并将原始参数复制到数组中，再把数组的一个切片作为参数传给被调用函数。
var s = sum(1,2,3,4,5) 

//如果原始参数已经是切片类型，只需在最后一个参数后加上省略符。
values := []int{1, 2, 3, 4}
var s = sum(values...) 

//但是不能同时传入参数列表和切片，下面的写法无法通过编译
var s = sum(1,2,3,4, values...) 
```



#### 函数作为变量

函数值可以作为一个变量，用于赋值或者传参。**函数变量可以与nil做比较，如果函数变量为nil表示该变量没有指向实际的函数，但是但是函数变量之间不可相互比较，也不能用函数变量值作为map的key。**

```go
f := HourMinSec
var f = HourMinSec
var f func(int) (int, int, int)  //f == nil is true
```





### defer修饰词

defer类似于java中的finally，或者是为目标函数绑定一个事件函数。不论包含defer语句的函数是通过return正常结束，还是由于panic导致的异常结束，都会调用defer绑定的函数。使用的时候只需要在调用普通函数或方法前加上关键字defer即可，一个函数中声明多条defer语句，它们的执行顺序与声明顺序相反。

defer语句经常被用于处理成对的操作，如打开、关闭、连接、断开连接、加锁、释放锁。通过defer机制，不论函数逻辑多复杂，都能保证在任何执行路径下，资源被释放。释放资源的defer应该直接跟在请求资源的语句后。

比如对文件的操作：

```Go
package ioutil
func ReadFile(filename string) ([]byte, error) {
    f, err := os.Open(filename)
    if err != nil {
        return nil, err
    }
    defer f.Close() //函数工作完毕之后调用
    return ReadAll(f)
}
```

或是处理互斥锁：

```Go
var mu sync.Mutex
var m = make(map[string]int)
func lookup(key string) int {
    mu.Lock()
    defer mu.Unlock() //保证锁一定会被释放
    return m[key]
}
```

或者用于记录函数的执行时间：

```Go
func bigSlowOperation() {
    defer trace("bigSlowOperation")() // trace返回的是一个函数，所以需要加括号表示调用
    time.Sleep(10 * time.Second) // simulate slow operation by sleeping
}
func trace(msg string) func() {
    start := time.Now() //首次调用时记录调用的时间
    log.Printf("enter %s", msg)
    return func() { //返回的函数在此被调用是，计算两个时间差值
        log.Printf("exit %s (%s)", msg,time.Since(start)) 
    }
}
```

**对于关闭资源的相关操作，不宜将defer放在循环中，因为很有可能导致循环过程中不断申请的资源没有及时释放导致资源耗尽**



defer语句中的函数会在return语句更新返回值变量后再执行，又因为在函数中定义的匿名函数可以访问该函数包括返回值变量在内的所有变量，所以，对匿名函数采用defer机制，可以使其使用或者修改函数的返回值。

```Go
func triple(x int) (result int) {
    defer func() { result += x }()
    return x + x
}
fmt.Println(triple(4)) // "12"  (4 + 4) + 4
```



下面是一个多defer的示例：

```go
func triple() int {
   x := []int{1}
   defer fmt.Println(x)

   x = append(x, 2)
   defer fmt.Println(x)

   x = append(x, 3)
   defer fmt.Println(x)
   return 4
}

func main() {
   fmt.Println(triple())
}

输出结果：
[1 2 3]
[1 2]
[1]
4
```



# 方法

## 方法定义

在函数声明时，在其名字之前放上一个变量，即是一个方法。这个附加的参数会将该函数附加到这种类型上，即相当于为这种类型定义了一个独占的方法。

方法在调用是同样是传递参数的副本，在方法中对变量中值的修改不会影响调用者的原始值。

```go
//这个函数对s变量的修改在函数退出之后不会生效，因为传入的变量是拷贝
func (s Student) input(name string, math int, chinese int)  {
   s.name = name
   s.math = math
   s.chinese = chinese
   s.total = math + chinese
   s.avg = float32(s.total / 2)
}

//这个方法是可以正常工作的，因为只访问了变量
func (s Student) toString() string {
	return fmt.Sprintf("Student {name: %s, math: %d, chinese: %d, total: %d, avg: %.2f}",
		s.name, s.math, s.chinese, s.total, s.avg)
}
```



如果是传递体积很大的变量或者是在方法内需要对数据进行修改，则传入指针。

```go
func (s *Student) input(name string, math int, chinese int)  {
   s.name = name
   s.math = math
   s.chinese = chinese
   s.total = math + chinese
   s.avg = float32(s.total / 2)
}
```



为了避免歧义，在声明方法时，如果一个类型名本身是一个指针的话，是不允许其出现在接收器中的，比如下面这个例子：

```go
type P *int
func (P) f() { /* ... */ } // compile error: invalid receiver type
```





## 方法调用的三种情况

在每一个合法的方法调用表达式中，也就是下面三种情况里的任意一种情况都是可以的：

1. 接收器的实际参数和其形式参数是相同的类型，比如两者都是类型T或者都是类型`*T`：

2. 接收器实参是类型T，但接收器形参是类型`*T`，这种情况下编译器会隐式地为我们取变量的地址：

3. 接收器实参是类型`*T`，形参是类型T。编译器会隐式地为我们解引用，取到指针指向的实际变量：

**方法的需要注意的有两点：**

1. 不管你的method的receiver是指针类型还是非指针类型，都是可以通过指针/非指针类型进行调用的，编译器会帮你做类型转换。
2. 在声明一个method的receiver该是指针还是非指针类型时，你需要考虑两方面的因素，第一方面是这个对象本身是不是特别大，如果声明为非指针变量时，调用会产生一次拷贝；第二方面是如果你用指针类型作为receiver，那么你一定要注意，这种指针类型指向的始终是一块内存地址，就算你对其进行了拷贝。熟悉C或者C++的人这里应该很快能明白。



## Nil也是一个合法的接收器类型

就像一些函数允许nil指针作为参数一样，方法理论上也可以用nil指针作为其接收器，尤其当nil对于对象来说是合法的零值时，比如map或者slice。在下面的简单int链表的例子里，nil代表的是空链表：

```go
// An IntList is a linked list of integers.
// A nil *IntList represents the empty list.
type IntList struct {
    Value int
    Tail  *IntList
}
// Sum returns the sum of the list elements.
func (list *IntList) Sum() int {
    if list == nil {
        return 0
    }
    return list.Value + list.Tail.Sum()
}
```

当你定义一个允许nil作为接收器值的方法的类型时，在类型前面的注释中指出nil变量代表的意义是很有必要的，就像我们上面例子里做的这样。



## 组合结构体中的方法

如果一个结构体Rectange被匿名组合到另一个结构体Board中，那么在结构体Rectangle中定义的方法可以直接通过结构体Board的变量名以`b.方法名()`的方式方案，_相当于继承了Rectangle中的方法_。

如果一个结构体中有多个匿名结构体，你如下面这种形式：

```go
type structC struct {
    structA
    structB
    n int
}
```

然后structC便会同时拥有structA和structB类型的所有方法，以及直接定义在structC中的方法。当编译器解析一个选择器到方法时，比如structC实例变量c的方法work()，编译器首先去找直接定义在structC中的work方法，然后找被structC的内嵌字段们引入的方法，然后去找structA和structB的内嵌字段引入的方法，然后一直递归向下找。如果选择器有二义性的话编译器会报错，比如你在同一级里有两个同名的方法。



```go
type Rectange struct {
    x       int
    y       int
    acreage int
}

func (r *Rectange) calAcreage() int {
    r.acreage = r.x * r.y
    return r.acreage
}

func (r *Rectange) addAcreage(a Rectange) int {
    return r.calAcreage() + a.calAcreage()
}

func (r *Rectange) scale(n int) {
    r.x *= n
    r.y *= n
}


type Board struct {
    Rectange
    color string
}

func main() {
    var b Board = Board{Rectange{x: 12, y: 23}, "red"}
	var c Board = Board{Rectange{x: 3, y: 23}, "red"}

   	b.scale(2)
	c.scale(2)
	
    var sum = b.addAcreage(c) //错误的调用方法
	var sum = b.addAcreage(c.Rectange) //正确的调用方法
}
```



## 方法值和方法表达式

方法也可以作为变量传递，分为方法值和方法表达式两种情况：

1. 方法值：针对一个具体实例红的方法，调用该方法的时候和普通方法没有区别。

2. 方法表达式：针对一个类中的方法，在赋值方法到变量和调用方法时需要注意一下：

    1. 赋值方法到变量时，接收器为普通变量和指针需要用不同的方式。如果接收器是指针，则需要使用`(*T).方法名`访问函数。
    2. 调用方法时，方法的第一参数需要传入具体的对象实例（类似于Python的self），后面的参数才是具体的方法参数。

方法值的示例：

```go
var b = Board{Rectange{x: 12, y: 23}, "red"}
//将某一个具体实例的函数保存为变量
var sc = b.scale     
var ts = b.toString  
sc(2)
fmt.Println(ts())
```

方法表达式的示例：
```go
var b = Board{Rectange{x: 12, y: 23}, "red"}
//将某一类型的方法保存为变量
var bsc = (*Board).scale //如果接收器是指针，则需要使用(*T).方法名访问函数。
var bts = Board.toString //非指针直接访问方法名。
bsc(&b, 5)
fmt.Println(bts(b))
```



## 封装

Go语言只有一种控制可见性的手段：大写首字母的标识符会从定义它们的包中被导出，小写字母的则不会。这种限制包内成员的方式同样适用于struct或者一个类型的方法。

因而如果我们想要封装一个对象，我们必须将其定义为一个struct。这种基于名字的手段使得在语言中最小的封装单元是package，而不能针对每个单独的变量设置访问权限。一个struct类型的字段对同一个包的所有代码都有可见性，无论你的代码是写在一个函数还是一个方法里。



# 接口

接口类型是对其它类型行为的抽象和概括；因为接口类型不会和特定的实现细节绑定在一起，通过这种抽象的方式我们可以让我们的函数更加灵活和更具有适应能力。

很多面向对象的语言都有相似的接口概念，但Go语言中接口类型的独特之处在于它是满足隐式实现的。也就是说，我们没有必要对于给定的具体类型定义所有满足的接口类型；简单地拥有一些必需的方法就足够了。这种设计可以让你创建一个新的接口类型满足已经存在的具体类型却不会去改变这些类型的定义；当我们使用的类型来自于不受我们控制的包时这种设计尤其有用。



## 接口的定义

接口的声明类似于结构体，函数的声明同函数的定义一样，但是需要移除`func`修饰词。

```go
type Reader interface {
    Read(p []byte) (n int, err error)
    ReadStr(str string)
}

type Closer interface {
    Close() error
}
```

接口可以以组合的形式继承

```go
//将上面两个接口组合成一个新的接口
type ReadWriter interface {
    Reader
    Writer
}

//组合旧的接口，同时定义新的方法
type ReadWriteAdder interface {
    Reader
    Writer
    Add(a int, b int) (s int)
}
```

一个类型如果拥有一个接口需要的所有方法，那么这个类型就实现了这个接口。

我们来定义一个Sayer接口：

```go
// Sayer 接口
type Sayer interface {    
    say()
}
```

定义dog和cat两个结构体：

```go
type dog struct {}
type cat struct {}
```

因为Sayer接口里只有一个say方法，所以我们只需要给dog和cat 分别实现say方法就可以实现Sayer接口了。

```go
// dog实现了Sayer接口
func (d dog) say() {    
    fmt.Println("汪汪汪")
}

// cat实现了Sayer接口
func (c cat) say() {    
    fmt.Println("喵喵喵")
}
```

接口的实现就是这么简单，只要实现了接口中的所有方法，就实现了这个接口。



```go
w = ReadWriteAdder //OK,ReadWriteAdder实现了左侧接口的全部方法
ReadWriteAdder = Writer //not OK, Writer仅实现了ReadWriteAdder的部分方法，赋值失败
```

任意一个值都可以赋值给空接口类型

``` go
var any interface{}
any = true
any = 12.34
any = "hello"
any = map[string]int{"one": 1}
any = new(bytes.Buffer)
```



可以通过使用w==nil或者w!=nil来判断接口值是否为空。调用一个空接口值上的任意方法都会产生panic。**通常在编译期，我们不知道接口值的动态类型是什么，所以一个接口上的调用必须使用动态分配**。



接口值可以使用==和!＝来进行比较。两个接口值相等仅当它们都是nil值，或者它们的动态类型相同并且动态值也根据这个动态类型的==操作相等。因为接口值是可比较的，所以它们可以用在map的键或者作为switch语句的操作数。