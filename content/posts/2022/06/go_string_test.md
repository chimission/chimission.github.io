---
title: "GO字符串拼接性能分析"
date: 2022-06-27
draft: false
tags: [ "go", "performance",]
archives: ["2022", "2022-06",]
categories: [ "go", ] 
description: "基准测试"
---
>本文将对GO常见的几种字符串拼接方法做基准测试，并总结它们各自的使用场景
 <!--more-->
如果你想在GO中拼接多个字符串成一个，GO提供本身和标准库了多种实现方式，下面几种比较常用的方式

1. `+` 运算符
2. string.Join
3. bytes.Buffer
4. bytes.Builder

可以看到有很多种方式可以实现字符串拼接，比较灵活，但是多种选择也会对使用者造成一定程度的困扰，不清楚哪种方式才是最符合自己的使用场景，所以本篇文章将探讨以上常用的四种字符串拼接的性能，找出它们适用的场景。

## 如何测试性能

既然我们要比较这几种拼接方式，那么我们该怎么测试它们的性能呢？

这里我们使用 go 语言自带的 `Benchmark【基准测试】`

Benchmark 可以统计出函数的执行时间和内存分配等信息，并且提供了丰富的命令行参数用来定制测试需求。

```go
func BenchmarkTestFunc(b *testing.B) {
	for i := 0; i < b.N; i++ {
		TestFunc()	}
}
```

拿上边这个例子说明，go会在一定时间内循环调用 TestFunc 并统计调用的资源使用，最后输出

```go
BenchmarkTestFunc     9180396     218.8 ns/op    56 B/op   3 allocs/op
```

第1列是函数名

第2例表明这次测试一共调用了多少次TestFunc

第3列表明调用一次平均花费时间

第4列表明调用一次平均内存占用

第5列表明调用一次平均内存分配次数

我们这里只是使用Benchmark 用于性能测试，暂且不讨论Benchmark的使用细节，感兴趣的同学可以看这里

## 开始测试

