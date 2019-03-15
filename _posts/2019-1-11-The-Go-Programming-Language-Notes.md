---
layout: post
title: Go语言圣经读书笔记
date: 2019-01-11
tags:
    - Golang
    - ReadingNotes
description: Go语言圣经读书笔记。
---
# 前言


<a id="org66b1281"></a>

## Go起源

-   Ken Thompson、Rob Pike、Robert Griesemer:为了解决21世纪多核和网络环境越来越复杂的问题。
-   "C类似语言"、"21世纪的C语言"
-   自动垃圾回收、包系统、函数一等公民、词法作用域、系统调用接口、原生支持Unicode。
-   静态、编译。简单的类型系统。
-   没有隐式数值转换、没有构造函数和析构函数、没有运算符重载、没有默认参数、没有继承、没有泛型、没有异常、没有宏、没有函数修饰等。


<a id="orgd2c64af"></a>

# 入门


<a id="org3660632"></a>

## Hello, World

    package main

    import "fmt"

    func main() {
        fmt.Println("Hello, 世界")
    }

-   main函数是程序执行的入口
-   函数的声明由func关键字、函数名、参数列表、返回值列表、函数体组成。


<a id="org942ff5f"></a>

## 命令行参数

    for initialization; condition; post {
        // zero or more statements
    }

-   空标识符 \_ 用于语法需要逻辑不需要的时候。


<a id="orga9c9e69"></a>

### 练习1.1

    package main

    import (
        "fmt"
        "os"
        "strings"
    )

    func main() {
        fmt.Println(strings.Join(os.Args[0:], " "))
    }


<a id="org3e748ee"></a>

### 练习1.2

    package main

    import (
        "fmt"
        "os"
    )

    func main() {
        for i, arg := range os.Args[1:] {
            fmt.Println(i, arg)
        }
    }


<a id="org733cfd9"></a>

## 查找重复的行

-   函数和包级别的变量可以以任意顺序声明，并不影响其被调用。
-   常量声明的值必须是一个数字值、字符串、布尔值。
-   map作为参数传递给某函数时，该函数接受这个引用的一份拷贝，类似c++中的引用传递，实际上指针是另一个指针，但指向同一块内存。

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<tbody>
<tr>
<td class="org-left">%d</td>
<td class="org-left">十进制整数</td>
</tr>


<tr>
<td class="org-left">%x, %o, %b</td>
<td class="org-left">十六进制，八进制，二进制</td>
</tr>


<tr>
<td class="org-left">%f, %g, %e</td>
<td class="org-left">浮点数</td>
</tr>


<tr>
<td class="org-left">%t</td>
<td class="org-left">布尔</td>
</tr>


<tr>
<td class="org-left">%c</td>
<td class="org-left">字符</td>
</tr>


<tr>
<td class="org-left">%s</td>
<td class="org-left">字符串</td>
</tr>


<tr>
<td class="org-left">%v</td>
<td class="org-left">变量的自然形式</td>
</tr>


<tr>
<td class="org-left">%T</td>
<td class="org-left">变量的类型</td>
</tr>
</tbody>
</table>


<a id="org69b1928"></a>

## 获取URL


<a id="org6c2b79f"></a>

### 练习1.7

    package main

    import (
        "fmt"
        "io"
        "net/http"
        "os"
    )

    func main() {
        for _, url := range os.Args[1:] {
            resp, err := http.Get(url)
            if err != nil {
                fmt.Fprintf(os.Stderr, "fetch: %v\n", err)
                os.Exit(1)
            }
            // b, err := ioutil.ReadAll(resp.Body)
            _, err1 := io.Copy(os.Stdout, resp.Body)
            resp.Body.Close()
            if err1 != nil {
                fmt.Fprintf(os.Stderr, "fetch: reading %s: %v\n", url, err1)
                os.Exit(1)
            }
            fmt.Printf("%v", os.Stdout)
        }
    }


