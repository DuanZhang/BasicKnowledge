### 任务列表
- [ ] `KVC`、`KVO`基础知识
- [ ] `KVC`、`KVO`使用场景
- [ ] `KVC`、`KVO`的底层实现原理
- [ ] `KVC`、`KVO`解决了什么问题
- [ ] `KVC`、`KVO`的替代方案
- [ ] `KVO`如何触发
- [ ] OC、Swift中的区别
 
### KVC
#### `KVC`基础知识 
KVC(Key-Value Coding)，定义在NSKeyValueCoding.h文件中，是一个非正式协议。KVC提供了一种间接访问其属性方法或成员变量的机制，可以通过key来访问对应的属性方法或成员变量,而不需要调用明确的存取方法。可以在运行时动态的访问和修改对象的属性。

#### KVC的基本使用
- 通过key来取值
1. valueForKey:
2. valueForKeyPath:
3. dictionaryWithValuesForKeys:
4. valueForUndefinedKey:
5. mutableArrayValueForKey:
6. mutableArrayValueForKeyPath:
7. mutableSetValueForKey:
8. mutableSetValueForKeyPath:
9. mutableOrderedSetValueForKey:
10. mutableOrderedSetValueForKeyPath:

- 通过key来设值
1. setValue:forKeyPath:
2. setValuesForKeysWithDictionary:
3. setNilValueForKey:
4. setValue:forKey:
5. setValue:forUndefinedKey:

- 修改默认行为
1. accessInstanceVariablesDirectly

- 验证
1. validateValue:forKey:error:
2. validateValue:forKeyPath:error:

#### `KVC`使用场景
- 动态取值和设值
- 访问、修改私有成员变量的值（比如替换系统自带的tabBar）
> 替换系统自带的tabBar,系统的是 readonly
[self setValue:[[MyTabBar alloc] init] forKeyPath:@"tabBar"];
- 字典转模型
- 操作集合
- `KVC`中使用KeyPath(暂缺)

#### KVC的底层实现（配套有测试的demo）
- 当一个对象调用`setValue:forKey:`方法时，方法内部会做以下操作
1. 检查是否存在相应key的setter方法，如果存在，就调用
2. 如果set方法不存在,并且`accessInstanceVariablesDirectly`为默认值，就查找与key相同名称并且带下划线的成员属性，如果有直接给成员属性赋值
3. 如果还没有找到_key，则查找相同名称的属性key，如果有就直接赋值(按照`_key、_isKey、key、isKey`的顺序搜索成员名)
4. 如果还没有找到则调用`setValue:forUndefinedKey:`方法,默认是抛出异常
- 当一个对象调用`valueForKey`方法时，方法内部做如下操作（暂缺）





#### KVC内部实现机制（暂缺）





#### KVC异常处理
1. 模型里的属性要和JSON 解析的值一致
>  -(id)valueForUndefinedKey:(NSString *)key{
    return nil;
}

2. 模型中的属性与json解析中的不一致的字段（例如id等和系统关键字冲突的属性）

> -(void)setValue:(id)value forUndefinedKey:(NSString *)key{ 
if ([key isEqualToString:@"id"]) {
    [self setValue:value forKey:@"ID"]; 
  }
}

3. 使用了错误的key，或者传递了nil
> -(void)setNilValueForKey:(NSString *)key{
   //提示
}



### KVO
#### KVO的基本知识
KVO(Key-Value Observing),利用一个key来找到某个属性并监听其值的改变。其实这也是一种典型的观察者模式。

#### KVO的基本用法
1. 添加观察者
2. 在观察者中实现监听方法
> observeValueForKeyPath: ofObject: change: context:
3. 移除观察者

#### KVO的底层实现
![KVO实现原理](https://github.com/DuanZhang/BasicKnowledge/blob/master/Notes/KVC-KVO/1429890-b28e010d3a7dbdb8.png)
1. KVO是基于runtime机制实现的
2. 当某个类的属性对象第一次被观察时，系统就会在运行期动态地创建该类的一个派生类，在这个派生类中重写基类中任何被观察属性的setter 方法。派生类在被重写的setter方法内实现真正的通知机制
3. 如果原类为Person，那么生成的派生类名为NSKVONotifying_Person
4. 每个类对象中都有一个isa指针指向当前类，当一个类对象的第一次被观察，那么系统会偷偷将isa指针指向动态生成的派生类，从而在给被监控属性赋值时执行的是派生类的setter方法
键值观察通知依赖于NSObject 的两个方法: willChangeValueForKey: 和 didChangevlueForKey:；在一个被观察属性发生改变之前， willChangeValueForKey:一定会被调用，这就 会记录旧的值。而当改变发生后，didChangeValueForKey:会被调用，继而 observeValueForKey:ofObject:change:context: 也会被调用。

补充：KVO的这套实现机制中苹果还偷偷重写了class方法，让我们误认为还是使用的当前类，从而达到隐藏生成的派生类




#### KVO的优缺点
##### 优点
1. 能够提供一种简单的方法实现两个对象间的同步，例如model和view
2. 能够
3. 可以得到观察属性的最新值以及先前值
4. 使用keypaths进行观察，可以得到嵌套的对象
5. 完成了对观察对象的抽象，不需要额外的代码来允许观察值能够被观察

#### 相关链接
[KVC&KVO](https://www.jianshu.com/p/f1393d10109d)

[KVO的底层实现原理](https://www.jianshu.com/p/829864680648)

[详解KVC](https://www.jianshu.com/p/45cbd324ea65)

[KVC/KVO原理详解及编程指南](https://blog.csdn.net/iunion/article/details/46890809)

[isa-swizzling](http://www.pluto-y.com/isa-swizzling-and-runtime/)



