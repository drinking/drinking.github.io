---
layout: post
title: "Swift递归实现的不重复随机序列生成函数"
date: 2015-02-01 15:11:15 +0800
comments: true
categories: 
---
 
#### 题目来自于《编程珠玑》第一章节的一道题：生成位于0～N-1之间的k个不同的随机序列的随机整数

#### 算法保证在没有生成过数的区间进行随机生成操作，能够满足不重复需求。但是递归原因容易造成随机数的集中。

{% highlight swift %}
func createGenerator(count:Int)->(Int,Int)->[Int]{
    //http://stackoverflow.com/questions/24270693/nested-recursive-function-in-swift
    var generator:(Int,Int)->[Int] = {_,_ in return []} // give it a no-op definition
    var total = count
    generator = {min,max in
        if (total <= 0 || min>max) {
            return []
        }else{
            total--;
            var random = Int(arc4random_uniform(UInt32(max-min)))
            var mid = min + random
            return [mid]+generator(min, mid-1)+generator(mid+1, max)
        }
    }
    
    return generator
}

createGenerator(10)(0,100)
{% endhighlight %}