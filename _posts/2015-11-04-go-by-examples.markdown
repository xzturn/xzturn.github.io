---
layout: post
title:  "Go By Examples"
date:   2015-11-04 06:21:01
categories: go
---

最近又重新翻了一遍 [Go][go] 的在线文档和相关文章，发现对 [Go][go] 的[并发机制][concurrency]理解还不是很深，故在 [Github][github] 上建了一个 [repo][go-by-example]，记录一下几种典型的 [go][go] [concurrency][concurrency] patterns。

## 关于 [Closure][closure]
[Go][go] 从语言级支持 [closure][closure]，而一个 [closure][closure] 是一个“函数”，它允许访问函数外部的变量（此时称该函数被“绑定”到了这个变量上） —— 这一点对于实现并发很重要，下面很多所谓的并发模式都是基于这种 [closure][closure] 机制的。下面举一个最简单的 fibonacci 数列的实现：

{% highlight go %}
func fibonacci() func() int {
    x, y := 0, 1
    return func() int {
        z := x + y
        x, y = y, z
        return z
    }
}

func main() {
    f := fibonacci()
    for i := 0; i < 10; i++ {
        fmt.Println(f())
    }
}
{% endhighlight %}

## 几种典型的 [Go][go] 并发模式
1. [Generator](https://github.com/xzturn/go-by-example/blob/master/generator.go)
2. [Timeout](https://github.com/xzturn/go-by-example/blob/master/timeout.go)
3. [Ticker](https://github.com/xzturn/go-by-example/blob/master/ticker.go)
4. [Pipelines and cancellation](https://github.com/xzturn/go-by-example/blob/master/pipeline.go)
5. TBD ...

详见 [repo][go-by-example] 代码吧。

[go]: https://golang.org
[go-by-example]: https://github.com/xzturn/go-by-example
[github]: https://github.com
[concurrency]: https://en.wikipedia.org/wiki/Concurrency_(computer_science)
[closure]: https://en.wikipedia.org/wiki/Concurrency_(computer_science)
