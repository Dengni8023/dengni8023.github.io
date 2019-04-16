---
layout:     post
title:      iOS开发之KVC、KVB、KVO
date:       2018-03-01
author:     dengni8023
catalog:    true
tags:
    - iOS
    - KVC
    - KVB
    - KVO
---

# 相关概念

Objective-C每个类都有一张方法表，是一个hash表，值是函数指针IMP，SEL的名称就是查表时所用的键。
>  <strong>isa指针</strong>：当一个对象被创建时，内存布局中的第一个元素是指向类结构的指针，即isa。通过isa指针，一个对象可以访问它的类结构，进而访问继承的类结构。
> 
> <strong>SEL数据类型</strong>：它是编译器运行Objective-C里的方法的环境参数，查找方法表时所用的键。定义成char*，实质上可以理解成int值。
> 
> <strong>IMP数据类型</strong>：他其实就是一个编译器内部实现时候的函数指针。当Objective-C编译器去处理实现一个方法的时候，就会指向一个IMP对象，这个对象是C语言表述的类型（在Objective-C的编译器处理的时候，基本上都是C语言的）。

# KVC

> Key Value Coding，即键值编码，通常是用来给某一个对象的属性进行赋值，是一种可以通过字符串的名字（key）来访问对象属性的机制，而不是通过调用Setter、Getter方法访问。

KVC运用了isa-swizzing技术。isa-swizzling就是类型混合指针机制。KVC通过isa-swizzling，来实现其内部查找定位。isa（is a kind of）指针指向维护分发表的对象的类。该分发表实际上包含了指向实现类中的方法的指针和其它数据。

如下一行KVC代码：

```
[person setValue:@"姓名" forKey:@"name"];
```

会被编译器处理为：

```
SEL sel = sel_getUid("setValue:forKey:");
IMP method = objc_msg_loopup(person->isa, sel);
method(person, sel, @"姓名", @"name");
```

如上，使用KVC在调用setValue:forKey:对象时，大概分成三个步骤：
>
1. 获取SEL对象
1. 通过sel对象和当前的isa指针，获取函数指针(IMP对象)
1. 通过C函数指针的调用方式，直接调用函数(此处省去了发送消息的开销)

KVC关键方法定义在<Foundation/NSKeyValueCoding.h>文件中，KVC支持类对象和BOOL、int等内建基本数据类型。

## KVC使用

1. 读取属性值

	```
	- (nullable id)valueForKey:(NSString *)key; // 传入NSString属性的名字
	- (nullable id)valueForKeyPath:(NSString *)keyPath; // 属性的路径，属性多层嵌套，xx.key
	- (nullable id)valueForUndefinedKey:(NSString *)key; // 默认实现是抛出异常，可重写这个函数做错误处理
	
	// 获取可变类型数据方法
	- (NSMutableArray *)mutableArrayValueForKey:(NSString *)key;
	- (NSMutableOrderedSet *)mutableOrderedSetValueForKey:(NSString *)key;
	- (NSMutableSet *)mutableSetValueForKey:(NSString *)key;
	```
	
	> 注意：
	> 
	> 获取可变数据后，对获取的可变数据进行操作，会影响原对象的属性值
	
