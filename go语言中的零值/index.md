# 聊一聊Go语言中的零值，它有什么用？


## 背景

哈喽，大家好，我是`asong`。今天与大家聊一聊Go语言中的零值。大学时期我是一名`C`语言爱好者，工作了以后感觉`Go`语言和`C`语言很像，所以选择了`Go`语言的工作，时不时就会把这两种语言的一些特性做个比较，今天要比较的就是零值特性。熟悉`C`语言的朋友知道在`C`语言中默认情况下不初始化局部变量。 未初始化的变量可以包含任何值，其使用会导致未定义的行为；如果我们未初始局部变量，在编译时就会报警告 C4700，这个警告指示一个`Bug`，这个`Bug`可能导致程序中出现不可预测的结果或故障。而在Go语言就不会有这样的问题，Go语言的设计者吸取了在设计`C`语言时的一些经验，所以`Go`语言的零值规范如下：

以下内容来自官方blog：https://golang.org/ref/spec#The_zero_value

当通过声明或 new 调用为变量分配存储空间时，或通过复合文字或 make 调用创建新值时，且未提供显式初始化，则给出变量或值一个默认值。此类变量或值的每个元素都为其类型设置为零值：布尔型为 false，数字类型为 0，字符串为 ""，指针、函数、接口、切片、通道和映射为 nil。此初始化是递归完成的，例如，如果未指定任何值，则结构体数组的每个元素的字段都将其清零。

例如这两个简单的声明是等价的：

```go
var i int 
var i int = 0
```

在或者这个结构体的声明：

```go
type T struct { i int; f float64; next *T }
t := new(T)
```

这个结构体`t`中成员字段零值如下：

```go
t.i == 0
t.f == 0.0
t.next == nil
```

`Go`语言中这种始终将值设置为已知默认值的特性对于程序的安全性和正确性起到了很重要的作用，这样也使整个`Go`程序更简单、更紧凑。



## 零值有什么用

### 通过零值来提供默认值

我们在看一些`Go`语言库的时候，都会看到在初始化对象时采用"动态初始化"的模式，其实就是在创建对象时判断如果是零值就使用默认值，比如我们在分析`hystrix-go`这个库时，在配置`Command`时就是使用的这种方式：

```go
func ConfigureCommand(name string, config CommandConfig) {
	settingsMutex.Lock()
	defer settingsMutex.Unlock()

	timeout := DefaultTimeout
	if config.Timeout != 0 {
		timeout = config.Timeout
	}

	max := DefaultMaxConcurrent
	if config.MaxConcurrentRequests != 0 {
		max = config.MaxConcurrentRequests
	}

	volume := DefaultVolumeThreshold
	if config.RequestVolumeThreshold != 0 {
		volume = config.RequestVolumeThreshold
	}

	sleep := DefaultSleepWindow
	if config.SleepWindow != 0 {
		sleep = config.SleepWindow
	}

	errorPercent := DefaultErrorPercentThreshold
	if config.ErrorPercentThreshold != 0 {
		errorPercent = config.ErrorPercentThreshold
	}

	circuitSettings[name] = &Settings{
		Timeout:                time.Duration(timeout) * time.Millisecond,
		MaxConcurrentRequests:  max,
		RequestVolumeThreshold: uint64(volume),
		SleepWindow:            time.Duration(sleep) * time.Millisecond,
		ErrorPercentThreshold:  errorPercent,
	}
}
```

通过零值判断进行默认值赋值，增强了`Go`程序的健壮性。



### 开箱即用

为什么叫开箱即用呢？因为`Go`语言的零值让程序变得更简单了，有些场景我们不需要显示初始化就可以直接用，举几个例子：

- 切片，他的零值是`nil`，即使不用`make`进行初始化也是可以直接使用的，例如：

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    var s []string

    s = append(s, "asong")
    s = append(s, "真帅")
    fmt.Println(strings.Join(s, " "))
}
```

但是零值也并不是万能的，零值切片不能直接进行赋值操作：

```go
var s []string
s[0] = "asong真帅"
```

这样的程序就报错了。

- 方法接收者的归纳

利用零值可用的特性，我们配合空结构体的方法接受者特性，可以将方法组合起来，在业务代码中便于后续扩展和维护：

```go
type T struct{}

func (t *T) Run() {
  fmt.Println("we run")
}

