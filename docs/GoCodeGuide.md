# Go编码规范指南

文档维护 | 辰枫
---|---
更新日期 | 2018-4-10
文档版本 | v1.0

参考：http://golanghome.com/post/550

## 接口设计规范

### 接口命名规范

- 请求与响应字段均为**首字母小写的驼峰命名**，数据结构
- 含义相同的字段在不同接口使用相同的名称
- 接口功能保持单一，一个接口不可通过特殊的调用方式执行不同的功能

## 行长约定

一行最长不超过80个字符，超过的请使用换行展示，尽量保持格式优雅。

## go vet

vet工具可以帮我们静态分析我们的源码存在的各种问题，例如多余的代码，提前return的逻辑，struct的tag是否符合标准等。

`go get golang.org/x/tools/cmd/vet`

使用如下：

`go vet .`

## package名字

保持package的名字和目录保持一致，尽量采取有意义的包名，简短，有意义，尽量和标准库不要冲突。

## 合理封装

合理的封装可以确保使用者可以轻松理解代码的功能，代码编写者应该做到：
- 分清公有、私有成员，调用者不会用到或者不允许使用的成员均设置为私有成员
- 用中文写清代码注释，在GoLand中可以通过```quick documents```快速查看注释

## import 规范

import在多行的情况下，goimports会自动帮你格式化，但是我们这里还是规范一下import的一些规范，如果你在一个文件里面引入了一个package，还是建议采用如下格式：
```go
import (
    "fmt"
)
```
如果你的包引入了三种类型的包，标准库包，程序内部包，第三方包，建议采用如下方式进行组织你的包：
```go
import (
    "encoding/json"
    "strings"

    "myproject/models"
    "myproject/controller"
    "myproject/utils"

    "github.com/astaxie/beego"
    "github.com/go-sql-driver/mysql"
)
```
有顺序的引入包，不同的类型采用空格分离，第一种实标准库，第二是项目包，第三是第三方包。

在项目中不要使用相对路径引入包：
```go
// 这是不好的导入
import "../net"

// 这是正确的做法
import "github.com/repo/proj/src/net"
```

## 变量申明

变量名采用驼峰标准，不要使用_来命名变量名，多个变量申明放在一起
```go
var (
    Found bool
    count int
)
```
在函数外部申明必须使用var,不要采用:=，容易踩到变量的作用域的问题。

## 自定义类型的string循环问题

如果自定义的类型定义了String方法，那么在打印的时候会产生隐藏的一些bug
```go
type MyInt int
func (m MyInt) String() string {
    return fmt.Sprint(m)   //BUG:死循环
}

func(m MyInt) String() string {
    return fmt.Sprint(int(m))   //这是安全的,因为我们内部进行了类型转换
}
```

## 避免返回命名的参数

如果你的函数很短小，少于10行代码，那么可以使用，不然请直接使用类型，因为如果使用命名变量很
容易引起隐藏的bug
```go
func Foo(a int, b int) (string, ok){

}
```
当然如果是有多个相同类型的参数返回，那么命名参数可能更清晰：
```go
func (f *Foo) Location() (float64, float64, error)
```
下面的代码就更清晰了：
```go
// Location returns f's latitude and longitude.
// Negative values mean south and west, respectively.
func (f *Foo) Location() (lat, long float64, err error)
```

## 错误处理

错误处理的原则就是不能丢弃任何有返回err的调用，不要采用_丢弃，必须全部处理。接收到错误，要么返回err，要么实在不行就panic，或者使用log记录下来

### error 信息

error的信息不要采用大写字母，尽量保持你的错误简短，但是要足够表达你的错误的意思。

## 长句子打印或者调用，使用参数进行格式化分行

我们在调用fmt.Sprint或者log.Sprint之类的函数时，有时候会遇到很长的句子，我们需要在参数调用处进行多行分割：

