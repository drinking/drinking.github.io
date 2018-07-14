---
title: Mantle源码阅读笔记
date: 2016-01-25 16:58:58 +0800
layout: post
current: post
cover:  assets/images/welcome.jpg
navigation: True
tags: [source code mantle]
class: post-template
subclass: 'post tag-getting-started'
author: Drinking
comments: true
---


Mantle是一个为`Cocoa`和`Cocoa Touch`的Model层提供`JSON`和`Object`之间转换的框架，因其提供了较其它框架更为丰富的功能而被广泛应用。下面逐一谈下主要特性的实现原理及我的收获。

#### 相同属性名的映射

Mantle通过`MTLModel`协议规定了一个可转换对象应当具有的行为，并在`MTLModel`类中给出了实现。`MTLModel`类虽然给出了协议的所有实现，但是本身并没有用到。只是通过KVC的方式提供了`NSDictionary`到`Model`的映射，但是处理得非常谨慎，考虑到了内存泄露的问题，可以参考以下代码。

```objc

static BOOL MTLValidateAndSetValue(id obj, NSString *key, id value, BOOL forceUpdate, NSError **error) {
	// Mark this as being autoreleased, because validateValue may return
	// a new object to be stored in this variable (and we don't want ARC to
	// double-free or leak the old or new values).
	__autoreleasing id validatedValue = value;

	@try {
		if (![obj validateValue:&validatedValue forKey:key error:error]) return NO;
		if (forceUpdate || value != validatedValue) {
			[obj setValue:validatedValue forKey:key];
		}
		return YES;
	} @catch (NSException *ex) {
		NSLog(@"*** Caught exception setting key \"%@\" : %@", key, ex);
		// Fail fast in Debug builds.
		#if DEBUG
		@throw ex;
		#else
		if (error != NULL) {
			*error = [NSError mtl_modelErrorWithException:ex];
		}
		return NO;
		#endif
	}
}

```

`MTLJSONSerializing`协议继承了`MTLModel`协议并加入了三个重要的拓展，大大增加了灵活性。

- JSONKeyPathsByPropertyKey 使得JSON中的Key和Property的名称不用一样，通过一个字典找到对应的名称即可。
- JSONTransformerForKey 可以自定义Value的转换方式，比如一个JSON中的dateString可以和一个NSDate对象相互转换。
- classForParsingJSONDictionary 将JSONDictionary转换成换指定的另一个类的对象。



#### property的类型
Mantle通过定义`MTLPropertyStorage`这个枚举，将`property`分为三种类型。有一种没有用到(也可能是随着版本迭代而弃用了)，实际上最后将`property`分为可储值和不可储值的。不可储值的`property`没有变量空间`ivar`，`readonly`属性的变量容易出现这种情况。下面的代码给出了具体的实现，思维的缜密性在于考虑到了子类可能覆盖父类的`property`属性，一个被覆盖的`readonly`的属性可能在其父类中存在变量空间。

```objc

+ (MTLPropertyStorage)storageBehaviorForPropertyWithKey:(NSString *)propertyKey {
	objc_property_t property = class_getProperty(self.class, propertyKey.UTF8String);

	if (property == NULL) return MTLPropertyStorageNone;

	mtl_propertyAttributes *attributes = mtl_copyPropertyAttributes(property);
	@onExit {
		free(attributes);
	};
	
	BOOL hasGetter = [self instancesRespondToSelector:attributes->getter];
	BOOL hasSetter = [self instancesRespondToSelector:attributes->setter];
	if (!attributes->dynamic && attributes->ivar == NULL && !hasGetter && !hasSetter) {
		return MTLPropertyStorageNone;
	} else if (attributes->readonly && attributes->ivar == NULL) {
		if ([self isEqual:MTLModel.class]) {
			return MTLPropertyStorageNone;
		} else {
			// Check superclass in case the subclass redeclares a property that
			// falls through
			return [self.superclass storageBehaviorForPropertyWithKey:propertyKey];
		}
	} else {
		return MTLPropertyStoragePermanent;
	}
}

```

#### 获取propertyKeys 
获取`property`名称的方式自然是用到了`runtime`的特性，逐次遍历`class_copyPropertyList`得到的所有属性，然后通过上面提到的类型筛选出可以真正赋值的变量名集合。值得注意的是，为了避免相同类的每个对象都重复这个操作，`Mantle`动态为[self class]类对象添加属性，作为缓存。这里更新了自己知识上的盲点，在iOS中类的本质上也是个对象，描述对象的对象。所以动态添加成员变量也就成了可能。以下摘自[Stackoverflow](http://stackoverflow.com/questions/15609149/is-it-correct-to-use-objc-setassociatedobject-for-class-object):

> Yes, a class object is a full-fledged object, so you can do anything to it that you can do with a regular object.
> However, it is clearer and simpler to use a global variable.
> p.s. Associating it with [self class] is not the same as using a global variable, because [self class] gives you the actual class of the current object, which may vary as this method is inherited by subclasses. Whereas with a global variable it would always be the same variable.

#### Transformer

NSValueTransformer在Mantle中对象转换中起了相当重要的作用。下面的代码是官方示例中的使用方法，是由“Property Key”+“JSONTransformer”组成的。

```objc
+ (NSValueTransformer *)updatedAtJSONTransformer {
    return [MTLValueTransformer transformerUsingForwardBlock:^id(NSString *dateString, BOOL *success, NSError *__autoreleasing *error) {
        return [self.dateFormatter dateFromString:dateString];
    } reverseBlock:^id(NSDate *date, BOOL *success, NSError *__autoreleasing *error) {
        return [self.dateFormatter stringFromDate:date];
    }];
}
```

`MTLJSONAdapter`在进行Modle和JSON之间转换的过程中，会通过runtime来构造所有可能的JSONTransformer，构造代码非常典型，如果不按上述的命名方式是无法匹配到的。此外由于Class对象没有`performSelector`方法，所以采用非常trick方式来调用JSONTransformer类方法。因为之前意识到类也是一个对象，那么类方法就是类对象的方法，运行时的那一套依旧适用，动态添加类变量或者类方法，想想还是挺有趣的。

```objc
for (NSString *key in [modelClass propertyKeys]) {
        SEL selector = MTLSelectorWithKeyPattern(key, "JSONTransformer");
        if ([modelClass respondsToSelector:selector]) {
                IMP imp = [modelClass methodForSelector:selector];
                NSValueTransformer * (*function)(id, SEL) = (__typeof__(function))imp;
                NSValueTransformer *transformer = function(modelClass, selector);
                if (transformer != nil) result[key] = transformer;
                continue;
        }
}
```

最后一个问题一个MTLModel<MTLJSONSerializing>的子类中的成员变量中有同样是MTLModel<MTLJSONSerializing>类型的，那该怎么办？当然最直接的想法是再手动写一个JSONTransformer。然而这次你不用费心，Mantle如果发现一个对象没有定义JSONTransformer，会去判断这个对象的类型，若符合MTLJSONSerializing协议则会创建一个JSONTransformer。该Transformer的实现仍然是用到MTLJSONAdapter的相互转换的特性。这里有一种MTLJSONAdapter递归调用的思想。以下是实现可以参考函数dictionaryTransformerWithModelClass。

这篇文章就写到这里，Mantle的源码一大特点就是判断逻辑非常多，这些都是为了确保转换的稳定性和尽可能多的可转换性，不亲自参与或者认真对待代码是无法想到那么的情况。在以后的编程过程中我也应当更充分地考虑各种边界条件，提高代码的稳定性。