func main() {
  var t T
  t.Run()
}
```

我在一些开源项目中看到很多地方都这样使用了，这样的代码最结构化～。

- 标准库无需显示初始化

我们经常使用`sync`包中的`mutex`、`once`、`waitgroup`都是无需显示初始化即可使用，拿`mutex`包来举例说明，我们看到`mutex`的结构如下：

```go
type Mutex struct {
	state int32
	sema  uint32
}
```

这两个字段在未显示初始化时默认零值都是`0`，所以我们就看到上锁代码就针对这个特性来写的：

```go
func (m *Mutex) Lock() {
	// Fast path: grab unlocked mutex.
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
		if race.Enabled {
			race.Acquire(unsafe.Pointer(m))
		}
		return
	}
	// Slow path (outlined so that the fast path can be inlined)
	m.lockSlow()
}

```

原子操作交换时使用的`old`值就是`0`，这种设计让`mutex`调用者无需考虑对`mutex`的初始化则可以直接使用。

还有一些其他标准库也使用零值可用的特性，使用方法都一样，就不在举例了。



## 零值并不是万能

`Go`语言零值的设计大大便利了开发者，但是零值并不是万能的，有些场景下零值是不可以直接使用的：

- 未显示初始化的切片、map，他们可以直接操作，但是不能写入数据，否则会引发程序panic：

```go
var s []string
s[0] = "asong"
var m map[string]bool
m["asong"] = true
```

这两种写法都是错误的使用。

- 零值的指针

零值的指针就是指向`nil`的指针，无法直接进行运算，因为是没有无内容的地址：

```go
var p *uint32
*p++ // panic: panic: runtime error: invalid memory address or nil pointer dereference
```

这样才可以：

```go
func main() {
	var p *uint64
	a := uint64(0)
	p = &a
	*p++
	fmt.Println(*p) // 1
}
```

- 零值的error类型

error内置接口类型是表示错误条件的常规接口，nil值表示没有错误，所以调用`Error`方法时类型`error`不能是零值，否则会引发`panic`：

```go
func main() {
	rs := res()
	fmt.Println(rs.Error())
}

func res() error {
	return nil
}
panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0x1 addr=0x0 pc=0x10a6f27]
```

- 闭包中的nil函数

在日常开发中我们会使用到闭包，但是这其中隐藏一个问题，如果我们函数忘记初始化了，那么就会引发`panic`：

```go
var f func(a,b,c int)

func main(){
  f(1,2,3) // panic: runtime error: invalid memory address or nil pointer dereference
}
```

- 零值channels

我们都知道`channels`的默认值是`nil`，给定一个` nil channel c`:

- `<-c` 从` c` 接收将永远阻塞
- `c <- v` 发送值到` c` 会永远阻塞
- `close(c)` 关闭` c` 引发` panic`

关于零值不可用的场景先介绍这些，掌握这些才能在日常开发中减少写`bug`的频率。



## 总结

总结一下本文叙说的几个知识点：

- `Go`语言中所有变量或者值都有默认值，对程序的安全性和正确性起到了很重要的作用
- `Go`语言中的一些标准库利用零值特性来实现，简化操作
- 可以利用"零值可用"的特性可以提升代码的结构化、使代码更简单、更紧凑
- 零值也不是万能的，有一些场景下零值是不可用的，开发时要注意

**好啦，本文就到这里了，素质三连（分享、点赞、在看）都是笔者持续创作更多优质内容的动力！我是`asong`，我们下期见。**

**创建了读者交流群，欢迎各位大佬们踊跃入群，一起学习交流。入群方式：关注公众号获取。更多学习资料请到公众号领取。**

![](https://song-oss.oss-cn-beijing.aliyuncs.com/golang_dream/article/static/%E6%89%AB%E7%A0%81_%E6%90%9C%E7%B4%A2%E8%81%94%E5%90%88%E4%BC%A0%E6%92%AD%E6%A0%B7%E5%BC%8F-%E7%99%BD%E8%89%B2%E7%89%88-20210717170231906-20210801174715998.png)

推荐往期文章：

- [学习channel设计：从入门到放弃](https://mp.weixin.qq.com/s/E2XwSIXw1Si1EVSO1tMW7Q)
- [详解内存对齐](https://mp.weixin.qq.com/s/ig8LDNdpflEBWlypU1NRhw)
- [[警惕\] 请勿滥用goroutine](https://mp.weixin.qq.com/s/JC14dWffHub0nfPlPipsHQ)
- [源码剖析panic与recover，看不懂你打我好了！](https://mp.weixin.qq.com/s/yJ05a6pNxr_G72eiWTJ-rw)
- [面试官：小松子来聊一聊内存逃逸](https://mp.weixin.qq.com/s/MepbrrSlGVhNrEkTQhfhhQ)
- [面试官：两个nil比较结果是什么？](https://mp.weixin.qq.com/s/CNOLLLRzHomjBnbZMnw0Gg)
- [Go语言如何操纵Kafka保证无消息丢失](https://mp.weixin.qq.com/s/XoSi3Cgp7ij-n9t4pvBoXQ)





