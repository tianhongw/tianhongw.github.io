---
layout: post
title: Golang 之 Array Slice 以及 Append 机制简介
date: 2019-03-15
tags:
    - golang
description: array slice append
---


# Array

在使用Go编程的时候,大部分情况下不用array,因为数组长度是数组类型的一部分,灵活性不够好.但是它是Golang很重要的部分,与slice有密切的关系.

定义: `var buffer [256]byte`

定义了一个拥有256个字节的数组,在内存中大概是这么表示的:

buffer: byte byte &#x2026; 256 times &#x2026; byte byte

可一通过 `buffer[0]` 至 `buffer[255]` 访问buffer数组. 试图访问数组之外的元素会导致程序挂掉.使用内置函数 `len()` 返回数组的长度,在这个例子中, `len(buffer)` 返回256.

在Go的设计中, array是为slice服务的.


# Slice


## 什么是slice

Slice是一种数据结构,它描绘了底层数组一段连续的部分.

定义: `var slice []byte = buffer[100:150]` 或着 `var slice = buffer[100:150]`

如果在函数中这样定义 `slice := buffer[100:150]` 这样slice就是底层数组buffer[100, 149]区间的一个视图.

可以简单的把slice想象成如下的数据结构:

    type sliceHeader struct {
        Length  int
        Element *byte
    }
    slice := sliceHeader{
        Length:  50,
        Element: &buffer[100],
    }

可以用slice定义slice: `slice2 := slice[5:10]` 新的slice包含原slice[5,9]区间的元素,也就是说包含buffer[105,109]区间的元素.类似:

    slice2 := sliceHeader{
        Length:  5,
        Element: &buffer[105],
    }

注意到slice2仍然指向底层的buffer数组,对slice2做改变会影响底层buffer和其它指向底层数组的slice.


## 函数调用时Slice的传递

要理解: 虽然slice包含了指向底层数组的指针,但是slice本身仍然是一个值类型.

在函数调用的时候传递的是类似sliceHeader的结构.比如定义了一个函数:

    func f(s []int) {
        //...
    }

`f(slice)` 在值传递的过程中,s拷贝了一份slice的sliceHeader. 也就是说 `f(slice)` 可以改变slice底层所指向的数组,但是不能改变slice的其它属性,比如 `length` 以及还有没有提及的 `cap`

    package main

    import "fmt"

    func main() {
        s := make([]int, 3, 10)
        double(s)
        fmt.Println("slice: ", s, "length: ", len(s), "cap: ", cap(s))
    }

    func double(s []int) {
        s = append(s, s...)
    }

    // Output:
    // slice:  [0 0 0] length:  3 cap:  10

main函数中的slice并没有改变它的 `length` 和 `cap` 属性.

如果一定要改变呢？可以让 `double(s []int) []int` 返回改变后的slice赋值给main函数中的slice.

第二种方法是使用指向slice的指针:

    package main

    import "fmt"

    func main() {
        s := make([]int, 2, 10)
        fmt.Println("Before: len(s) = ", len(s))
        ptrSubtractOneFromLength(&s)
        fmt.Println("After: len(s) = ", len(s))
    }

    func ptrSubtractOneFromLength(slicePtr *[]int) {
        *slicePtr = (*slicePtr)[0 : len(*slicePtr)-1]
    }

    // Output:
    // Before: len(s) =  2
    // After: len(s) =  1

比起第二种方法,使用指针接收者改变slice是比较常见的方式:

    package main

    import (
        "bytes"
        "fmt"
    )

    type path []byte

    func (p *path) TruncateAtFinalSlash() {
        i := bytes.LastIndex(*p, []byte("/"))
        if i >= 0 {
            *p = (*p)[0:i]
        }
    }

    func main() {
        pathName := path("/usr/bin/tso")
        pathName.TruncateAtFinalSlash()
        fmt.Printf("%s\n", pathName)
    }

    // Output:
    // /usr/bin

如果你只想改变slice指向的底层数组的值,可以使用值接收者:

    package main

    import (
        "fmt"
    )

    type path []byte

    func (p path) ToUpper() {
        for i, b := range p {
            if b >= 'a' && b <= 'z' {
                p[i] = b + 'A' - 'a'
            }
        }
    }

    func main() {
        pathName := path("/usr/bin/tso")
        pathName.ToUpper()
        fmt.Printf("%s\n", pathName)
    }

    // Output:
    // /USR/BIN/TSO


## Cap

除了指向底层数组的指针和长度,slice拥有另外一个属性:容量cap.

    type sliceHeader struct {
        Element  *byte
        Length   int
        Capacity int
    }

cap属性表示了slice指向的底层数组最多拥有的空间长度,如果在超过容量的情况下试图增长slice会导致panic.
看看下面的函数,该函数增加一个元素到传入的切片上:

    package main

    import (
        "fmt"
    )

    func Extend(slice []int, element int) []int {
        n := len(slice)
        slice = slice[0 : n+1]
        slice[n] = element
        return slice
    }

    func main() {
        var buffer [10]int
        slice := buffer[0:0]
        for i := 0; i < 20; i++ {
            slice = Extend(slice, i)
            fmt.Println(slice)
        }
    }

    // Output:
    // [0]
    // [0 1]
    // [0 1 2]
    // [0 1 2 3]
    // [0 1 2 3 4]
    // [0 1 2 3 4 5]
    // [0 1 2 3 4 5 6]
    // [0 1 2 3 4 5 6 7]
    // [0 1 2 3 4 5 6 7 8]
    // [0 1 2 3 4 5 6 7 8 9]
    // panic: runtime error: slice bounds out of range