示例代码：[- (void)personKVCMutableSetFunction {...}](https://github.com/dengni8023/Dengni8023BlogDemo/blob/master/JGBlogExample/JGBlogExample/Classes/KVC_KVO/KVC_KVOViewController.m)

1. 修改（设置）属性值

	```
	- (void)setValue:(nullable id)value forKey:(NSString *)key; // 如value为nil则调用setNilValueForKey:方法
	- (void)setValue:(nullable id)value forKeyPath:(NSString *)keyPath; // 属性的路径，属性多层嵌套，xx.key
	- (void)setValue:(nullable id)value forUndefinedKey:(NSString *)key; // 默认实现是抛出异常，可重写这个函数做错误处理
	- (void)setNilValueForKey:(NSString *)key; // 默认实现是抛出异常，可重写这个函数做错误处理
	```

## KVC键值查找key

### 修改（设置）属性值搜索方式

调用方法 `- (void)setValue:(id)value forKey:(NSString *)key`，搜索步骤如下：

1. 搜索`- (void)set<#Key#>:(NSString *)<#key#>`方法（key指成员变量名，首字母大写）。

1. 如果没有找到`- (void)set<#Key#>:(NSString *)<#key#>`方法，而类方法`+ (BOOL)accessInstanceVariablesDirectly`返回`YES`，则按`_<#key#>`，`_is<#Key#>`，`<#key#>`，`is<#key#>`的顺序搜索成员名。
	
	> 1. `+ (BOOL)accessInstanceVariablesDirectly`默认实现为返回`YES`。
	> 1. 若`+ (BOOL)accessInstanceVariablesDirectly`返回`NO`，则调用`- (void)setValue:(id)value forUndefinedKey:(NSString *)key`方法。

1. 如果没有找到成员变量，则调用`- (void)setValue:(id)value forUndefinedKey:(NSString *)key`方法。

示例代码：[- (void)personKVCSetSearchFunction {...}](https://github.com/dengni8023/Dengni8023BlogDemo/blob/master/JGBlogExample/JGBlogExample/Classes/KVC_KVO/KVC_KVOViewController.m)
 
### 读取属性值搜索方式

调用方法 `- (id)valueForKey:(NSString *)key`，搜索步骤如下：

1. 按`get<#Key#>`，`<#key#>`，`is<#Key#>`，`_<#key#>`的顺序查找getter方法，找到则直接调用。对于BOOL、int等内建值类型，会做NSNumber的转换。

1. getter方法没有找到，查找`- (NSInteger)countOf<#Key#>`，`- (id)objectIn<#Key#>AtIndex:(NSUInteger)index`，`- (NSArray *)<#key#>AtIndexes:(NSIndexSet *)indexes`格式的方法。如果`- (NSInteger)countOf<#Key#>` <strong>和</strong> 另外两个方法中的一个找到，那么就会返回一个可以响应NSArray所有方法的代理集合（它是NSKeyValueArray，是NSArray的子类）。

	> 1. 这一步为针对既未声明property，也未定义成员变量的key，返回 <strong>有序集合</strong>
	> 1. 两个方法同时有实现时 `- (id)objectIn<#Key#>AtIndex:(NSUInteger)index` <strong>优先级高于</strong> `- (NSArray *)<#key#>AtIndexes:(NSIndexSet *)indexes`
	> 1. 若重写了`- (void)get<#Key#>:(NSString * __unsafe_unretained *)buffer range:(NSRange)inRange`以提高性能
	>
	>> 1. `- (id)objectIn<#Key#>AtIndex:(NSUInteger)index` 和 `- (NSArray *)<#key#>AtIndexes:(NSIndexSet *)indexes`的内部实现无效
	>> 1. `- (id)objectIn<#Key#>AtIndex:(NSUInteger)index` 和 `- (NSArray *)<#key#>AtIndexes:(NSIndexSet *)indexes`其中之一必须实现

1. 如果上面的方法没有找到，查找`- (NSInteger)countOf<#Key#>`，`- (NSEnumerator *)enumeratorOf<#Key#>`，`- (id)memberOf<#Key#>:(id)object`格式的方法。如果这三个方法都找到，那么就返回一个可以响应NSSet所有方法的代理集合（它是NSKeyValueSet，是NSSet的子类）。

	> 1. 这一步为针对既未声明property，也未定义成员变量的key，返回 <strong>无序集合</strong>
	> 1. `- (id)memberOf<#Key#>:(id)object`方法的返回类型和`object`类型均为集合内部元素的类型
 
1. 如果还没有找到，再检查类方法`+ (BOOL)accessInstanceVariablesDirectly`返回`YES`，则按`_<#key#>`，`_is<#Key#>`，`<#key#>`，`is<#key#>`的顺序搜索成员名。

	> 1. `+ (BOOL)accessInstanceVariablesDirectly`默认实现为返回`YES`。
	> 1. 若`+ (BOOL)accessInstanceVariablesDirectly`返回`NO`，则调用`- (id)valueForUndefinedKey:(NSString *)key`方法。

1. 还没有找到的话，调用`- (id)valueForUndefinedKey:(NSString *)key`方法。

示例代码：[- (void)personKVCGetSearchFunction {...}](https://github.com/dengni8023/Dengni8023BlogDemo/blob/master/JGBlogExample/JGBlogExample/Classes/KVC_KVO/KVC_KVOViewController.m)

## KVC键值验证KVV

KVV（Key Value Validation），KVC提供用来验证可以Key对应Value是否可用的方法:

`- (BOOL)validateValue:(inout id  _Nullable __autoreleasing *)ioValue forKey:(NSString *)inKey error:(out NSError * _Nullable __autoreleasing *)outError;`

方法的默认实现是查找对应Key是否有实现以下方法

`- (BOOL)validate<#Key#>:(inout id  _Nullable __autoreleasing *)ioValue error:(out NSError * _Nullable __autoreleasing *)outError`

如果有实现，则调用该方法，否则直接返回YES

> KVC在设值时不会主动去做验证，需要开发者手动去验证。所以即使你在类里面写了验证方法，但是KVC因为不会去主动验证，所以还是能够设值成功。
> 
> 可重写 `- (void)setValue:(id)value forKey:(NSString *)key`方法，在内部调用验证方法进行逻辑验证。

`- (BOOL)validate<#Key#>:(inout id  _Nullable __autoreleasing *)ioValue error:(out NSError * _Nullable __autoreleasing *)outError`在多个Key都需要定义验证逻辑时需要针对不同Key实现多个Key的验证逻辑接口，可以将该逻辑放到以下接口中进行，除了需要自定义验证逻辑的Key，其他的使用`[super validateValue:ioValue forKey:inKey error:outError]`即可处理

示例代码:[- (BOOL)validateValue:(inout id  _Nullable __autoreleasing *)ioValue forKey:(NSString *)inKey error:(out NSError * _Nullable __autoreleasing *)outError {...}](https://github.com/dengni8023/dengni8023.github.io/blob/master/_SourceCode/JGBlogExample/JGBlogExample/Classes/KVC_KVO/KVCPerson.m)

## KVC中的集合操作函数

KVC提供很复杂的函数用于操作对象集合，主要有下面这些：

1. 简单运算符5种，如下：

	>
		1. @avg 取平均值，`[<#Array#> valueForKeyPath:@"@avg.<#key#>"]`
		2. @count 对象个数，`[<#Array#> valueForKeyPath:@"@count"]`
		3. @max 最大值，`[<#Array#> valueForKeyPath:@"@max.<#key#>"]`
		4. @min 最小值，`[<#Array#> valueForKeyPath:@"@min.<#key#>"]`
		5. @sum 求和，`[<#Array#> valueForKeyPath:@"@sum.<#key#>"]`

示例代码：[- (void)personKVCFuncCalculateFuntion {...}](https://github.com/dengni8023/Dengni8023BlogDemo/blob/master/JGBlogExample/JGBlogExample/Classes/KVC_KVO/KVC_KVOViewController.m)

2. 对象运算符2种，如下：

	>
		1. @distinctUnionOfObjects 元素去重后的Array，`[<#Array#> valueForKeyPath:@"@distinctUnionOfObjects.<#key#>"]`
		2. @unionOfObjects 元素的全集Array，`[<#Array#> valueForKeyPath:@"@unionOfObjects.<#key#>"]`
		
示例代码：[- (void)personKVCFuncObjectFuntion {...}](https://github.com/dengni8023/Dengni8023BlogDemo/blob/master/JGBlogExample/JGBlogExample/Classes/KVC_KVO/KVC_KVOViewController.m)

3. 针对集合属性的Array和Set运算符，如下：

	>
		1. @distinctUnionOfArrays 去重后的Array，`[<#Array#> valueForKeyPath:@"@distinctUnionOfArrays.<#key#>"]`，Key对应属性为Array或者Set
		2. @unionOfArrays 元素的全集Array，`[<#Array#> valueForKeyPath:@"@ unionOfArrays.<#key#>"]`，Key对应属性为Array或者Set
		3. @distinctUnionOfSets，Set不支持重复元素，因此和@distinctUnionOfArrays类似，Key对应属性必须为Set

示例代码：[- (void)personKVCFuncCollectFuntion {...}](https://github.com/dengni8023/Dengni8023BlogDemo/blob/master/JGBlogExample/JGBlogExample/Classes/KVC_KVO/KVC_KVOViewController.m)

# KVB

> Key Value Binding，用来绑定对象与观察者，即为对象的观察者与被观察者之间建立联系。

KVB为对象的观察者与被观察者之间建立联系的方法包括：

1. 为对象添加观察者Observer

	```
	- (void)addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(nullable void *)context;
	```
	
	> 相关参数：
	>
	> observer: 观察者，KVO通知的订阅者（通知消息接收者），订阅者必须实现方法
	>
	> ```
	> - (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
	> 
	> }
	> ```
	>
	> keyPath: 被观察对象属性的路径keyPath，支持属性多层嵌套，单层时 key，多层嵌套时 xx.key
	>
	> options: KVO的属性配置，有四个选项
	>
	> 1. NSKeyValueObservingOptionInitial: 注册后立刻触发KVO通知
	> 1. NSKeyValueObservingOptionPrior: 值改变前是否通知（这个选项决定了是否在改变前后分别通知）
	> 1. NSKeyValueObservingOptionOld: change字典包括改变前的值
	> 1. NSKeyValueObservingOptionNew: change字典包括改变后的值
	>
	> context: 上下文，将传递到订阅者的收到的消息中。通常用来区分不同的KVO，所以context的唯一性很重要，通常在当前.m文件里用静态变量定义`static void * <#KVOIdentifier#>Context = 0;`或者`static int const <#KVOIdentifier#>Context;`，使用`static int const <#constKVOIdentifier#>Context;`时注意context参数使用`(void *)&<#constKVOIdentifier#>Context`做强制转换。
	
	> 注意：
	> 为对象添加观察者后，在对象释放前要移除已添加的观察者，否则会导致内存泄漏。ARC模式下，不移除观察者会导致崩溃问题；MRC模式，不移除观察者，自动释放内存的被观察对象不会释放内存，手动释放被观察者内存时会崩溃。
	>
	> 1. 局部使用的对象在使用完成时移除观察者
	> 1. 全局使用的对象在添加观察者的对象的`- (void)dealloc`方法中移除观察者
	> 1. context一般不传参数，即使用nil，但是为了避免[KVB陷阱](#TrapOfKVB) ，建议每次添加、移除观察者时指定context。
	
1. 观察者Observer收到信息的处理

	```
	- (void)observeValueForKeyPath:(nullable NSString *)keyPath ofObject:(nullable id)object change:(nullable NSDictionary<NSKeyValueChangeKey, id> *)change context:(nullable void *)context;
	```

	> 相关参数：
	>
	> keyPath: 被观察对象属性的路径keyPath，支持属性多层嵌套，单层时 key，多层嵌套时 xx.key
	>
	> object: 被观察对象，可通过该对象获取修改后的值
	>
	> change: 值改变的信息，根据添加观察者时的options选项，可能有旧值、新值以及其他信息
	>
	> context: 上下文，添加观察者时的context回传，用来判断是否时相同的KVO监听，判断方法 `context == <#KVOIdentifier#>Context` 或者 `context == (void *)&<#constKVOIdentifier#>Context`
	
1. 为对象移除观察者Observer

	```
	- (void)removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath context:(nullable void *)context;
	- (void)removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath;
	```
	
	> 注意：
	>
	> 1. 针对相同keyPath、context，移除观察者时按照先添加册先移除规则进行移除。
	> 2. 针对相同keyPath，移除观察者时若不指定context，将优先移除添加观察者时未指定context的观察者，所有未指定context的观察者已全部移除，则按照先添加册先移除规则移除指定了context添的观察者。
	> 3. 为避免添加、移除时context导致添加移除不成对问题，建议添加、移除观察者时均制定context。
	
## <span id="TrapOfKVB">KVB陷阱

1. 移除观察者陷阱

	> KVO针对同一keyPath观察者移除次数超过添加次数时，将会导致崩溃。常见于不指定context，父类与子类均在dealloc时做移除观察者处理，就会出现该问题。解决方法为添加观察者时指定context，在父类和子类中使用不同的context，dealloc移除观察者时指定context为添加时使用的context即可解决。

1. 接受消息通知处理陷阱

	```
	- (void)observeValueForKeyPath:(nullable NSString *)keyPath ofObject:(nullable id)object change:(nullable NSDictionary<NSKeyValueChangeKey, id> *)change context:(nullable void *)context {
		
		if (context == <#KVOIdentifier#Context> && [object isEqual:<#object#>] && [keyPath isEqualToString:<#keyPath#>]) {
	        
	        // 本类处理
	    }
	    else {
	        [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
	    }
	}
	```
	
	> 若不执行else逻辑，KVO响应逻辑在执行完本类的逻辑处理后，KVO触发的响应将导致终止。但是不满足if条件的逻辑，很可能需要在父类、父类的父类...中处理，因此需要添加else逻辑调用触发父类的处理逻辑。

# KVO

> Key Value Observing，它提供一种机制，当指定的对象的属性被修改后，则对象就会接受到通知。简单的说就是每次指定的被观察的对象的属性被修改后，KVO就会自动通知相应的观察者了。
> 
> KVO是“观察者”设计模式的一种实现（另一种实现：NSNotification），利用它可以很容易实现视图组件和数据模型的分离，当数据模型的属性值改变之后作为监听器的视图组件就会被激发，激发时就会回调监听器自身。
> 
> 这种模式有利于两个类间的解耦合，尤其是对于业务逻辑与视图控制 这两个功能的解耦合。在MVC模式下，KVO机制很适合实现Model模型和View视图之间的通信。
>
> KVO的触发必需通过Setter方法，`_key = <#value#>`将不会触发KVO。

## KVO实现原理

KVO同KVC一样，通过 isa-swizzling 技术来实现。当观察者被注册为一个对象的属性的观察对象的isa指针被修改，指向一个中间类，而不是在真实的类。其结果是，isa指针的值并不一定反映实例的实际类。所以不能依靠isa指针来确定对象是否是一个类的成员。应该使用class方法来确定对象实例的类。

Apple 使用了 isa 混写（isa-swizzling）来实现 KVO 。当观察对象A时，KVO机制动态创建一个新的名为：NSKVONotifying_A 的新类，该类继承自对象A的本类，且 KVO 为 NSKVONotifying_A 重写观察属性的 setter 方法，setter 方法会负责在调用原 setter 方法之前和之后，通知所有观察对象属性值的更改情况。

![](http://github-blog.dengni8023.com/iOS开发之KVC、KVB、KVO-1.png)

1. NSKVONotifying_A 类剖析
	> 在这个过程，被观察对象的 isa 指针从指向原来的 A 类，被 KVO 机制修改为指向系统新创建的子类 NSKVONotifying_A 类，来实现当前类属性值改变的监听。
	>
	> 所以当我们从应用层面上看来，完全没有意识到有新的类出现，这是系统“隐瞒”了对 KVO 的底层实现过程，让我们误以为还是原来的类。但是此时如果我们创建一个新的名为 NSKVONotifying_A 的类()，就会发现系统运行到注册 KVO 的那段代码时程序就崩溃，因为系统在注册监听的时候动态创建了名为 NSKVONotifying_A 的中间类，并指向这个中间类了。
	>
	>（isa 指针的作用：每个对象都有 isa 指针，指向该对象的类，它告诉 Runtime 系统这个对象的类是什么。所以对象注册为观察者时，isa 指针指向新子类，那么这个被观察的对象就神奇地变成新子类的对象（或实例）了。） 因而在该对象上对 setter 的调用就会调用已重写的 setter，从而激活键值通知机制。

1. 子类setter方法剖析
	> KVO 的键值观察通知依赖于 NSObject 的两个方法`willChangeValueForKey:`和`didChangevlueForKey:`，在存取数值的前后分别调用 2 个方法
	
	> 1. 被观察属性发生改变之前，`willChangeValueForKey:`被调用，通知系统该 keyPath 的属性值即将变更
	> 1. 当改变发生后，`didChangeValueForKey:`被调用，通知系统该 keyPath 的属性值已经变更
	
	> 之后， observeValueForKey:ofObject:change:context: 也会被调用。且重写观察属性的 setter 方法这种继承方式的注入是在运行时而不是编译时实现的。
	
KVO 为子类的观察者属性重写调用存取方法的工作原理在代码中相当于：

```
- (void)setName:(NSString *)name {
    [self willChangeValueForKey:@"name"]; //KVO 在调用存取方法之前调用 
    [super setName:name]; //调用父类的存取方法 
    [self didChangeValueForKey:@"name"]; //KVO 在调用存取方法之后调用
}
```

## KVO线程

KVO消息处理是在被观察值发生变化的线程上进行的，KVO消息处理是同步进行的，处理过程中将阻塞线程，当所有观察者处理完消息后才回释放线程资源。如过一个对象有多个观察者，各观察者收到订阅消息是无序的。

在多线程修改值会KVO时，应确定所有的观察者都用线程安全的方法处理KVO通知。通常请不要在多线程上使用KVO。

## 手动KVO

通常定义类时，系统自动帮我们生成了KVO相关处理逻辑，如果想要灵活添加自定义处理逻辑，则可以使用手动KVO。手动KVO需要实现`- (void)set<#Key#>:(id)<#key#>`方法以及`+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key`或者`+ (BOOL)automaticallyNotifiesObserversOf<#Key#>`

> `+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key`优先级 <strong>高于</strong>`+ (BOOL)automaticallyNotifiesObserversOf<#Key#>`
> 
> `+ (BOOL)automaticallyNotifiesObserversOfName`默认实现直接返回YES，使用系统默认KVO逻辑，返回NO时才执行手动KVO逻辑，如果返回YES的同时`- (void)set<#Key#>:(id)<#key#>`方法内部有手动KVO逻辑，手动、自动KVO都将触发，即观察者将收到重复消息。
> 
> 手动KVO时`- (void)set<#Key#>:(id)<#key#>`方法内部值修改前必须执行`[self willChangeValueForKey:<#key#>];`，否则完全不会触发手动KVO；而不执行`[self didChangeValueForKey:<#key#>];`则仅是值修改后不会发送KVO消息。

## KVO键值观察依赖键

有时候对象的一个属性的值依赖于对象中的一个或多个属性，如果这些属性中任一属性的值发生变更，被依赖的属性值也应当为其变更进行标记，因此引入了依赖键。 

依赖键需要实现`- (void)set<#Key#>:(id)<#key#>`方法以及`+ (NSSet<NSString *> *)keyPathsForValuesAffectingValueForKey:(NSString *)key`或`+ (NSSet *)keyPathsForValuesAffecting<#Key#>`

> `+ (NSSet<NSString *> *)keyPathsForValuesAffectingValueForKey:(NSString *)key`或`+ (NSSet *)keyPathsForValuesAffecting<#Key#>`，返回key依赖的keyPath的Set集合。
> `+ (NSSet<NSString *> *)keyPathsForValuesAffectingValueForKey:(NSString *)key`优先级 <strong>高于</strong>`+ (NSSet *)keyPathsForValuesAffecting<#Key#>`。

### 测试代码

[测试代码 - 请使用KVC_KVOViewController类进行修改测试](https://github.com/dengni8023/Dengni8023BlogDemo)

# 参考资料

1. [iPhone KVO、KVC、KVB介绍](https://www.cnblogs.com/ydhliphonedev/archive/2013/04/22/3018664.html)
1. [深入理解 KVC\KVO 实现机制 — KVC](https://www.cnblogs.com/zy1987/p/4616063.html)
1. [Key-Value Coding](http://blog.csdn.net/qq_19762007/article/details/50654822)
1. [详解KVC，我来告诉你KVC的一切](http://ios.jobbole.com/84954/)
1. [iOS下KVO使用过程中的陷阱](https://www.cnblogs.com/wengzilin/p/4346775.html)
1. [深入理解 KVC\KVO 实现机制 — KVO](http://www.cnblogs.com/zy1987/p/4616764.html)
1. [KVC和KVO](https://www.jianshu.com/p/f1393d10109d)
1. [KVO 进阶](https://www.cnblogs.com/Jenaral/p/5431364.html)
1. [iOS开发 -- KVO的实现原理与具体应用](https://www.jianshu.com/p/e59bb8f59302)
1. [Objective-C之KVO(键值监听)](http://blog.csdn.net/lxl_815520/article/details/51153168)