<a id="org9d54f36"></a>

## 本章总结

-   Go语言里没有指针运算，不能像c语言里对指针进行加减操作。
-   在源文件开头和函数开头写注释是个好习惯，会被godoc之类的工具检测到。


<a id="org55dda5f"></a>

# 程序结构


<a id="org7271ad1"></a>

## 命名

-   命名规则：以字母或下划线开头，后面可以跟任意数量的字母、数字、下划线。大小写字母是不同的。
-   在函数内部定义的变量只在函数内部有效，在函数外部定义的变量在整个包内有效。
-   包级变量如果以大写字母开头则可以被外部包访问，小写开头的变量则不行。
-   推荐使用 **驼峰式** 命名。


<a id="org463677e"></a>

## 声明

-   每个源文件以包的声明开始，之后是import语句导入依赖的其他包，之后是包一级的类型、变量、常亮、函数的声明，包一级的各种类型声明顺序无关紧要。
-   包一级的变量可以在包对应的所有源文件中访问，而不仅仅是在声明它的源文件中访问。


<a id="org4cdcd49"></a>

## 变量

-   变量声明的语法：var 变量名称 变量类型 = 表达式
-   包级别声明的变量在main函数执行前初始化，局部变量在执行到的时候初始化。
-   简短变量声明左边的变量可能之前声明过的，这些变量是赋值行为，注意简短变量声明中必须要有新变量。
-   在go中返回局部变量的地址是安全的。
-   变量的生命周期：包一级的变量和整个程序的运行周期一致；局部变量的生命周期是动态的：从创建开始到不再被引用为止。

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">类型</th>
<th scope="col" class="org-left">零值</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">数值</td>
<td class="org-left">0</td>
</tr>


<tr>
<td class="org-left">布尔</td>
<td class="org-left">false</td>
</tr>


<tr>
<td class="org-left">字符串</td>
<td class="org-left">空字符串</td>
</tr>


<tr>
<td class="org-left">接口、引用(slice/map/chan/函数)</td>
<td class="org-left">nil</td>
</tr>


<tr>
<td class="org-left">数组、结构体</td>
<td class="org-left">每个元素或字段的零值</td>
</tr>
</tbody>
</table>


<a id="org0f84300"></a>

## 赋值

-   自增自减是语句而不是表达式，因此x = i++之类的语句是错误的。
-   可赋值性的简单规则：类型必须完全匹配，nil可以赋值给任何指针或者引用类型的变量。


<a id="orgcf555ba"></a>

## 类型

-   类型定义了数值在内存中的存储大小、如何组织的、支持的操作等。
-   类型声明语句：type 类型名字 底层类型；
-   对于每一个类型T，都有一个类型转换操作T(x)，用于将x类型转换为T类型。转换的前提是两种类型拥有相同的底层类型。


<a id="org4b1136b"></a>

## 包和文件

-   目的是为了支持模块化、封装、单独编译和代码重用。
-   每个包对应一个独立的命名空间。
-   一个包通常只有一个源文件有包注释。


<a id="org6256fc1"></a>

# 基础数据类型

Go语言数据类型分类：基础类型、符合类型、引用类型、接口类型。基础类型包括：数字、字符串、布尔类型；符合类型包括：数组、结构体；引用类型包括：指针、切片、map、函数、通道。


<a id="org8fcaa76"></a>

## 整型

-   有符号整型：int8,int16,int32,int64分别对应8/16/32/64位有符号整数。
-   无符号整型：uint8,uint16,uint32,uint64分别对应8/16/32/64位无符号整数。
-   int和uint是32位还是64位取决于编译器和机器平台。
-   rune类型和int32类型等价，byte类型和uint8类型等价。


<a id="orge2ad827"></a>

## 浮点型

-   float32类型大约6个十进制精度、float64类型大约15个十进制精度。
-   %g %e %f用于打印浮点数。


