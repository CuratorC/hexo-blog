---
title: 位移符效率对比
description: 以前总是看到位移符。因为它总是能轻易把一个数字变成我不认识的模样，所以我也没有深入了解过。在看到讨论区留言后才意识到：自己的格局太小了。简单读了读 位移符 的文档和实现原理，我觉得这种方案还是值得一试的。
keywords: PHP, 位移符, 效率对比
top_img: /images/php/PHPCover2.png
cover: /images/php/PHPCover2.png
tags:
  - PHP
  - 探讨
categories:
  - PHP
date: 2021-08-12 16:36:13
updated: 2021-08-12 16:36:13
---
# 位移符效率对比

## 讨论
在上一篇博客『[如何使用PHP最高效率的将一个正整数扩大一千倍？](/php/PhpExpandsPositiveIntegersByAFactorOf1000)』的讨论区，有人提出位移符应该才是运算最快的方案。

以前总是看到位移符`<<`这样的符号。因为它总是能轻易把一个数字变成我不认识的模样，所以我也没有深入了解过。

在看到讨论区留言后才意识到：自己的格局太小了。

简单读了读`位移符`的文档和实现原理，我觉得这种方案还是值得一试的。按照之前两篇博客的心得，我首先抛出我的猜测：
* 在`PHP`中，位移符方案效率要高于`*1024`，但是会低于`*1000`。
* 在`C#`中，位移符的效率与其他两种方案持平。
    * 论证过程见[『Golang』用计算证明：我远远低估了编译器](/go/CompiledLanguageComputationOfMultiplication)

现在进行一个简单的验证。
* 方案一：$integer * 1000
* 方案二：($integer << 10) - ($integer * 24)
* 方案三：($integer * 1024) - ($integer * 24)

## PHP
### 代码部分

这次吸取讨论区提到的“效率权重”，将随机数部分转移出来。

```php
<?php

namespace App\Console\Commands;

use Carbon\Carbon;
use Illuminate\Console\Command;

class DemoCommand extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'demo:test';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Command description';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return int
     */
    public function handle()
    {
        // 图表内容
        $headers = ['次数', '方案1：乘1000', '方案2：移位符', '方案3：乘以 1024', '补充测试：仅位移'];
        $data = [
            [0 => '第一次'],
            [0 => '第二次'],
            [0 => '第三次']
        ];

        // 随机数提出
        $integer = rand(1, 999);

        // 每个方法执行三次
        for ($count = 0; $count < 3; $count++) {
            $start = Carbon::now()->getPreciseTimestamp();
            for ($i = 0; $i < 10000000; $i++) {
                $result = $integer * 1000;
            }
            $end = Carbon::now()->getPreciseTimestamp();
            $data[$count][] = ($end - $start) / 1000000 . '秒';
        }

        for ($count = 0; $count < 3; $count++) {
            $start = Carbon::now()->getPreciseTimestamp();
            for ($i = 0; $i < 10000000; $i++) {
                $result = ($integer << 10) - ($integer * 24);
            }
            $end = Carbon::now()->getPreciseTimestamp();
            $data[$count][] = ($end - $start) / 1000000 . '秒';
        }

        for ($count = 0; $count < 3; $count++) {
            $start = Carbon::now()->getPreciseTimestamp();
            for ($i = 0; $i < 10000000; $i++) {
                $result = ($integer * 1024) - ($integer * 24);
            }
            $end = Carbon::now()->getPreciseTimestamp();
            $data[$count][] = ($end - $start) / 1000000 . '秒';
        }

        for ($count = 0; $count < 3; $count++) {
            $start = Carbon::now()->getPreciseTimestamp();
            for ($i = 0; $i < 10000000; $i++) {
                $result = $integer << 10;
            }
            $end = Carbon::now()->getPreciseTimestamp();
            $data[$count][] = ($end - $start) / 1000000 . '秒';
        }
        $this->table($headers, $data);
    }
}

```

### 运算结果

多次运行并且调整前后顺序，均得到一个较为稳定的结果：

![image01](/images/php/DisplacementEfficiencyComparison01.png)

这个结果在意料之中，但又不完全在。位移符`<<`从理论上来讲和一般的乘法有着本质上的不同，应该有不错的效率提升。它之所以会比`$integer * 1000`效率低，应该是被第二步运算` - $integer * 24`拖累了，但是`位移符`却也没有与`*1024`拉开差距。为了证实这一点，我又补上了单独进行位移计算的测试。结果证明：

{% note info flat %}
位移符在`PHP`这里就仅仅是一个普通的乘除法计算而已。
{% endnote %}

## GO
### 代码部分

```go
func main() {
	// 计算次数
	maxI := 10000000

	rand.Seed(time.Now().UnixNano())
	integer := rand.Intn(999)

	echoString := ""

	fmt.Println("方案1：乘1000")
	for count := 0; count < 3; count++ {
		// 开始
		start := time.Now()
		echoString += "第" + strconv.Itoa(count+1) + "次:"

		for i := 0; i < maxI; i++ {
			_ = integer * 1000
		}

		// 结束
		end := time.Now()

		betweenTime := float64(end.Sub(start).Nanoseconds()) / 1000000000

		echoString += strconv.FormatFloat(betweenTime, 'f', 10, 64) + "秒"

	}

	fmt.Println(echoString)

	echoString = ""

	fmt.Println("方案2：位移符")
	for count := 0; count < 3; count++ {
		// 开始
		start := time.Now()
		echoString += "第" + strconv.Itoa(count+1) + "次:"

		for i := 0; i < maxI; i++ {
			_ = (integer << 10) - (integer * 24)
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
		echoString += "第" + strconv.Itoa(count+1) + "次:"

		for i := 0; i < maxI; i++ {
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
### 运算结果

运行结果同样不出所料。

![image01](/images/php/DisplacementEfficiencyComparison02.png)

不过横向对比一下，`PHP`的计算效率被`GO`秒的渣渣都不剩了。身为一名从`PHP`入行编程，并且现在的主要技术栈和生产能力依旧在`PHP`身上的我，不免产生一种`好可怜啊`的悲凉。

{% note warning flat %}
位移符可以帮助我们理解计算机的原理，但是在`GO`这类编译型语言这里，编译器已经帮你完成了位移符能完成的计算。
{% endnote %}