在main函数中 `slice := buffer[0:0]` 相当于:

    slice := sliceHeader{
        Element:  &buffer[0],
        Length:   0,
        Capacity: 10,
    }


## Make

   如果你想不受限于slice的容量来增长它呢？从slice的定义上来说是不能做到的,容量是slice所容纳数据数量的上限.但是可以通过重新分配一个容量更大的新数组，把数据拷贝过去,让原来的slice指向新的底层数组达到扩张slice的目的.内置的 `make` 方法接受3个参数: *slice的类型*、*初始长度*、*slice的容量*,如果不指定容量,默认容量和其初始长度相等.它具体的工作就是创建一个指定属性的数组，然后返回一个指向底层数组的slice。

    package main

    import "fmt"

    func Insert(slice []int, index, value int) []int {
        slice = slice[0 : len(slice)+1]
        copy(slice[index+1:], slice[index:])
        slice[index] = value
        return slice
    }

    func main() {
        slice := make([]int, 10, 20)
        for i := range slice {
            slice[i] = i
        }
        fmt.Println(slice)
        slice = Insert(slice, 5, 100)
        fmt.Println(slice)
    }


## Copy

在上一节中定义了 `newslice` 并且用循环去给它赋值,Go内置 `copy` 函数使其更简单:

    newSlice := make([]int, len(slice), 2*cap(slice))
    copy(newSlice, slice)

`copy` 足够聪明,它只拷贝两个slice中长度最短的部分，同时它返回一个int类型的值表示有多少元素被拷贝.

下面是使用 `copy` 函数的一个例子:

    package main

    import "fmt"

    func Insert(slice []int, index, value int) []int {
        slice = slice[0 : len(slice)+1]
        copy(slice[index+1:], slice[index:])
        slice[index] = value
        return slice
    }

    func main() {
        slice := make([]int, 10, 20)
        for i := range slice {
            slice[i] = i
        }
        fmt.Println(slice)
        slice = Insert(slice, 5, 100)
        fmt.Println(slice)
    }

    // Output:
    // [0 1 2 3 4 5 6 7 8 9]
    // [0 1 2 3 4 100 5 6 7 8 9]


## Append

在Cap章节中写的 `Extend` 函数存在bug:如果slice的容量太小,向slice添加元素会导致程序崩溃.下面来修复这个bug:

    package main

    import "fmt"

    func Extend(slice []int, element int) []int {
        n := len(slice)
        if n == cap(slice) {
            newSlice := make([]int, n, 2*n+1)
            copy(newSlice, slice)
            slice = newSlice
        }
        slice = slice[0 : n+1]
        slice[n] = element
        return slice
    }

    func main() {
        slice := make([]int, 0, 5)
        for i := 0; i < 10; i++ {
            slice = Extend(slice, i)
            fmt.Printf("len = %d, cap = %d slice = %v\n", len(slice), cap(slice), slice)
            fmt.Println("address of 0th element: ", &slice[0])
        }
    }

    // Output:
    // len = 1, cap = 5 slice = [0]
    // address of 0th element:  0xc0000160c0
    // len = 2, cap = 5 slice = [0 1]
    // address of 0th element:  0xc0000160c0
    // len = 3, cap = 5 slice = [0 1 2]
    // address of 0th element:  0xc0000160c0
    // len = 4, cap = 5 slice = [0 1 2 3]
    // address of 0th element:  0xc0000160c0
    // len = 5, cap = 5 slice = [0 1 2 3 4]
    // address of 0th element:  0xc0000160c0
    // len = 6, cap = 11 slice = [0 1 2 3 4 5]
    // address of 0th element:  0xc000070060
    // len = 7, cap = 11 slice = [0 1 2 3 4 5 6]
    // address of 0th element:  0xc000070060
    // len = 8, cap = 11 slice = [0 1 2 3 4 5 6 7]
    // address of 0th element:  0xc000070060
    // len = 9, cap = 11 slice = [0 1 2 3 4 5 6 7 8]
    // address of 0th element:  0xc000070060
    // len = 10, cap = 11 slice = [0 1 2 3 4 5 6 7 8 9]
    // address of 0th element:  0xc000070060

注意到当原来的slice被填满后,新分配的slice容量和地址都发生了改变,也就是说slice指向了新的底层数组.

类似于内置 `append()` 函数,要支持添加多个元素可以这样写:

    func Append(slice []int, elements ...int) []int {
        n := len(slice)
        total := len(slice) + len(elements)
        if total > cap(slice) {
            newCap := total*2 + 1
            newSlice := make([]int, total, newCap)
            copy(newSlice, slice)
            slice = newSlice
        }
        slice = slice[:total]
        copy(slice[n:], elements)
        return slice
    }

这就是Go内置的 `append()` 的工作原理,只是在容量增长算法上更复杂，支持任何类型的slice.


## nil slice

nil slice 是指:

    nilSlice := sliceHeader{
        Element:  nil,
        Length:   0,
        Capacity: 0,
    }

或者这样定义: `nilSlice := sliceHeader{}`

但是这样定义的slice不属于nil slice: `notNilSlice := buffer[0:0]`

关键点在于指向底层数组的指针是否为空.