<a id="org4921b90"></a>

## 复数

-   类型：complex64/complex128分别对应float32和float64的精度。
-   定义：var x complex128 = complex(1, 2) // 1 + 2i.或者 x := 1 + 2i.
-   返回复数的实部和虚部分别用内建函数real和imag.


<a id="org5c561fc"></a>

## 布尔型

-   布尔型的值只有两个：true和false.
-   布尔类型可以和&&、||操作符进行运算。如果运算符左边的值可以确定整个表达式的值，则运算符右边的值不再执行。
-   布尔值不会隐式转换为0和1.


<a id="orgdc9391d"></a>

## 字符串

-   字符串是不可改变的序列。
-   第i个字节并不一定是字符串的第i个字符，对于飞ASCII字符的UTF-8编码会要两个或多个字节表示一个字符。
-   s[i:j]基于原始字符串的第i个字节开始到第j个字节(不包括j本身)生成一个新的字符串。
-   讲一个整数转型为字符串意思是生成对应Unicode码点的UTF-8字符串。


<a id="org2d5f210"></a>

## 常量

-   常量表达式的值在编译期计算，而不是在运行期。
-   常量之间的算数运算、逻辑运算、比较运算的结果也是常量。
-   无类型常量
-   无类型整数常量默认转换为int，浮点数和复数常量默认转换为float64和complex128.


<a id="org7db74d0"></a>

# 复合数据类型

数组是由同构的元素组成，结构体是由异构的元素组成。


<a id="orgeea54f6"></a>

## 数组

-   数组是由一个固定长度的特定类型元素组成的序列，由于数组的长度是固定的，在Go中很少使用数组，而是和它对应的slice.
-   数组的长度是数组类型的一部分，因此[2]int和[3]int是不同的类型。
-   数组的长度必须是常量表达式，数组的长度在编译时确定。


<a id="org22ac1c3"></a>

## Slice

-   slice代表变长的序列，序列中每个元素有相同的类型，一个slice类型通常写作[]T.
-   构成：指针、长度、容量。
-   slice之间不能像数组一样直接比较。
-   内置的make函数创建一个制定元素类型、长度和容量的slice.


<a id="org0ade1cc"></a>

## Map

-   无序key/value对的集合。常数时间检索、更新、删除。
-   key必须是支持==比较运算符的类型。将浮点数作为key是个不好的选择。
-   m := make(map[string]int)
-   m := map[string]int{}创建一个空map.
-   如果查找失败则会返回value类型的零值。
-   禁止对map元素取地址。map随着元素增长可能重新分配内存地址。
-   for range对map迭代，顺序是不确定的。
-   map的绝大部分操作，包括查找、删除、len、range循环都可以安全工作在nil值的map上。但是向nil值的map存入元素将导致panic异常。


<a id="orge636c53"></a>

## 结构体

-   结构体是一种聚合类型，由零个或多个任意类型聚合而成。
-   结构体成员的输入顺序也有重要意义，成员的声明顺序不同定义不同的结构体类型。
-   名为s的结构体不能再包含s类型的成员，但是可以包含\*s类型的成员，比如用于二叉树。
-   结构体零值是每个成员对应都是零值。
-   如果要在函数内部修改结构体成员，必须用指针传入。
-   如果结构体的全部成员是可比较的，那么结构体也可以比较，可以作为map的key.
-   结构体嵌入和匿名成员。外层的结构体不仅获得匿名成员的所有成员，而且获得的匿名成员的所有方法。


<a id="org07af649"></a>

## JSON

-   一种用于发送和接受结构化信息的标准协议。


<a id="orgac1cae5"></a>

## 文本和HTML模板


<a id="org3be7507"></a>

# 函数


<a id="org82dd5f1"></a>

## 函数声明

