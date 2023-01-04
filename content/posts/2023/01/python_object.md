---
title: "Python源码学习1 对象的秘密"
date: 2023-01-03T21:48:21+08:00
draft: false
author: "chimission"
categories: [""]
tags: ["python","源码"]
archives: ['2023', '2023-01']
comments: false
description: ""
url: "/posts/python_object/"
---
> 天天写python,新年开始咱也试着学习下python源码,python version 3.7
 <!--more-->
### 1.Everything in Python is an object
在 python 中，一切都是「对象」。但是这句话代表什么含义？ 英文原文「Everything in Python is an object」。「对象」对应英文中的「object」，在面向对象的语言中，「object」和「class」（类）是成对出现的，通常来讲「class」是数据的抽象模型，「object」是「class」的实体表现，但在 python 中对象的含义并非如此简单。我认为 一切都是「对象」有3层表达含义：

>1. 几乎所有的东西都是某个类的实例「instance」
>2. 几乎所有的东西都具有属性「attributes」
>3. 几乎所有的东西都可以被变量赋值从而被函数参数传递「variable」

这里讲几乎所有是因为要排除诸如「for」「def」等关键字。  

让我们来依次举例说明这3条含义，首先是第一条：「几乎所有的东西都是某个类的实例」  
```python
# type用于获取对象的类型 isinstance用于判断是不是某个类型的实例
from types import FunctionType

class Person:
    pass

def test():
    pass

print(type("Hello"), isinstance("Hello", str))
print(type(100), isinstance(100, int))
print(type(100.23), isinstance(100.23, float))
print(type(100 + 2j), isinstance(100 + 2j, complex))
print(type(True), isinstance(True, bool))
print(type(None), isinstance(None, type(None)))
print(type([]), isinstance([], list))
print(type(()), isinstance((), tuple))
print(type({}), isinstance({}, dict))
print(type({""}), isinstance({""}, set))
print(type(Person), isinstance(Person, type))
print(type(test), isinstance(test, FunctionType))

# 输出
#<class 'str'> True
#<class 'int'> True
#<class 'float'> True
#<class 'complex'> True
#<class 'bool'> True
#<class 'NoneType'> True
#<class 'list'> True
#<class 'tuple'> True
#<class 'dict'> True
#<class 'set'> True
#<class 'type'> True
#<class 'function'> True
```  
可以看到所有的变量或者类甚至是函数都有自己的类型，并且是某个类型的实例。需要注意的是「Person」类型，它本身是一个类，但是通过「type」函数得到了它的类型是「type」，而它自身也确实是「type」的实例，这个我们后面会继续讲到。  

然后是第二条「几乎所有的东西都具有属性」：
```python
# 内建函数dir用于获取对象参数的属性
def test():
    pass

print(dir(test))
# 输出
#['__annotations__',
# '__call__',
# '__class__',
# '__closure__',
# '__code__',
# '__defaults__',
# '__delattr__',
# '__dict__',
# '__dir__',
# '__doc__',
# '__eq__',
# '__format__',
# '__ge__',
# '__get__',
# '__getattribute__',
# '__globals__',
# '__gt__',
# '__hash__',
# '__init__',
# '__init_subclass__',
# '__kwdefaults__',
# '__le__',
# '__lt__',
# '__module__',
# '__name__',
# '__ne__',
# '__new__',
# '__qualname__',
# '__reduce__',
# '__reduce_ex__',
# '__repr__',
# '__setattr__',
# '__sizeof__',
# '__str__',
# '__subclasshook__']

print(test.__class__)
# 输出
# function
```  
这里我声明了一个空函数，即使是空函数也包含了很多内建属性和方法， 篇幅有限其他就不演示了，大家可以在自己电脑上试一试。  

最后是第三条「几乎所有的东西都可以被变量赋值从而被函数参数传递」：
```python
class Test:
    pass
def test():
    pass

def print_type(var):
    print(type(var))
a=int
print_type(a)
a=test
print_type(a)
a=1
print_type(a)
a=Test
print_type(Test)

#输出
#<class 'type'>
#<class 'function'>
#<class 'int'>
#<class 'type'>
```  
可以看到 无论是内建类型 int ，自定义类型 Test 还是函数test，都可以赋值给变量a，然后通过参数传递给函数执行。由此我们可以想到 python 在内部一定是统一了「对象」的表示，以此来实现「一切皆对象」，使用统一的API来管理python的对象。  

