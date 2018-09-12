### Python进阶-函数默认参数

#### 写在前面

> 
> 
> 如非特别说明，下文均基于`Python3`
> 
> 

#### 一、默认参数

python为了简化函数的调用，提供了默认参数机制：

```
def pow(x, n = 2):

    r = 1
    while n > 0:
        r *= x
        n -= 1
    return r
```

这样在调用`pow`函数时，就可以省略最后一个参数不写：

```
print(pow(5)) # output: 25
```

在定义有默认参数的函数时，需要注意以下：

1.  必选参数必须在前面，默认参数在后；
2.  设置何种参数为默认参数？一般来说，将参数值变化小的设置为默认参数。

**python标准库实践**
python内建函数：
`print(*objects, sep=' ', end='\n', file=sys.stdout, flush=False)`

函数签名可以看出，使用`print('hello python')`这样的简单调用的打印语句，实际上传入了许多默认值，默认参数使得函数的调用变得非常简单。

#### 二、一个坑？

引用一个官方的经典示例[地址](http://python.net/~goodger/projects/pycon/2007/idiomatic/handout.html#default-parameter-values) ：

```
def bad_append(new_item, a_list=[]):
    a_list.append(new_item)
    return a_list

print(bad_append('1'))
print(bad_append('2'))
```

这个示例并没有按照预期打印：

```
['1']
['2']
```

而是打印了：

```
['1']
['1', '2']
```

其实这个错误问题不在默认参数上，而是我们对于及默认参数的初始化的理解有误。

#### 三、函数初始化

按照`Python`哲学：

> 
> 
> 一切皆对象
> 
> 

函数也是一个对象，如下示例：

```
import types

def test():
    pass

print(type(test)) # <class 'function'>
print(isinstance(test, types.FunctionType)) # True
```

如此，函数就是类`types.FunctionType`或者其子类的实例对象。那么对象必然有其初始化的时候，一般来说，解释器在读到函数末尾时完成函数实例的初始化。初始化后，就有了函数名到函数对象这样一个映射关系，可以通过函数名访问到函数对象了，并且，函数的一切属性也确定下来，包括所需的参数，**默认参数的值**。因此每次调用函数时，默认参数值是相同的（如果有默认参数）。

* * *

我们以一个直观的例子来说明：

```
import datetime as dt
from time import sleep

def log_time(msg, time=dt.datetime.now()):

    sleep(1) # 线程暂停一秒
    print("%s: %s" % (time.isoformat(), msg))

log_time('msg 1')
log_time('msg 2')
log_time('msg 3')
```

运行这个程序，得到的输出是：

```
2017-05-17T12:23:46.327258: msg 1
2017-05-17T12:23:46.327258: msg 2
2017-05-17T12:23:46.327258: msg 3
```

即使使用了`sleep(1)`让线程暂停一秒，排除了程序执行很快的因素。输出中三次调用打印出的时间还是相同的，即三次调用中默认参数`time`的值是相同的。

上面的示例或许还不能完全说明问题，以下通过观察默认参数的内存地址的方式来说明。

首先需要了解内建函数`id(object)` :

> 
> 
> **id(object)**
> Return the “identity” of an object. This is an integer which is guaranteed to be unique and constant for this object during its lifetime. Two objects with non-overlapping lifetimes may have the same id() value.
> 
> 

> 
> 
> **CPython implementation detail**: This is the address of the object in memory.
> 
> 

即`id(object)`函数返回一个对象的唯一标识。这个标识是一个在对象的生命周期期间保证唯一并且不变的整数。在重叠的生命周期中，两个对象可能有相同的id值。
在`CPython`解释器实现中，`id(object)`的值为对象的内存地址。

如下示例使用`id(object)`函数清楚说明了问题：

```
def bad_append(new_item, a_list=[]):

    print('address of a_list:', id(a_list))
    a_list.append(new_item)
    return a_list

print(bad_append('1'))
print(bad_append('2'))
```

output:

```
address of a_list: 31128072
['1']
address of a_list: 31128072
['1', '2']
```

两次调用`bad_append`，默认参数`a_list`的地址是相同的。
而且`a_list`是可变对象，使用`append`方法添加新元素并不会造成`list`对象的重新创建，地址的重新分配。这样，‘恰好’就在默认参数指向的地址处修改了对象，下一次调用再次使用这个地址时，就可以看到上一次的修改了。

那么，出现上述的输出就不奇怪了，因为它们本来就是指向同一内存地址。

#### 四、可变与不可变

当默认参数指向可变类型对象和不可变类型对象时，会表现出不同的行为。

**可变默认参数** 的表现就像上诉示例一样。

**不可变默认参数**
首先看一个示例：

```
def immutable_test(i = 1):
    print('before operation, address of i', id(i))
    i += 1
    print('after operation, address of i', id(i))
    return i

print(immutable_test())
print(immutable_test())
```

Output:

```
before operation, address of i 1470514832
after operation, address of i 1470514848
2
before operation, address of i 1470514832
after operation, address of i 1470514848
2
```

很明显，第二次调用时默认参数`i`的值不会受第一次调用的影响。因为`i`指向的是不可变对象，对`i`的操作会造成内存重新分配，对象重新创建，那么函数中`i += 1`之后名字`i`指向了另外的地址；根据默认参数的规则，下次调用时，`i`指向的地址还是函数定义时赋予的地址，这个地址的值`1`并没有被改变。

其实，可变默认参数和不可变默认参数放在这里讨论并没太大的价值，就像其他语言中所谓的`值传递还是引用传递`一样，不只会对默认参数造成影响。

#### 五、最佳实践

不可变的默认参数的多次调用不会造成任何影响，可变默认参数的多次调用的结果不符合预期。那么在使用可变默认参数时，就不能只在函数定义时初始化一次，而应该在每次调用时初始化。

**最佳实践**是定义函数时指定可变默认参数的值为`None`，在函数体内部重新绑定默认参数的值。以下是对上面的两个可变默认参数示例最佳实践的应用：

```
def good_append(new_item, a_list = None):

    if a_list is None:
        a_list = []

    a_list.append(new_item)
    return a_list

print(good_append('1'))
print(good_append('2'))
print(good_append('c', ['a', 'b']))
```

```
import datetime as dt
from time import sleep

def log_time(msg, time = None):

    if time is None:
        time = dt.datetime.now()

    sleep(1)
    print("%s: %s" % (time.isoformat(), msg))

log_time('msg 1')
log_time('msg 2')
log_time('msg 3')
```