---
layout: post
title: 什么情况下在 Go 中使用 <基础类型指针>
author: Joey <majunjiev@gmail.com>
tags: [Golang]
---


## 前言

读书学 C 时我知道了指针数组和数组指针，使我对指针有了份特殊的感情。后来做项目我用了5年 Java， 近3年用 Python，本以为这辈子都不会碰指针了，但没想到 Go 的横空出世，让我和指针又续上了缘分。

我还是 Go 菜鸟，写代码没过 3k 行。于是对指针的理解上就比较狭隘：指针，基本是用来修饰 struct 的，而基础类型（string/bool/int 等）的指针，只有一个用处：**作为函数的参数，将传值改为传引用，在函数内部修改传入的参数值**。

直到后来我在做 [oVirt Go SDK](https://github.com/imjoey/ovirt-engine-sdk-go) 时，oVirt 社区的 maintainers 给我提了一些修改建议，其中就使用基础类型指针，很巧妙的解决了一个棘手问题。

下面先介绍基础类型指针比较基础的一个应用：函数参数，然后介绍它的另一大用处。


## 基础类型指针作为函数参数

这个作用比较基础，因为 Go 的函数参数都是传值，所以如果想在函数内部修改参数值，就必须传指针。比如下面这个例子：[执行代码](https://play.golang.org/p/RRPt9C3Lua)
```go
package main

import (
	"fmt"
)

func CannotModify(value int64) {
	value = 63
}

func CanModify(value *int64) {
	*value = 500
}

func main() {
	var value int64 = 65
	CannotModify(value)
	fmt.Printf("value is %v, without modified\n", value)
	CanModify(&value)
	fmt.Printf("value is %v, with modified\n", value)
}

```

根据结果可以看到，函数 CanModify 使用 *int64 作为参数，成功修改了变量 value 的值。


## Unmarshal 时判断是否有值

oVirt SDK 是通过调用 oVirt 的接口实现管理功能，接口的数据都是 XML 格式。所以在实现时少不了使用 Go 的 xml.Marshall 和 xml.Unmarshall 函数实现 XML 与 Go struct 之间的转换。

> 写到这里，需要先介绍一下 Go 的**零值**，其实在 Java 和 Python 中，任何类型（包含基础类型）的变量，在赋值都可以设置为 null/None。但 Go 下，基础类型的零值并不是 nil，也不能设为 nil。如：string 的零值是 ""，bool 的零值是 false，int 的零值是0。

以下面这段代码介绍 Go 零值的一个问题：（[执行代码](https://play.golang.org/p/2bpy1tg09r)）

```go
package main

import (
	"fmt"
	"encoding/xml"
)


type VM struct {
	Name string `xml:"name,omitempty"`
	Desc string `xml:"desc,omitempty"`
}

type VMPtr struct {
	Name *string `xml:"name,omitempty"`
	Desc *string `xml:"desc,omitempty"`
}

var xmlStr = `
<vm>
<name>vm-name</name>
</vm>
`

var xmlStr2 = `
<vm>
<name>vm-name</name>
<desc></desc>
</vm>
`


func main() {
	var vm1 VM
	xml.Unmarshal([]byte(xmlStr), &vm1)
	fmt.Printf("vm1 name is %#v, desc is %#v\n", vm1.Name, vm1.Desc)
	
	var vm2 VM
	xml.Unmarshal([]byte(xmlStr2), &vm2)
	fmt.Printf("vm2 name is %#v, desc is %#v\n", vm2.Name, vm2.Desc)
	
	// 对于以上代码，如果 Desc 属性是 string 类型，则在 Unmarshal 时无法区分 xml 中：<不存在desc属性> 还是 <desc是空串("")>
	
	var vmPtr1 VMPtr
	xml.Unmarshal([]byte(xmlStr), &vmPtr1)
	fmt.Printf("vm (ptr) name is %#v(addr:%v), desc is %#v(addr: not exits)\n", *vmPtr1.Name, vmPtr1.Name, vmPtr1.Desc)
	
	var vmPtr2 VMPtr
	xml.Unmarshal([]byte(xmlStr2), &vmPtr2)
	fmt.Printf("vm (ptr) name is %#v(addr:%v), desc is %#v(addr:%v)\n", *vmPtr2.Name, vmPtr2.Name, *vmPtr2.Desc, vmPtr2.Desc)
	
	// 而如果 Desc 属性是 *string 类型，则就可以通过 <*string 指针是否是 nil> 去判断xml中是否有desc属性了

}

```

从以上代码的注释和输出，可以看出来使用 *string 可以区分出 VM 的 Desc 属性是否在 xml.Unmarshal 过程中被赋值了。而这，就是基础类型指针的最大用处了。

> 附加福利：如果使用基础类型指针（比如*string），在 xml.Marshal 时，不用在 tag 中定义omitempty，xml.Marshal 自动不输出 nil 的指针。以下面代码为例：[执行代码](https://play.golang.org/p/ZMaXB3_qnS)

```go
package main

import (
	"fmt"
	"encoding/xml"
)


type VM struct {
	Name string `xml:"name,omitempty"`
	Desc string `xml:"desc"`
}

type VMPtr struct {
	Name *string `xml:"name"`
	Desc *string `xml:"desc"`
}


func main() {
	var vm VM
	vm.Name = "vm-name"
	xmlVMBytes, _ := xml.Marshal(vm)
	fmt.Printf("vm marshal is %v\n", string(xmlVMBytes))
	fmt.Println("----------------------------------------")
	var vmPtr VMPtr
	temp := "vmPtr-name"
	vmPtr.Name = &temp
	xmlVMPtrBytes, _ := xml.Marshal(vmPtr)
	fmt.Printf("vmPtr marshal is %v\n", string(xmlVMPtrBytes))

}
```


## 总结

基础类型指针在 xml/json/ini 的 Unmarshal 过程中，可以自动判断属性是否被赋值，同时在 Marshal 过程中，可以不用在 tag 中定义 omitempty。