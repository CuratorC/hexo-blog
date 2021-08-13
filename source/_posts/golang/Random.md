---
title: GO 生成随机数
description: 随机数，是使用一个确定性的算法计算出来随机数序。在程序开发中经常需要产生随机数，如随机数验证码登陆、作为唯一身份标识数据等等。Go 中生成随机数的有两个包,分别是 math/rand 和 crypto/rand 
keywords: Golang, 随机数, Random
top_img: /images/golang/GolangCover1.png
cover: /images/golang/GolangCover1.png
tags:
  - Golang
  - 入门 
categories:
  - Golang
date: 2021-08-12 17:31:03
updated: 2021-08-12 17:31:03
---

# GO 生成随机数

## 什么是随机数

随机数，是使用一个确定性的算法计算出来随机数序。在程序开发中经常需要产生随机数，如随机数验证码登陆、作为唯一身份标识数据等等。

### GO中生成随机数的包
Go 中生成随机数的有两个包,分别是`math/rand`和`crypto/rand`
* `math/rand`实现了伪随机数生成器
* `crypto/rand`实现了用于加解密的跟安全的随机数生成器,当然,性能也就降下来了,毕竟鱼与熊掌不可兼得

## 随机数生成

```go
func main() {
	for i:=0; i<10; i++ {
		fmt.Print(rand.Intn(10), " ")
	}
}
```

* 结果: 1 7 7 9 1 8 5 0 6 0
* 再来一次: 1 7 7 9 1 8 5 0 6 0
* 还是这个结果,这叫啥子随机数,查看文档,继续试验

### 初始化随机种子函数
```go
func Seed(seed int64)
```
* 官方文档:

{% note info flat %}
Seed uses the provided seed value to initialize the default Source to a deterministic state. If Seed is not called, the generator behaves as if seeded by Seed(1). Seed values that have the same remainder when divided by 2^31-1 generate the same pseudo-random sequence. Seed, unlike the Rand.Seed method, is safe for concurrent use.
{% endnote %}

* 我的理解：
* 
{% note info flat %}
系统每次都会先用Seed函数初始化系统资源，如果用户不提供seed参数，则默认用seed=1来初始化，这就是为什么每次都输出一样的值的原因,而且,Seed方法是并发安全的.
所谓种子,通俗理解可以理解为一个抽奖的奖池,我们自定义一个奖池,从我们的奖池中进行随机抽奖,种子就是我们奖池中的数据。
{% endnote %}
* 
### 使用种子生成随机数

* 我们一般使用系统时间来进行初始化
```go
rand.Seed(time.Now().UnixNano())
for i:=0; i<10; i++ {
	fmt.Print(rand.Intn(10), " ")
}
```

* 或者我们可以使用rand.NewSource()
```go
r := rand.New(rand.NewSource(time.Now().UnixNano()))
num := r.Intn(128)
```

### Intn函数
* 生成 [0,n)区间的一个随机数（注意：不包括n）
```go
func Intn(n int) int
```

### 生成指定位数的随机数
* 以生成8位为例
```go
s := fmt.Sprintf("%08v", rand.New(rand.NewSource(time.Now().UnixNano())).Int63n(100000000))
fmt.Println(s)
```

## 真随机数
* 如果我们的应用对安全性要求比较高，需要使用真随机数的话，那么可以使用 crypto/rand 包中的方法,这样生成的每次都是不同的随机数。
```go
package main
import (
    "crypto/rand"
    "fmt"
    "math/big"
)
func main() {
    // 生成 10 个 [0, 128) 范围的真随机数。
    for i := 0; i < 10; i++ {
        result, _ := rand.Int(rand.Reader, big.NewInt(128))
        fmt.Println(result)
    }
}
```

> [转载链接](https://blog.csdn.net/study_in/article/details/102919019)