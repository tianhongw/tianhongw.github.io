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


<a id="org7c6a3b1"></a>

## Go起源

-   Ken Thompson、Rob Pike、Robert Griesemer:为了解决21世纪多核和网络环境越来越复杂的问题。
-   "C类似语言"、"21世纪的C语言"
-   自动垃圾回收、包系统、函数一等公民、词法作用域、系统调用接口、原生支持Unicode。
-   静态、编译。简单的类型系统。
-   没有隐式数值转换、没有构造函数和析构函数、没有运算符重载、没有默认参数、没有继承、没有泛型、没有异常、没有宏、没有函数修饰等。


<a id="orgd45813e"></a>

# 入门


<a id="org30f16f8"></a>

## Hello, World

    package main

    import "fmt"

    func main() {
        fmt.Println("Hello, 世界")
    }

-   main函数是程序执行的入口
-   函数的声明由func关键字、函数名、参数列表、返回值列表、函数体组成。


<a id="org6c21a73"></a>

## 命令行参数

    for initialization; condition; post {
        // zero or more statements
    }

-   空标识符 \_ 用于语法需要逻辑不需要的时候。


<a id="orgd65fe80"></a>

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


<a id="orgfe54f92"></a>

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


<a id="orgcfb3b71"></a>

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


<a id="orgf7e1d6f"></a>

## 获取URL


<a id="orgc5761cd"></a>

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


<a id="org81446c8"></a>

## 本章总结

-   Go语言里没有指针运算，不能像c语言里对指针进行加减操作。
-   在源文件开头和函数开头写注释是个好习惯，会被godoc之类的工具检测到。
