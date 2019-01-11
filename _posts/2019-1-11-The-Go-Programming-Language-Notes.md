---
layout: post
title: Go语言圣经读书笔记
date: 2019-01-11
tags:
    - golang
    - 读书笔记
description: Go语言圣经读书笔记。
---
# 前言

<a id="org68ccc8d"></a>

## Go起源

-   Ken Thompson、Rob Pike、Robert Griesemer:为了解决21世纪多核和网络环境越来越复杂的问题。
-   "C类似语言"、"21世纪的C语言"
-   自动垃圾回收、包系统、函数一等公民、词法作用域、系统调用接口、原生支持Unicode。
-   静态、编译。简单的类型系统。
-   没有隐式数值转换、没有构造函数和析构函数、没有运算符重载、没有默认参数、没有继承、没有泛型、没有异常、没有宏、没有函数修饰等。


<a id="orgf16c75c"></a>

# 入门


<a id="org9a79c05"></a>

## Hello, World

    package main

    import "fmt"

    func main() {
        fmt.Println("Hello, 世界")
    }

-   main函数是程序执行的入口
-   函数的声明由func关键字、函数名、参数列表、返回值列表、函数体组成。


<a id="org08562c9"></a>

## 命令行参数

    for initialization; condition; post {
        // zero or more statements
    }

-   空标识符 \_ 用于语法需要逻辑不需要的时候。


<a id="orgce5ffe9"></a>

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


<a id="org8976822"></a>

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


<a id="orgdd1e809"></a>

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


<a id="orgead84eb"></a>

## 获取URL


<a id="org433b0ef"></a>

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


<a id="org82af4c3"></a>

## 本章总结

-   Go语言里没有指针运算，不能像c语言里对指针进行加减操作。
-   在源文件开头和函数开头写注释是个好习惯，会被godoc之类的工具检测到。


<a id="org5d38632"></a>

# 程序结构


<a id="org01a48dc"></a>

## 命名

-   命名规则：以字母或下划线开头，后面可以跟任意数量的字母、数字、下划线。大小写字母是不同的。
-   在函数内部定义的变量只在函数内部有效，在函数外部定义的变量在整个包内有效。
-   包级变量如果以大写字母开头则可以被外部包访问，小写开头的变量则不行。
-   推荐使用 **驼峰式** 命名。


<a id="org1ed53e1"></a>

## 声明

-   每个源文件以包的声明开始，之后是import语句导入依赖的其他包，之后是包一级的类型、变量、常亮、函数的声明，包一级的各种类型声明顺序无关紧要。
-   包一级的变量可以在包对应的所有源文件中访问，而不仅仅是在声明它的源文件中访问。


<a id="org6407bc3"></a>

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


<a id="orgf7727d1"></a>

## 赋值

-   自增自减是语句而不是表达式，因此x = i++之类的语句是错误的。
-   可赋值性的简单规则：类型必须完全匹配，nil可以赋值给任何指针或者引用类型的变量。


<a id="org8d286bd"></a>

## 类型

-   类型定义了数值在内存中的存储大小、如何组织的、支持的操作等。
-   类型声明语句：type 类型名字 底层类型；
-   对于每一个类型T，都有一个类型转换操作T(x)，用于将x类型转换为T类型。转换的前提是两种类型拥有相同的底层类型。


<a id="org60334c4"></a>

## 包和文件

-   目的是为了支持模块化、封装、单独编译和代码重用。
-   每个包对应一个独立的命名空间。
-   一个包通常只有一个源文件有包注释。