下面是错误的方式：
```go
log.Printf("A long format string: %s %d %d %s", myStringParameter, len(a),
    expected.Size, defrobnicate("Anotherlongstringparameter",
        expected.Growth.Nanoseconds() /1e6))
```
应该是如下的方式：
```go
log.Printf(
    "A long format string: %s %d %d %s",
    myStringParameter,
    len(a),
    expected.Size,
    defrobnicate(
        "Anotherlongstringparameter",
        expected.Growth.Nanoseconds()/1e6,
    ),
）
```
## 注意闭包的调用

在循环中调用函数或者goroutine方法，一定要采用显示的变量调用，不要再闭包函数里面调用循环的参数
```go
fori:=0;i<limit;i++{
    go func(){ DoSomething(i) }() //错误的做法
    go func(i int){ DoSomething(i) }(i)//正确的做法
}
```

参考：http://golang.org/doc/articles/race_detector.html#Race_on_loop_counter

## 在逻辑处理中禁用panic

在main包中只有当实在不可运行的情况采用panic，例如文件无法打开，数据库无法连接导致程序无法
正常运行，但是对于其他的package对外的接口不能有panic，只能在包内采用。

强烈建议在main包中使用log.Fatal来记录错误，这样就可以由log来结束程序。

## 注释规范

```go
// Copyright 2011 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

/*
foo 包实现了一些简单的数学函数，本注释仅做示例用途。
如果你有任何疑问，请添加至邮件列表：
golang-nuts@googlegroups.com.
*/

package foo

import "fmt"

// Foo 比较了两个输入值，并且返回较大的那一个，若值相等，则返回0
func Foo(a, b int) (ret int, err error) {
    if a > b {
        return a, nil
    } else {
        return b, nil
    }
        return 0, nil
    }

// BUG(小明): #1: 不要意思这份代码有些bug
// BUG(小红): #2: 已将一份issue提给了另外一位开发者。
```

在这段代码里，我们添加了4条注释：版权说明注释、包说明注释、函数说明注释和最后添
加的遗留问题说明。这些注释可以方便地通过```go doc```或goland的```quick document```
展示给使用者。

为了使注释可以文档化，必须遵守以下规则：

- 注释需要紧贴在对应的包声明和函数之前，不能有空行。
- 注释如果要新起一个段落，应该用一个空白注释行隔开，因为直接换行书写会被认为是
正常的段内折行。
- 开发者可以直接在代码内用// BUG(author): 的方式记录该代码片段中的遗留问题，这
些遗留问题也会被抽取到文档中。

延伸阅读：https://wizardforcel.gitbooks.io/golang-doc/content/32.html


### 无用代码直接删除

注释无用的代码极易给后续维护代码的人造成误解，因此，已经决定不用的代码不可通过注释的方式保留在包中，直接删除。

如果未来还会用到，则需要加上TODO，并写明保留原因。

### bug注释

针对代码中出现的bug，可以采用如下教程使用特殊的注释，在godocs可以做到注释高亮：
```go
// BUG(astaxie):This divides by zero.
var i float = 1/0
```
http://blog.golang.org/2011/03/godoc­documenting­go­code.html
## struct规范

### struct申明和初始化格式采用多行：

定义如下：
```go
type User struct{
    Username  string
    Email     string
}
```
初始化如下：
```go
u := User{
    Username: "astaxie",
    Email:    "astaxie@gmail.com",
}
```
### recieved是值类型还是指针类型

到底是采用值类型还是指针类型主要参考如下原则：
```go
func(w Win) Tally(playerPlayer)int    //w不会有任何改变
func(w *Win) Tally(playerPlayer)int    //w会改变数据
```
更多的请参考：https://code.google.com/p/go-wiki/wiki/CodeReviewComments#Receiver_Type

### 带mutex的struct必须是指针receivers

如果你定义的struct中带有mutex,那么你的receivers必须是指针

## 参考资料

1.https://code.google.com/p/go-wiki/wiki/CodeReviewComments

2.http://golang.org/doc/effective_go.html
