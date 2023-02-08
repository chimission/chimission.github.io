---
title: "Python 对象的秘密"
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

`_PyObject_HEAD_EXTRA` 是一个c语言宏，编译器会将其展开，实际上展开后`PyObject`有4个字段：ob_refcnt代表引用计数，ob_type代表类型指针，*_ob_next和*_ob_prev是一个双向链表用于跟踪所有 活跃堆对象，一般看源码用不到，这里我也没有深入学习。可以看到注释里说明了实际上在python里并没有任何对象是 `PyObject` 类型，但是任意一个对象都可以通过类型转换为 `PyObject*` 类型，从而可以实现使用统一的api管理不同的对象。注释里还提到了可变长对象 `PyVarObject*`，其实 `PyObject`只能用来表示定长对象例如float，毕竟它没有字段表示 size 大小的信息。在面对list，dict等变长对象时就需要拓展，于是 `PyVarObject` 在 `PyObject` 的基础上增加了 `ob_size` 字段，用于表示当前对象的长度。   
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

### PyTypeObject，类型的统一表示  
说完了对象，那么对象的类型又是怎么实现的？在python中类型也是对象，那么类型的实现会和 `PyObject` 有关系吗？ 接下来让我继续深入源码探究。我们回过头来看 `PyObject` 里包含了一个字段 `ob_type`， 它是一个 `PyTypeObject` 类型的指针，从命名上不难看出这个字段指向的就是对象的类型，`PyTypeObject` 在 `Include/object.h` 中定义，因为字段太多，我们这里只看最需要关心的部分： 
```c
typedef struct _typeobject {
    PyObject_VAR_HEAD
    const char *tp_name; /* For printing, in format "<module>.<name>" */
    // ...
    /* Attribute descriptor and subclassing stuff */
    struct _typeobject *tp_base;
    // ......
} PyTypeObject;
```  
类型对象 `PyTypeObject` 是一个 变长对象 ，包含变长对象头部宏 `PyObject_VAR_HEAD`，所以类型也是拓展自 `PyObject`， 从而实现了「类型也是对象」。`PyTypeObject` 就是 类型对象 在 Python 中的表现形式，对应着面向对象中 `类` 的概念。PyTypeObject 保存着对象的 元信息 ，描述对象的 类型 。其中我们关心的两个拓展字段：
>tp_name:类型的名字;  
>tp_base:类型的继承字段指向基类对象;  

但是 `PyTypeObject` 只是一个底层类型， 就像 `PyObject` 需要扩展成为特定的结构体才会被使用，例如float类型 `PyFloat_Type`， 定义在 `Object/floatobject.c`， 它拓展了 `PyTypeObject`： 
```c
PyTypeObject PyFloat_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "float",
    sizeof(PyFloatObject),
    0,
    (destructor)float_dealloc,                  /* tp_dealloc */
    // ...
    (reprfunc)float_repr,                       /* tp_repr */
    // ...
};
```
接下来我们以float对象和float类型为例，观察下各种结构体的引用关系，假设我们在python声明一个变量：
```python
In [1]: a=1.1
In [2]: type(a)
Out[2]: float 
```  
![float类型](/img/python_float_type.png) 
> ps: 我用的七牛云oss上传这张图片居然说我图里有敏感内容...试了几次都不行只能先放在本地了..无奈的是还找不到客服反馈 

变量 `a` 指向的对象是 `PyFloatObject` 结构体，除了公共头部字段 `ob_refcnt` 和 `ob_type` ，专有字段 `ob_fval` 保存了对应的数值。浮点 `类型对象` 是一个 `PyTypeObject` 结构体， 保存了类型名、内存分配信息以及浮点相关操作，实例对象 `ob_type` 字段指向类型对象 `PyFloat_Type`, `a` 变量只是一个指向实际对象的指针。  
可以看到在 `PyTypeObject` 结构体里也存在着表示类型的字段 `ob_type`，这就意味着类型对象本身也是具有类型的，这句话有点绕，看下实际代码：
```python
In [1]: a=1.1

In [2]: type(a)
Out[2]: float

In [3]: type(float)
Out[3]: type

In [3]: type(type)
Out[3]: type
```  
可以看到， type(float)返回了type类型，也就是说`float`类型对象也有自己的类型`type`， 在源码中也是如此，`PyFloat_Type`的 `ob_type` 字段指向的就是 `PyType_Type` 对象。  
```c
PyVarObject_HEAD_INIT(&PyType_Type, 0)  
```

### PyType_Type，类型的类型 
在Python中，类型也是一种对象。  
```c
PyTypeObject PyType_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "type",                                     /* tp_name */
    sizeof(PyHeapTypeObject),                   /* tp_basicsize */
    sizeof(PyMemberDef),                        /* tp_itemsize */
    (destructor)type_dealloc,                   /* tp_dealloc */

    // ...
    (reprfunc)type_repr,                        /* tp_repr */

    // ...
};
```
内建类型和自定义类对应的 PyTypeObject 对象都是这个通过 PyType_Type 创建的。 PyType_Type 在 Python 的类型机制中是一个至关重要的对象，它是所有类型的类型，称为 元类型 ( meta class )。

注意到， PyType_Type 将自己的 ob_type 字段设置成它自己(第 2 行)，这跟我们在 Python 中看到的行为是对应的`tpye(type)=type`

### 参考
[1.https://linux.die.net/diveintopython/html/getting_to_know_python/everything_is_an_object.html](https://linux.die.net/diveintopython/html/getting_to_know_python/everything_is_an_object.html)  
[2.https://stackoverflow.com/questions/865911/is-everything-an-object-in-python-like-ruby](https://stackoverflow.com/questions/865911/is-everything-an-object-in-python-like-ruby)  

