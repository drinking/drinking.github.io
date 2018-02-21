---
layout: post
title: Functional Programming
date: 2018-02-03 14:30:41 +0800
comments: true
categories: 
---
注：这是一篇为分享准备的演讲稿，相比现场会有些精简。同时因为有不同技术栈的工程师，所以展示代码中既有Java又有Swift。演讲的后半部分是关于Monad的现场编程，没有在本文中展示出来。

什么是函数式编程？根据接触的深浅不同，每个人心里都有各自的理解，有的人认为链式调用就是函数式编程，有的认为闭包或者柯里化是。所以到底是什么呢？这篇文章就来谈一谈我自己的浅薄理解。

## Trending
通过查看[TIOBE](https://www.tiobe.com/tiobe-index/)指数我们发现2016年后主流语言Java和C以及C++有一个明显的下降趋势，Go是当年的年度语言，Swift、Kotlin、Clojure等具有函数式能力的语言有大幅的增长。那时候特别明显印象就是去参加各种技术讲座，几乎都充斥着函数式主题的内容。

## Lambda
Lambda最早是从数学概念中引入，λ演算是一套从数学逻辑中发展，以变量绑定和替换的规则，来研究函数如何抽象化定义、函数如何被应用以及递归的形式系统。它由数学家阿隆佐·邱奇在20世纪30年代首次发表。并且被作为函数式语言的基础，Lisp、Haskel都深受其影响。

Lambda expression，Anonymous function和Closure是我们在函数式语言里经常看到的相似概念，那么他们之间到底有什么差异性呢？
Lambda expression（Lambda表达式）和Anonymous function（匿名函数）是一个等价的概念。而Closure（闭包）呢，这个在js当中听的最多，是指一段包含上下文的匿名函数。

````python
//Lambda
map( lambda x: x*2, [y for y in range(10)])

//Closure
def multiplier(num):
    return lambda x: x*num
map(multiplier(2), [y for y in range(10)])
````

如例子所示，multiplier中的num不是以参数引入，而是作为一个上下文变量直接引用，从而形成闭包。通常引入后的数据是不可变的。

## Conceptions
在函数式中，总是有那么多相似的概念，却因为宿主语言的不同有各自的叫法或者使用方式。因此在学习过程中，很容易造成混淆和困惑。下面我们就来介绍一下函数式的几个基本特征。
## First Class
首先最基本的概念，也就是函数式的立身之本——First Class（一等公民）。作为语言的基础数据类型，和Int，Float，String，面向对象里的各类对象一样，能够作为参数传入和返回，并且能够作为一个变量，或者存储到一个数据结构里。

## Higher order
其实当你在参数中或者返回值中使用函数时，它就已经是高阶函数了。Map、Reduce、Filter这些常见的函数都是，以至于习以为常。以至于我一直认为有嵌套三层以上的才算高阶的错误认识。
## Currying
提到高阶就不得不提柯里化，把低阶函数升高，使得接受多个参数的函数变换成接受一个单一参数的函数。可以用来避免类似的函数重复声明，或者惰性求值。柯里化过程是一个体力劳动。Swift有一个[Currying](https://github.com/thoughtbot/Curry/blob/master/Source/Curry.swift)的开源库。它所做的就是一步一步升阶。
柯里是个逻辑学家，为了表彰他的贡献，用他的名字命名这个升阶行为。对于初学的人很难将升阶和Currying联系起来。柯里化的优点是延迟计算，动态创建函数以及模块化函数和重用代码。

## Pure and Immutable
函数式另一个显著的特征就是纯粹性和不变性。就如同一个数学函数，固定的输入必然会有唯一且固定的返回值。纯函数就应该避免依赖或者修改上下文。

````swift
var other = [Int]()
[12, 67, 1, 34, 9, 78, 6, 31].map { (i) -> Int in
    if i < 50 {
        other.append(i*2)
    }
    return i
}
````

上面代码有两个问题，用map来代替遍历操作，只是为了省事做遍历，并没有用到其映射概念，这属于一种误用。发现这种问题出现的时候，还是老老实实用for循环，避免为了用而用的情况。另一种就是涉及到对外操作，写入数组。单线程还好，如果多线程就不安全了。

没有可变共享状态时，函数可以有效、安全地并行执行。所以当你需要处理非常大一组数据时，每个处理方法是相对独立且不可变，没有副作用的，所以就可以放心的并行去处理了。

## Map Reduce and Filter
Map、Reduce和Filter简直是函数式的三板斧。但凡翻开一本函数式书籍的前几章，必然能看见这几个熟悉的字眼。，相比某些语言的Limit，Skip，Peek这些功能，Map、Reduce、Filter是更具有普遍意义。程咬金凭借三板斧照样可以在隋唐吃得开，所以善用这些功能依旧能大幅改善编程效果。下面来看Java例子：

````java
String label = "满五唯一 满二 独家 地铁房";
String[] tagArray = label.split(" ");
for (String tag : tagArray) {
    if(StringUtil.equals(tag, TagsEnum.IS_FIVE_SOLE.value())){
        tags.add(TagsEnum.IS_FIVE_YEAR.value());
        continue;
    }
    if(!tag.equals(TagsEnum.IS_QUICK_ACTING.value())){
        tags.add(tag);
    }
}
````

这是一段标签筛选的的代码，for循环下两个if，需要凝神读一下，才能分清是做了两件事。
第一件，如果标签是“满五唯一”，则把它替换为“满五”，添加入数组。
第二件，如果标签不是“独家”，就把它添加数组。

````swift
//Swift 代码
extension Array {
    func filter(includeElement: Element -> Bool) -> [Element] {
        var result: [Element] = []
        for x in self where includeElement(x) {
            result.append(x)
        }
        return result
    }
}
````

再改进之前，让我们先把其中的Filter拎出来，看看它到底做了什么。因为函数称为一等公民之后，我们将精确描述过滤条件那段代码暴露出去，就是一个过滤过程的最小变化单元。你只要实现这个唯一变化的部分，就能满足一个filter功能。另一个常见的Sort函数也是如此，你不再需要关心底层的实现，语言层面肯定会帮你实现最优的排序方式。你只需要关心两个值的大小，这在代码复用层面有很好的效果。
Java在没有Lambda之前的版本也能达到同样的目的，需要将过滤条件封装成类的一个方法， 传递该类的一个实例，但这种方案却很难推广，因为它通常非常臃肿，既难于编写，也不易于维护。

````java
String label = "满五唯一 满二 独家 地铁房";
String tagLabel = Arrays.asList(label.split(" "))
    .stream()
    .filter(t -> !t.equals("独家"))
    .map(t -> t.equals("满五唯一") ? "满五" : t )
    .reduce("Tags:", (sum,tag)-> sum+" "+tag);
````

如果用函数式改进一下，我们来看一看差异。在Java里，首先把数组转化为Stream(),这样就具备了MapReduce能力。具体细节稍后详谈。在iOS的开发语言Objective-C中默认也不支持这种操作，由某些第三方框架实现称之为sequence的对象。这两种叫法都有种数据流或流水线的感觉，让数组中的数据可以依次被操作。让我们来具体看一下代码，Stream之后做了三件事。
第一件，过滤出不是独家的标签。
第二件，把“满五唯一”映射为“满五”。
第三件，汇聚标签，以空格分隔。
通过对比两段操作可以较为清晰地发现，声明式和命令式的特征。
 
## Imperative vs declarative
![declarative](http://www.vaikan.com/wordpress/wp-content/uploads/2013/06/Imperative-vs-Declarative-560x345.png)
可以从上图的走势看出来，如果一段代码抽象的程度越高，则越是趋于声明式，也就是趋于描述做了什么。而代码抽象程度越低，也就趋于命令式，也就是强调怎么做。从纵向来看，what就是对how的一层封装。如果what是上班，how里面就包含起床，洗脸刷牙，做地铁等一系列流程。

正如高级语言比机器语言更抽象，函数式语言比命令式语言更抽象。Map,Reduce,Filter都更加具有语言表现力，它们本身就能代表意图。而DSL相比高级语言就更加趋向于口语化的表达。正如Domain-specific language领域特定语言名称所预示着你不能用Markdown去查数据库，也不能用SQL去写网页，这些你都能用高级语言做，如果你真这么去做了，一定会崩溃的。

## A Joke
如果是普通的数据类型，并不能起到支持链式调用的能力，像下面这段代码，filter外面嵌套map，再外面嵌套reduce，可读性也不好。

````python
reduce(lambda x,y: x+y, map(lambda x:10**2, filter(lambda x: x%2==0,range(10))))
````

之前有个段子，据说一个黑客偷到了美国用于导弹控制的LISP代码的最后一页，却发现这最后一页竟然是？一整页的括号）））））））））））））））。所以为了解决嵌套地狱以及一致性，我们引入了Optional。
## Optional

Optional<T>（T为范型，任一类型）是一个容器对象，是对真正值的一层封装。表示一个Optional对象里面，可能存在这个值（T），也可能不存在（none）。并且Optional中有方法来明确处理值不存在的情况，如果你能一致地使用它的话,出现处理异常时可以统一用none来表示，就可以避免其他异常引起的副作用。。针对Optional的操作有一堆很重要的概念，Functor、Applicative以及Monad。有很多[文章](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)描述，这里就不累述了。

## Summary
- 一等公民和高阶函数，增强代码灵活性。我们可以通过函数来创建函数，实现更好的复用。
- 纯函数和不可变性，提高了代码的稳定性，可测试，拥有并行计算。
- Optional数据处理具有了一致性，定义具有广泛应用场景的函数，mapreducefilter，增加代码可读性。同时使得链式调用成为可能。

所以什么是函数式编程？
依托函数的特性，来实现稳定，可读、灵活复用的代码的一种编程思维。函数式编程不是万能药，选择合适的使用场景解决实际问题才是最重要的。

## Reference
- https://www.zhihu.com/question/64337143
- http://blog.leichunfeng.com/blog/2015/11/08/functor-applicative-and-monad/
- https://zh.wikipedia.org/wiki/%E5%B7%A6%E9%81%9E%E6%AD%B8
- http://www.ruanyifeng.com/blog/2015/07/monad.html
- https://zh.wikipedia.org/wiki/%E4%B8%8D%E5%8A%A8%E7%82%B9%E7%BB%84%E5%90%88%E5%AD%90
- https://clojurefun.wordpress.com/2012/08/27/what-defines-a-functional-programming-language/
- https://www.zhihu.com/question/20125256
- https://www.tiobe.com/tiobe-index/
- https://www.tiobe.com/tiobe-index/programming-languages-definition/
- https://octoverse.github.com/
- https://en.wikipedia.org/wiki/History_of_programming_languages
- https://www.reddit.com/r/golang/comments/68aix3/why_did_you_choose_go/
- http://erlang.org/pipermail/erlang-questions/2015-May/084509.html