### PyObject，对象的统一表示  
在python内部，对象使用 `PyObject` 结构体来同意表示，在此结构体基础上添加额外的字段组成新的结构体来表示特定类型的对象，例如`PyFloatObject`代表float对象，`PyListObject`代表list对象。`PyObject` 定义在 `Include/object.h` 文件里。  
```c
/* Define pointers to support a doubly-linked list of all live heap objects. */
#define _PyObject_HEAD_EXTRA            \
    struct _object *_ob_next;           \
    struct _object *_ob_prev;

/* Nothing is actually declared to be a PyObject, but every pointer to
 * a Python object can be cast to a PyObject*.  This is inheritance built
 * by hand.  Similarly every pointer to a variable-size Python object can,
 * in addition, be cast to PyVarObject*.
 */
typedef struct _object {
    _PyObject_HEAD_EXTRA
    Py_ssize_t ob_refcnt;
    struct _typeobject *ob_type;
} PyObject;
```  

`_PyObject_HEAD_EXTRA` 是一个c语言宏，编译器会将其展开，实际上展开后`PyObject`有4个字段：ob_refcnt代表引用计数，ob_type代表类型指针，*_ob_next和*_ob_prev是一个双向链表用于跟踪所有 活跃堆对象，一般看源码用不到，这里我也没有深入学习。可以看到注释里说明了实际上在python里并没有任何对象是 `PyObject` 类型，但是任意一个对象都可以通过类型转换为 `PyObject*` 类型，从而实现统一。注释里还提到了可变长对象 `PyVarObject*`，其实 `PyObject`只能用来表示定长对象例如float，毕竟它没有字段表示 size 大小的信息。在面对list，dict等变长对象时就需要拓展，于是 `PyVarObject` 在 `PyObject` 的基础上增加了 `ob_size` 字段，用于表示当前对象的长度。   
>ps:有趣的是 int 对象在python中使用的 `PyVarObject`表示的，后面我们会继续讨论为什么，原因和python中的int可以表示非常大的数字有关系，一般其他语言都会限定int的表示范围例如int64，而python不会。

```c
typedef struct {
    PyObject ob_base;
    Py_ssize_t ob_size; /* Number of items in variable part */
} PyVarObject;
```
![对比](https://images.chimission.cn/blog/pyobject.png)  
特定对象的实现，视其大小是否固定，需要包含头部 `PyObject` 或 `PyVarObject`。因此，头文件准备了两个宏定义，方便其他对象使用：
```c
#define PyObject_HEAD          PyObject ob_base;
#define PyObject_VAR_HEAD      PyVarObject ob_base;
``` 
例如对于 `float` 对象， 它的定义包含了`PyObject`，增加了一个 double 类型字段 `ob_fval` 用来存值
```c
typedef struct {
    PyObject_HEAD
    double ob_fval;
} PyFloatObject;
``` 
而对于 list 对象 ，他的定义则包含了 `PyVarObject` . 
```c
typedef struct {
    PyObject_VAR_HEAD
    PyObject **ob_item;
    Py_ssize_t allocated;
} PyListObject;
```
`PyListObject` 增加了2个字段， 分别是：
>ob_item ，指向 动态数组 的指针，数组保存元素对象指针；  
>allocated ，动态数组总长度，即列表当前的 容量 ；  

![对比](https://images.chimission.cn/blog/python_list_float.png)


### 参考
[1.https://linux.die.net/diveintopython/html/getting_to_know_python/everything_is_an_object.html](https://linux.die.net/diveintopython/html/getting_to_know_python/everything_is_an_object.html)  
[2.https://stackoverflow.com/questions/865911/is-everything-an-object-in-python-like-ruby](https://stackoverflow.com/questions/865911/is-everything-an-object-in-python-like-ruby)  