-   函数声明包括函数名、参数列表、返回值列表(可省略)、函数体。
-   Go语言没有默认参数值。
-   实参通过值传递，因此对形参的修改不会影响实参。如果实参包括引用类型，比如指针、slice、map、fuction、channel等类型，实参可能会由于函数的间接引用被修改。
-   没有函数体的函数声明表示函数不是以Go实现的。


<a id="org000321d"></a>

## 多返回值

-   在Go中一个函数可以返回多个值。许多标准库函数返回两个值，一个是期望得到的值，一个是函数出错时的错误信息。
-   调用多返回值的函数时，调用者必须显式的将这些值分配给变量，如果某个值不再被使用将其分配给 \_
-   如果一个函数将所有的返回值都显式的命名，则该函数的return语句可以省略操作数，不推荐使用因为这会使得代码难以理解。


<a id="org7f35e30"></a>

## 错误

-   内置的error是接口类型，可能是nil或者non-nil。nil意味着函数运行成功，non-nil表示函数运行失败。
-   当函数返回值是non-nil时其它的返回值是未定义的，在大部分情况下应该忽略这些返回值。
-   错误处理策略。


<a id="org6d5532b"></a>

## 函数值

-   函数被看做第一类值：函数像其它值一样，拥有类型，可以被赋值给其它变量，传递给函数，从函数返回。
-   函数类型的零值是nil。调用nil值的函数会引发panic异常。
-   函数之间是不可比较的，不能作为map的key.


<a id="orgc918ce3"></a>

## 匿名函数

-   闭包的概念。


<a id="org165e998"></a>

## 可变参数

-   声明可变参数函数时，在参数列表的最后一个参数类型前加上省略符号&#x2026;表示函数接受任意类型该参数。
-   可变参数函数和以slice作为参数的函数是不同的。


<a id="org018cf65"></a>

## defer

-   延迟执行，执行顺序和声明顺序相反。
-   用于成对的操作：打开、关闭、连接、断开连接、加锁、释放锁。


<a id="orgf9a6492"></a>

## panic异常

-   panic一般用于严重错误，大部分情况应该使用go提供的错误机制。


<a id="orgdc6124f"></a>

## recover捕获异常


<a id="org30e94da"></a>

# 方法

封装和组合


<a id="org9110f87"></a>

## 方法声明

-   在函数声明时，在名字前放上一个变量，就是一个方法。这个附加的参数会将函数附加到这种类型上。附加的参数称为接受者。
-   方法不仅可以定义在struct类型上，数值、字符串、slice、map等也可以定义方法。方法可以被声明到任意类型，除了指针和interface.


<a id="org4d71837"></a>

## 基于指针对象的方法

-   当调用一个函数时，会对每一个参数进行拷贝，如果需要更新变量或者参数太大为了避免拷贝，需要用到指针。
-   一般约定，如果一个类型有一个指针作为接受者的方法，那么这个类型的其它方法也要用指针接受者。
-   如果t是一个T类型的变量，Method方法被声明为指针接受者，t.Method()可以编译通过，编译器会隐式的用&t调用这个方法。
-   不管method的receiver是指针类型还是非指针类型，都可以通过指针/非指针类型调用，编译器会做类型转换。


<a id="org2513c5c"></a>

## 通过嵌入结构体扩展类型


<a id="org654893d"></a>

## 方法值和方法表达式


<a id="org0bb2117"></a>

## 封装

-   封装：变量和方法对调用者不可见。


<a id="org51d7d06"></a>

# 接口


<a id="orge3b5812"></a>

## 接口是合约

-   接口类型是一种抽象类型。


<a id="org8a021c2"></a>

## 接口类型

-   接口类型描述了一系列方法的集合，一个实现了这些方法的具体类型是这个接口类型的实例。
-   接口内嵌


<a id="org57b50f8"></a>

## 实现接口的条件

-   一个类型如果实现了一个接口的所有方法，那么这个类型就实现了这个接口。
-   interface{}被称为空接口，可以将任意类型赋值给空接口。