[完整项目代码](https://github.com/chimission/gostringbenchmark)

### + 运算

```go
func StringPlus() string {
	return "GO" + "字符串" + "链接" + "速度" + "测试" + "!"
}
```

首先是 最简单的`+` 运算符，几乎所有的语言都实现了使用 `+` 运算。

按照经验，一般来说 + 运算拼接字符串的效率是最低的，因为字符串是不可变数据结构，每次+都会内存分配生成新的字符串，比如上边的函数进行了5次+，会有5次内存分配。

但是真的所有的语言都遵行这个规则吗， 每次+运算都会内存分配吗？让我们来看下基准测试

```go
func BenchmarkStringPlus(b *testing.B) {
	for i := 0; i < b.N; i++ {
		StringPlus()
	}
}
```

```go
BenchmarkStringPlus           1000000000               0.3160 ns/op          0 B/op          0 allocs/op
```

哎？可以看到结果，如丝般畅滑，竟然一次内存分配都没有，而且执行时间也很短。难道go的性能如此强悍吗，其实不然，不要忘了go是一门编译语言，在执行前会进行编译，而go的编译优化也做的不错，对于这种字符串+拼接会提前进行编译优化，如果结果是可以提前预知的就会被优化，上述 "GO" + "字符串" + "链接" + "速度" + "测试" + "!" 会被优化为 `"GO字符串链接速度测试!"<span> </span>`所以编译后的函数并没有任何运算而是直接返回了整个字符串，怪不得会这么快。

所以我们需要绕过编译器优化，上面提到了如果结果是可以提前预知的就会被优化，那么我们就需要构造一个不能被编译器提前预知得字符串

```go
func StringPlus2(p []string) string {
	var s string
	l := len(p)
	for i := 0; i < l; i++ {
		s += p[i]
	}
	return s
}

const STRING = "GO字符串链接速度测试!"

func initStrings(N int) []string {
	s := make([]string, N)
	for i := 0; i < N; i++ {
		s[i] = STRING
	}
	return s
}
func BenchmarkStringPlus2(b *testing.B) {
	p := initStrings(100)
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		StringPlus2(p)
	}
}
```

这里我们重新定义了StringPlus2 函数，将字符串通过函数参数传入，这样编译器就没有办法提前优化了。

我们还定义了 initStrings 函数， 用来构造一个长度很长的字符串数组，这个函数也会重用在其他测试里。

让我们看下没有了优化器编译后的结果

```go
10000            100171 ns/op          161840 B/op         99 allocs/op
```

可以看到没有了编译器优化 + 运算就原形毕露了，在100个字符串的相加运算中，进行了99次内存分配，并且其他参数也不乐观，在线上环境完全不能接受。

### string.Join

go的string标准库提供了join函数，可以把给定的字符串数组拼接成一个新的字符串，同样的一般其他语言也有类似的实现。

```go
func StringJoin() string {
	stringList := []string{"GO", "字符串", "链接", "速度", "测试", "!"}
	return strings.Join(stringList, "")
}

func StringJoin(p []string) string {
	return strings.Join(p, "")
}
```

结果如下

```go
BenchmarkStringJoin     9582951       175.3 ns/op      32 B/op      1 allocs/op
```

可以看到 join函数只进行了一次内存分配，并且执行效率也还不错。join内部实现并没有傻乎乎的每个字符子串都去开辟内存，而是拿到所有需要拼接的字符串计算出需要的内存大小后只执行一次内存分配。

这里可能会有同学疑问，不论join参数里的字符串数组大小有多少元素，join都只会执行一次内训分配吗，让我们来实验一下

```go
func StringJoin2(p []string) string {
	return strings.Join(p, "")
}

func BenchmarkStringJoin2(b *testing.B) {
	p := initStrings(100)
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		StringJoin2(p)
	}
}
```

这里我们重用了上一节使用的 initStrings

```go
BenchmarkStringJoin2     409122        3008 ns/op         3072 B/op      1 allocs/op
```

可以看到我们将数组长度增加了到了100个，也确实还是值分配了一次内存。

### bytes.Buffer

但是毕竟 join函数还是不够灵活，需要提前拿到所有的字符串组装成字符串数组作为参数，如果有一边拼接一边其他操作的需求就无能为力了，所以go标准库又提供了另外一种实现，那就是`bytes.Buffer`

```go
func StringBuffer() string {
	var b bytes.Buffer
	b.WriteString("GO")
	b.WriteString("字符串")
	b.WriteString("链接")
	b.WriteString("速度")
	b.WriteString("测试")
	b.WriteString("!")
	return b.String()
}
func BenchmarkStringBuffer(b *testing.B) {
	for i := 0; i < b.N; i++ {
		StringBuffer()
	}
}
```

首先我们先声明一个 Buffer 对象， 调用此对象的 WriteString 方法传入需要拼接的字符串，最后调用 String 方法输出结果。

```go
BenchmarkStringBuffer      8894923       200.1 ns/op         96 B/op      2 allocs/op
```

可以看到 bytes.Buffer 结果也还不错，稍微比join要慢一点。

让我们也测试下大量的字符串拼接场景

```go
func StringBuffer2(p []string) string {
	var b bytes.Buffer
	l := len(p)
	for i := 0; i < l; i++ {
		b.WriteString(p[i])
	}
	return b.String()
}
func BenchmarkStringBuffer2(b *testing.B) {
	p := initStrings(100)
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		StringBuffer2(p)
	}
}
```

同样的，这里测试了拼接100个字符串的场景，结果如下

```go
BenchmarkStringBuffer2      211426         8234 ns/op      15168 B/op        8 allocs/op
```

好像并不是太好，和`+`号，Join拼接差好远，内存分配也比较多。每次操作耗时也很长。

### bytes.Builder

既然有了 bytes.Buffer， 那 bytes.Builder 又是做什么的呢，其实 bytes.Builder 是 bytes.Buffer 的优化版，在go 1.10中被加入到标准库，它的使用和 bytes.Buffer 几乎一样。

```go
func StringBuilder() string {
	var b strings.Builder
	b.WriteString("GO")
	b.WriteString("字符串")
	b.WriteString("链接")
	b.WriteString("速度")
	b.WriteString("测试")
	b.WriteString("!")
	return b.String()
}
func BenchmarkStringBuilder(b *testing.B) {
	for i := 0; i < b.N; i++ {
		StringBuilder()
	}
}
```

可以看到 bytes.Builder 在使用上和 bytes.Buffer 几乎一模一样。

```go
BenchmarkStringBuilder     9180396      218.8 ns/op       56 B/op     3 allocs/op
BenchmarkStringBuffer      8894923       200.1 ns/op         96 B/op      2 allocs/op
```

可以看到的确有提升了，虽然每次分配的内存次数有点多，但是每次分配的内存大小比buffer要少。但是执行时间没有明显优化，难道是我们的场景太简单？让我们试下复杂场景。

```go
func StringBuilder2(p []string) string {
	var b strings.Builder
	l := len(p)
	for i := 0; i < l; i++ {
		b.WriteString(p[i])
	}
	return b.String()
}

func BenchmarkStringBuilder2(b *testing.B) {
	p := initStrings(100)
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		StringBuilder2(p)
	}
}
```

这里同样使用了100个元素的字符串数组，结果如下

```go
BenchmarkStringBuffer2       211426          8234 ns/op     15168 B/op          8 allocs/op
BenchmarkStringBuilder2      158857          6506 ns/op     10464 B/op         10 allocs/op
```

可以看到，Builder 的性能确实比 Buffer 要好，不仅速度快，而且每次执行的平均内存大小也要小，不过会多一两次的内存分配调用。

那么 `Buffer<span> </span>`优化了哪里？

`bytes.Buffer` 和 `bytes.Builder` 底层都使用了 byte数组 作为缓冲区存储，在go中 正常的byte数组转换为string是有一定性能花费的，需要开辟新的内存区域，将byte数组里的内容一个个copy到新内存里。

但是 bytes.Builder 在 byte数组转换为string 这一步做了一些 hack处理，直接使用指针运算做的转换，避免了内存分配和数据拷贝。

```go
func (b *Buffer) String() string {
	if b == nil {
		// Special case, useful in debugging.
		return "<nil>"
	}
	return string(b.buf[b.off:])
}

func (b *Builder) String() string {
	return *(*string)(unsafe.Pointer(&b.buf))
}
```

## 总结

结果汇总

```go
BenchmarkStringPlus           1000000000               0.3160 ns/op          0 B/op          0 allocs/op
BenchmarkStringJoin          9582951               175.3 ns/op            32 B/op          1 allocs/op
BenchmarkStringBuffer          8894923               200.1 ns/op            96 B/op          2 allocs/op
BenchmarkStringBuilder         9180396               218.8 ns/op            56 B/op          3 allocs/op
BenchmarkStringPlus2             10000            100171 ns/op          161840 B/op         99 allocs/op
BenchmarkStringJoin2            409122              3008 ns/op            3072 B/op          1 allocs/op
BenchmarkStringBuffer2          211426              8234 ns/op           15168 B/op          8 allocs/op
BenchmarkStringBuilder2         158857              6506 ns/op           10464 B/op         10 allocs/op
```

1. `+` 连接适用于短小的、常量字符串（明确的，非变量），因为编译器会给我们优化。
2. `Join`是比较统一的拼接，不太灵活，但是性能最好，能用的时候就用。
3. `buffer`现版本基本上不再推荐，已经被`builder`代替
4. `builder`从性能和灵活性上，都是不错的选择，在一些需要一遍拼接一边进行其他操作的时候优先使用

[完整项目代码](https://github.com/chimission/gostringbenchmark)
