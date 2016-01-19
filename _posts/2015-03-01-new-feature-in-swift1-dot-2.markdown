---
layout: post
title: "Swift1.2中的新特性小记"
date: 2015-03-01 22:26:33 +0800
comments: true
categories: 
---

##引入集合概念
集合是一个没有重复的无序元素集,以下是一些基本操作

{% highlight swift %}
var set1 = Set(["a", "b", "c", "d"]) //集合创建
set1.insert("e") //集合插入
set1.remove("a") //删除操作
var set2 = Set(["b", "c", "f"])
set1.subtract(set2) //集合差集
//["e", "d"]
set1.union(set2) //集合并集
//["b","e","f","d","c"]
{% endhighlight %}

##if let语句的优化
之前由于optional特性的引入，在做判断时需要层层unwrap，使得代码嵌套严重，影响整洁。

{% highlight swift %}
if let name = validateName(nameTextField.text) {
    if !name.isEmpty {
        if let age = validateAge(ageTextField.text) {
            if age > 13 {
                
                // 做相应操作
            }
        }
    }
}
{% endhighlight %}

在优化if let之后，能够将判断集中处理，代码瞬间精简许多

{% highlight swift %}
if let name = validateName(nameTextField.text),
       age = validateAge(ageTextField.text)
    where age > 13 && !name.isEmpty {
    
                // 做相应操作
}
{% endhighlight %}

##as!强制类型转换
强制类型转换体现了苹果对类型安全的重视，相比as?转换失败返回nil，as!转换失败会引发运行时错误。这就需要程序员在转换时引起注意。

{% highlight swift %}
class A{}
class B:A{}

var a:A? = A()
var b:B? = a as? B //此时b为nil

var c:B? = a as! B //运行时错误，程序崩溃

{% endhighlight %}

另外在从objective－C到Swift基本类型转换时需要显示转换,反之不用

{% highlight swift %}

var nsstr1:NSString = NSString(string: "hello")
var str1:String = nsstr1 as String

var nsarray:NSArray = NSArray()
var array:[AnyObject] = nsarray as [AnyObject]

var nsdict:NSDictionary = NSDictionary()
var dict:Dictionary = nsdict as Dictionary
{% endhighlight %}

Xcode一般会给出类型转换的提示，但是由于beta版的不稳定，导致SourceKit崩溃而无法提示，导致编译过程中出现segment fault问题。以下是出错代码段。

{% highlight swift %}

class func changePassword(oldpassword: String, newpassword: String, doneAction:()->()) {
    
    API_USERS_CHANGE_PASSWORD.cancel()
    var api: API_USERS_CHANGE_PASSWORD = API_USERS_CHANGE_PASSWORD()
    
    //以参数形式构造的字典在Xcode6.3 beta2中会导致SourceKit崩溃
    var req = ["old_password": oldpassword, "new_password": newpassword] as NSDictionary
    //api.req是objective－C经bridge转换成代码，原objective－C中api.req接收NSDictionary的赋值，但是经bridge转换后，需要Dictionary值。
    api.req = req //api.req(Dictionary) = req(NSDictionary) 在赋值时没有编辑器错误警告，导致执行编译，最后出现编译不通过情况
    
    //正确的写法如下，是不需要在通过NSDictionary中转，直接赋值即可
    api.req = ["old_password": oldpassword, "new_password": newpassword]
    api.whenSucceed = {
        [unowned api] in
        doneAction()
    }
    api.whenFailed = {
        println("修改密码失败")
    }
    api.send()
}
{% endhighlight %}


Xcode是自带升级Swfit代码的功能，但是并不完善，甚至有荒谬的转换。所以还需要人工去仔细排查。有时候单纯通过编译器检查并不能确保运行时正确。