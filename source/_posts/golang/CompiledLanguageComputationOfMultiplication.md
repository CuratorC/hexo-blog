---
title: 用计算证明：我远远低估了编译器
description: 当我使用 解释型语言PHP 进行验算 “如何使用PHP最高效率的将一个正整数扩大一千倍” 后却得出了相反的结论:只进行一次运算比进行两次计算要更快。 于是我猜测可能是 编译型语言 与 解释型语言 的差异导致的。围绕这个观点展开了论证。
keywords: Golang, 编译器, 效率优化
top_img: /images/golang/GolangCover2.jpg
cover: /images/golang/GolangCover2.jpg
tags:
  - Golang
  - 探讨
categories:
  - Golang
date: 2021-08-12 17:38:07
updated: 2021-08-12 17:38:07
---
# 用计算证明：我远远低估了编译器

## 前情回顾
在`将正整数扩大1000倍`的计算当中，朋友提出了计算机二进制算法理论：

计算机会进行`正整数 X 512 + 正整数 X 256 + 正整数 X 128 + 正整数 X 128 + 正整数 X 64 + 正整数 X 32 + 正整数 X 8`，一直累加到凑齐 正整数的 1000 倍 为止。

而运算`正整数 X 2的10次方 - ( 正整数 X 16 + 正整数 X 8) `要比那一串加号更快接近结果。

但是当我使用`解释型语言PHP`进行验算 [如何使用PHP最高效率的将一个正整数扩大一千倍？](/php/PhpExpandsPositiveIntegersByAFactorOf1000) 后却得出了相反的结论:只进行一次运算比进行两次计算要更快。

于是我猜测可能是`编译型语言`与`解释型语言`的差异导致的。围绕这个观点展开了论证。

## 验算

### 代码部分

代码价值较低，为不影响阅读，置于[附录](/golang/CompiledLanguageComputationOfMultiplication#附录)

### 运算结果

将代码执行了数十次之后我得出了一个有些出乎意料的结论：方案三对方案二只有微乎其微的优势，甚至可以说两者的计算效率没有差距！

以下贴出两个比较有代表性的结果

![image01](/images/golang/CompiledLanguageComputationOfMultiplication01.png)

![image02](/images/golang/CompiledLanguageComputationOfMultiplication02.png)

## 修正运算

### 改变计算量级
后来我将计算量上升至`10亿次`，得出的结论也差不多。只靠这个结论我很难说服自己方案三比方案二更值得选择。但是运算结果也不合理：两者明明采取了不同的计算方案，为什么无法产生差距？

与其说计算时间没有差距，不如说现有的运行时间全部都消耗在了`for`, `rand`, 和`time`上。既然如此，那么提升计算复杂度，来降低过程类计算的干扰是否会得出结果呢？

### 改变计算复杂度
我将随机数的范围取从 `999` 改为了 `9999999999` ，将乘数从 `1000` 改为了 `8,589,934,590(2的33次方减2)`同样各进行一千万次计算，在其他处理步骤不变的情况下，计算耗时被明显延长，但两种方案的计算时长依旧没能拉开差距。

按照豆豆告诉我的理论，`integer * 8,589,934,590`会被解析`integer * 4,294,967,296 + integer * 2,147,483,648 ...`一直累加30多次来累计到目标乘积，而`integer * 8,589,934,592 - integer * 2`则避免了这样的运算。试验结论却不能支持这个观点。

## 再次讨论
带着这样的数据，我再次找到了豆豆。这次修行更进一步的豆豆同学嘲讽我一番后直接甩给我两篇知乎：
- [现代C/C++编译器有多智能？能做出什么厉害的优化？ - 苏远的回答 - 知乎](https://www.zhihu.com/question/43598164/answer/96118643)
- [现代C/C++编译器有多智能？能做出什么厉害的优化？ - 「已注销」的回答 - 知乎](https://www.zhihu.com/question/43598164/answer/142599680)

## 我的理解
两种方案的计算之所以没有产生时间差距，是因为两种写法经过编译之后，都运行了同样的算法。
* 我：我想让编辑器做一个复杂步骤的计算，再做一个简单步骤的计算，用两种计算的耗时差距来证明计算机的算法。
* 编译器：没想到吧崽种，我早已预判了你的预判！
*
要将编译器的预判囊括入我的预判，这一点`重新升格为大神的豆豆同学`也给出了方案
{% note info flat %}
你把你博客上的代码反汇编成汇编语言，看一看，就知道为什么时间差别不大了
{% endnote %}
但是编译器的算法早已经超出了蹒跚学步初学者的理解范畴。对于当前学习阶段的我而言，通过本次试验和讨论窥探一眼就好，看多了容易掉 SAN 值。日后若窥探到更多的内容，我会再写一篇博客出来，欢迎关注。

## 附录
### 附录代码

```go
func main() {
	// 计算次数
	maxI := 10000000

	fmt.Println("方案1：拼接法")

	// 随机数
	rand.Seed(time.Now().UnixNano())

	echoString := ""

	for count := 0; count < 3; count++ {
		// 开始
		start := time.Now()
		echoString += "第" + strconv.Itoa(count + 1) + "次:"

		for i := 0; i < maxI; i++ {
			integer := rand.Intn(999)
			_, _ = strconv.Atoi(strconv.Itoa(integer) + "000")
		}

		// 结束
		end := time.Now()

		betweenTime := float64(end.Sub(start).Nanoseconds()) / 1000000000

		echoString += strconv.FormatFloat(betweenTime, 'f', 10, 64) + "秒"
	}

	fmt.Println(echoString)
	echoString = ""

	fmt.Println("方案2：乘1000")
	for count := 0; count < 3; count++ {
		// 开始
		start := time.Now()
		echoString += "第" + strconv.Itoa(count + 1) + "次:"

		for i := 0; i < maxI; i++ {
			integer := rand.Intn(999)
			_ = integer * 1000
		}

		// 结束
		end := time.Now()

		betweenTime := float64(end.Sub(start).Nanoseconds()) / 1000000000

		echoString += strconv.FormatFloat(betweenTime, 'f', 10, 64) + "秒"

	}

	fmt.Println(echoString)
	echoString = ""

	fmt.Println("方案3：乘以1024")
	for count := 0; count < 3; count++ {
		// 开始
		start := time.Now()
		echoString += "第" + strconv.Itoa(count + 1) + "次:"

		for i := 0; i < maxI; i++ {
			integer := rand.Intn(999)
			_ = (integer * 1024) - (integer * 24)
		}

		// 结束
		end := time.Now()

		betweenTime := float64(end.Sub(start).Nanoseconds()) / 1000000000

		echoString += strconv.FormatFloat(betweenTime, 'f', 10, 64) + "秒"

	}

	fmt.Println(echoString)

}
```
