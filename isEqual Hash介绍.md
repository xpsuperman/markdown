#isEqual Hash介绍



在 iOS 中，判断两个对象是否相等，一般调用 `isEqual` 方法或者是 "变型" 方法（`isEqualToString` 等）。用 `==` 来判断两个对象是否相等，其实是判断两个对象的地址是否相等。

根据业务需求，自定义对象可能需要根据自身某个属性来判断是否相等（例如：根据对象 `id` 来判断两个对象是否相等），但是 `isEqual` 系统默认实现是比较两个对象的指针，这个时候我们就需要重写对象的 `isEqual` 方法来实现自身逻辑。

hash 方法的存在，是因为将对象加到 NSSet 等集合中时，需要利用对象的 Hash 值来标示对象在集合中的位置，将集合查找元素的时间复杂度优化成 O(1)。对于 Hash 值，系统默认是返回该对象的内存地址。



## 重写对象的 isEqual 方法

下面是重写对象 isEqual 方法代码：

```objective
- (BOOL)isEqual:(id)object {
    //1. == 判断地址
    if (self == object) return YES;

    //2.isKindOfClass 判断对象类型
    if (![object isKindOfClass:[self class]]) return NO;

    //3. 进行业务逻辑判断
    return [self isEqualToFather:(Father *)object];
}

- (BOOL)isEqualToFather:(Father *)object {
    //业务逻辑
    if ([self.name isEqualToString:object.name]) {
        return YES;
    }else {
        return NO;
    }
}
```

重写 `isEqual` 方法不是很难，只要根据自身的业务逻辑去实现就可以了。在这里，我们先判断对象地址是否相等，再判断对象类型，最后进行业务逻辑的判断，这样子可以更加高效、安全的去实现 `isEqual` 方法。



## 重写对象 hash 方法

我们知道 NSSet 不会添加重复元素，所以添加元素时候会判断对象是否与集合中的元素相等，流程如下：

1. 判断集合内的 hash 值是否和目标对象 hash 值一致，如果不一致则添加该对象，一致则进入第二步
2. 调用 `isEqual` 方法来判断对象是否一致，如果不一致则添加该对象，一致则不添加

这里我们可以知道：Hash 值是判断对象是否相等的充分非必要条件。

对于计算对象的 Hash 值，我们应该做到快速、重复率低、均匀等特性。[Mattt](https://nshipster.cn/authors/mattt/) 大神说：实际上，对于关键属性的散列值进行一个简单的 [`XOR`](https://en.wikipedia.org/wiki/Exclusive_or)操作，就能够满足在 99% 的情况下的需求了。具体可以看文末参考链接。



## 对象 isEqual 和 hash 方法需要同时重写

很多时候为了图方便，只会重写 `isEqual` 方法，忽略 `hash` 方法，这里我们看看下面这种情况：



### 重写 isEqual 方法，hash 方法没重写

这个时候会出现 `isEqual` 判断两个对象相同，但是 hash 值不同，但是这两个对象在 Set 集合中可以同时存在，这个在业务逻辑上是不合理的。



## 总结

只要弄清楚 isEqual 和 hash 两个方法存在的意义，且什么时候调用，就可以合理的重写这两个方法了。