---
layout: post
title: Golang 之 Arrary Slice 以及 Append 机制简介
date: 2019-03-15
tags:
    - Golang
description: Golang arrary,slice,append机制简介。
---


# Array

在使用Go编程的时候,大部分情况下不用array,因为数组长度是数组类型的一部分,灵活性不够好.但是它是Golang很重要的部分,与slice有密切的关系.

定义: `var buffer [256]byte`

定义了一个拥有256个字节的数组,在内存中大概是这么表示的:

buffer: byte byte &#x2026; 256 times &#x2026; byte byte

可一通过 `buffer[0]` 至 `buffer[255]` 访问buffer数组. 试图访问数组之外的元素会导致程序挂掉.使用内置函数 `len()` 返回 数组的长度,在这个例子中, `len(buffer)` 返回256.

在Go的设计中, array是为slice服务的.


# Slice


## 什么是slice

Slice是一种数据结构,它描绘了底层数组一段连续的部分.

定义: `var slice []byte = buffer[100:150]` 或着 `var slice = buffer[100:150]` 如果在函数中这样定义 `slice := buffer[100:150]`

这样slice就是底层数组buffer[100, 149]区间的一个视图.

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

`f(slice)` 在值传递的过程中,s拷贝了一份slice的sliceHeader. 也就是说 `f(slice)` 可以改变slice底层所指向的数组,但是不能改变slice的其它属性,比如 `length` 还有没有提及的 `cap`

    package main

    import "fmt"

    func main() {
        s := []int{1, 2, 3}
        double(s)
        fmt.Println("slice: ", s, "length: ", len(s), "cap: ", cap(s))
    }

    func double(s []int) {
        s = append(s, s...)
    }

main函数中的slice并没有改变它的 `length` 和 `cap` 属性.如果一定要改变呢？可以让 ~double(s []int) []int~返回改变后的slice赋值给main函数中的slice.
