---
layout: post
title: 翻译Writing Idiomatic Python
categories: Python
description: 翻译如何编写地道的Python代码
keywords: Idiomatic, Python
---

# README

翻译Writing Idiomatic Python

翻译肯定用词不当和不严禁的地方，请轻拍！^_^

欢迎和我联系：

QQ：2165426394  

EMAIL：2165426394@qq.com

# 1. 控制结构和函数

## 1.1 if语句

### 1.1.1 避免直接和True、False或者None进行比较

对于任意对象，内建还是用户定义的，本身都有真假的判断。当判断条件是否为真时，主要依赖于对象在条件语句中的真假性。真假性判断是非常清晰的。所有的下列条件都判断为假：

* None
* False
* 数值类型的0
* 空的序列
* 空的字典
* 当调用对象的__len__或__nonzero__方法时，返回值是0或False

其他的所有情况都判断为真(大部分情况是隐式为真)。最后一个通过检查__len__或__nonzero__方法返回值的判断条件，允许自定义的类如何决定对象的真假。

应该像Python中的if语句一样，在判断语句中隐式地利用真假。譬如下面判断变量foo是否为真的语句

```python
if foo == True: 
```

可以简单地写成

```python
if foo:
```

写成下面的原因有很多。最明显的就是当代码发生变化时，譬如foo从True或False变成int时，if语句仍然可以工作。不过深层次而言，判断真假是基于对象间的equality和identity。使用==判断两个对象是否包含同样的值(具体由_eq属性定义)时。使用is判断两个对象是否同一对象(应用相同)。

注意：虽然有些情况下条件成立，就像比较对象的之间的相等性一样，这些是特殊情况，不要依赖。

因此，避免直接和False、Node以及像[], {}, ()这样的空序列比较真假。如果my_list列表类型为空，直接调用 

```python
if my_list:
```

 结果就是假的。

有些时候，直接和None比较是必须的，虽然不是推荐的方式。函数中的某一个形参默认值为None时，而实际实参传递非空时，必须要将形参和None进行比较：

```python
def insert_value(value, position=None):
    """Inserts a value into my container, optionally at thespecified position"""
    if position is not None:
        ...
```

如果写成

```python
if position:
```

问题出在哪里？如果想要在位置0插值，函数没有任何赋值操作，因为当position为0时，判断为假。注意和None比较时，应该总是使用is或is not，而不是==（参考PEP8）.

#### 1.1.1.1 不好的风格

```python
def number_of_evil_robots_attacking():
    return 10
def should_raise_shields():
    # "We only raise Shields when one or more giant robots attack,
    # so I can just return that value..."
    return number_of_evil_robots_attacking()
if should_raise_shields() == True:
    raise_shields()
    print('Shields raised')
else:
    print('Safe! No giant robots attacking')
```

#### 1.1.1.2 python的风格

```python
def number_of_evil_robots_attacking():
    return 10
def should_raise_shields():
    # "We only raise Shields when one or more giant robots attack,
    # so I can just return that value..."
    return number_of_evil_robots_attacking()
if should_raise_shields():
    raise_shields()
    print('Shields raised')
else:
    print('Safe! No giant robots attacking')
```

### 1.1.2 避免在复合条件语句中重复变量名称

当检测变量是否对应一些列值时，重复使用变量进行数值比较是不必要的。使用迭代的方式可以让代码更加清晰，并且可读性更好。

#### 1.1.2.1 不好的风格

```python
is_generic_name = False
name = 'Tom'
if name == 'Tom' or name == 'Dick' or name == 'Harry':
    is_generic_name = True
```

#### 1.1.2.2 python的风格

```python
name = 'Tom'
is_generic_name = name in ('Tom', 'Dick', 'Harry')
```

### 1.1.3 避免把条件分支代码和分号放到同一行

使用缩进来表示范围(python就是这么做的)可以更容易知道代码是否是条件语句的一部分。if, elif和else语句应单独写成一行，冒号后面不应再写代码。

#### 1.1.3.1 不好的风格

```python
name = 'Jeff'
address = 'New York, NY'
if name: print(name)
print(address)
```

#### 1.1.3.2 python的风格

```python
name = 'Jeff'
address = 'New York, NY'
if name:
    print(name)
print(address)
```

# 1.2 for循环

### 1.2.1 在循环中使用enumerate函数而非创建index索引变量

其它编程语言的程序员经常显式声明一个索引变量，用于跟踪循环中的容器。譬如，在C++中：

```C++
for (int i=0; i < container.size(); ++i)
{
    // Do stuff
}
```

在python中，有内建的enumerate函数用于处理这种情况。

#### 1.2.1.1 不好的风格

```python
my_container = ['Larry', 'Moe', 'Curly']
index = 0
for element in my_container:
    print ('{} {}'.format(index, element))
    index += 1
```

#### 1.2.1.2 python的风格

```python
my_container = ['Larry', 'Moe', 'Curly']
for index, element in enumerate(my_container):
    print ('{} {}'.format(index, element))
```

### 1.2.2 使用in关键字迭代可迭代的对象

使用缺少for_each风格语言的程序员通过索引访问元素迭代容器。Python的in关键字很优雅地处理了这个情况。

#### 1.2.2.1 不好的风格

```python
my_list = ['Larry', 'Moe', 'Curly']
index = 0
while index < len(my_list):
    print (my_list[index])
    index += 1
```

#### 1.2.2.2 python的风格

```python
my_list = ['Larry', 'Moe', 'Curly']
for element in my_list:
    print (element)
```

### 1.2.3 for循环结束后使用else执行代码

很少有人知道for循环语句后面可以跟随一段else语句。当迭代器执行结束后，else语句才会被执行，除非循环因为break语句提前结束了。break语句允许在循环内检查某些条件，当条件满足时，结束循环，否则，继续执行直到循环结束(无论如何最终else语句还是需要执行的)。这样可以避免通过在循环内增加标志位来检查条件是否满足。

在下面的情况下，代码用于检测用户注册的邮件地址是否合法（一个用户可以注册多个地址）。python风格的代码比较简洁，得益于不用处理has_malformed_email_address标志。更何况，即使一个程序员不熟悉for .. else语法，他们也可以很容易地看懂代码。

#### 1.2.3.1 

```python
for user in get_all_users():
    has_malformed_email_address = False
    print ('Checking {}'.format(user))
    for email_address in user.get_all_email_addresses():
        if email_is_malformed(email_address):
            has_malformed_email_address = True
            print ('Has a malformed email address!')
            break
    if not has_malformed_email_address:
        print ('All email addresses are valid!')
``` 

#### 1.2.3.2

```python
for user in get_all_users():
    print ('Checking {}'.format(user))
    for email_address in user.get_all_email_addresses():
        if email_is_malformed(email_address):
            print ('Has a malformed email address!')
            break
    else:
        print ('All email addresses are valid!')
```

## 1.3 函数

### 1.3.1 避免使用'', []和{}作为函数参数的默认值

虽然在python教程中明确提到了这一点，很多人表示诧异，即使有经验的开发人员。简而言之，函数参数的默认值首选None，而非[]。下面是Python教程对这个问题的处理：

#### 1.3.1.1 不好的风格

```python
# The default value [of a function] is evaluated only once.
# This makes a difference when the default is a mutable object
# such as a list, dictionary, or instances of most classes. For
# example, the following function accumulates the arguments
# passed to it on subsequent calls.
def f(a, L=[]):
    L.append(a)
    return L
print(f(1))
print(f(2))
print(f(3))
# This will print
#
# [1]
# [1, 2]
# [1, 2, 3]
```

#### 1.3.1.2 python的风格

```python
# If you don't want the default to be shared between subsequent
# calls, you can write the function like this instead:
def f(a, L=None):
    if L is None:
        L = []
    L.append(a)
    return L
print(f(1))
print(f(2))
print(f(3))
# This will print
# [1]
# [2]
# [3]
```

### 1.3.2 使用*args和**kwargs接受任意多的参数

通常情况下，函数需要接受任意的位置参数列表或关键字参数，使用它们中的一部分，并将其余的部分传递给其他函数。 使用*args和**kwargs作为参数允许函数接受任意的位置和关键字参数列表。在维护API的向后兼容性时，这个习性也很有用。如果函数接受任意参数，那么可以在新版本中自由地添加新的参数而不会破会现有使用较少参数的代码。只要一切文档记录正确，函数的实际参数是什么并没有多少影响。

#### 1.3.2.1 不好的风格

```python
def make_api_call(foo, bar, baz):
    if baz in ('Unicorn', 'Oven', 'New York'):
        return foo(bar)
    else:
        return bar(foo)
# I need to add another parameter to `make_api_call`
# without breaking everyone's existing code.
# I have two options...
def so_many_options():
    # I can tack on new parameters, but only if I make
    # all of them optional...
    def make_api_call(foo, bar, baz, qux=None, foo_polarity=None,
                baz_coefficient=None, quux_capacitor=None,
                bar_has_hopped=None, true=None, false=None,
                file_not_found=None):
        # ... and so on ad infinitum
        return file_not_found
def version_graveyard():
    # ... or I can create a new function each time the signature
    # changes.
    def make_api_call_v2(foo, bar, baz, qux):
        return make_api_call(foo, bar, baz) - qux
    def make_api_call_v3(foo, bar, baz, qux, foo_polarity):
        if foo_polarity != 'reversed':
            return make_api_call_v2(foo, bar, baz, qux)
        return None

def make_api_call_v4(
        foo, bar, baz, qux, foo_polarity, baz_coefficient):
    return make_api_call_v3(
        foo, bar, baz, qux, foo_polarity) * baz_coefficient
def make_api_call_v5(
        foo, bar, baz, qux, foo_polarity,
        baz_coefficient, quux_capacitor):
    # I don't need 'foo', 'bar', or 'baz' anymore, but I have to
    # keep supporting them...
    return baz_coefficient * quux_capacitor
def make_api_call_v6(
        foo, bar, baz, qux, foo_polarity, baz_coefficient,
        quux_capacitor, bar_has_hopped):
    if bar_has_hopped:
        baz_coefficient *= -1
    return make_api_call_v5(foo, bar, baz, qux,
                    foo_polarity, baz_coefficient,
                    quux_capacitor)
def make_api_call_v7(
        foo, bar, baz, qux, foo_polarity, baz_coefficient,
        quux_capacitor, bar_has_hopped, true):
    return true
def make_api_call_v8(
        foo, bar, baz, qux, foo_polarity, baz_coefficient,
        quux_capacitor, bar_has_hopped, true, false):
    return false
def make_api_call_v9(
        foo, bar, baz, qux, foo_polarity, baz_coefficient,
        quux_capacitor, bar_has_hopped,
        true, false, file_not_found):
    return file_not_found
```

#### 1.3.2.2 python的风格

```python
def make_api_call(foo, bar, baz):
    if baz in ('Unicorn', 'Oven', 'New York'):
        return foo(bar)
    else:
        return bar(foo)
# I need to add another parameter to `make_api_call`
# without breaking everyone's existing code.
# Easy...
def new_hotness():
    def make_api_call(foo, bar, baz, *args, **kwargs):
        # Now I can accept any type and number of arguments
        # without worrying about breaking existing code.
        baz_coefficient = kwargs['the_baz']
        # I can even forward my args to a different function without
        # knowing their contents!
        return baz_coefficient in new_function(args)    
```

# 2. 使用数据

# 2.1 列表

### 2.1.1 使用列表推导创建基于现有列表的新列表

机智地使用列表推导，可以使得基于现有数据构建列表的代码很清晰。尤其当进行一些条件检测和转换时。

使用列表推导（或者使用生成器表达式）通常还会带来性能上的提升，这是因为cPython的解释器的优化。

####2.1.1.1 不好的风格

```python
some_other_list = range(10)
some_list = list()
for element in some_other_list:
    if is_prime(element):
        some_list.append(element + 5)
```

#### 2.1.1.2 python的风格

```python
some_other_list = range(10)
some_list = [element + 5
                for element in some_other_list
                if is_prime(element)]
```

#### 2.1.2 使用*操作符来表示列表的其余部分

通常情况下，尤其是在处理函数参数时，这是很有用的，可以提取列表头部（或尾部）的一些元素，然后剩下的后面使用。Python2没有简单的方法完成这一点，可以通过切片来模拟。Python3允许在左边使用*运算符用来表示列表的其余部分。

#### 2.1.2.1 不好的风格

```python
some_list = ['a', 'b', 'c', 'd', 'e']
(first, second, rest) = some_list[0], some_list[1], some_list[2:]
print(rest)
(first, middle, last) = some_list[0], some_list[1:-1], some_list[-1]
print(middle)
(head, penultimate, last) = some_list[:-2], some_list[-2], some_list[-1]
print(head)
```

#### 2.1.2.2 python的风格

```python
some_list = ['a', 'b', 'c', 'd', 'e']
(first, second, *rest) = some_list
print(rest)
(first, *middle, last) = some_list
print(middle)
(*head, penultimate, last) = some_list
print(head)
```

# 2.2 字典

### 2.2.1 使用dict.get方法的默认参数来提供默认值

在dict.get的定义中经常被忽略的是默认参数。没有使用默认（或collections.defaultdict类）值，代码将会被if语句搞晕。记住，要力求清晰。

#### 2.2.1.1 不好的风格

```python
log_severity = None
if 'severity' in configuration:
    log_severity = configuration['severity']
else:
    log_severity = 'Info'
```

#### 2.2.1.2 python的风格

```python
log_severity = configuration.get('severity', 'Info')
```

#### 2.2.2 使用字典推导来更清晰和有效地构建字典

列表推导是python知名的构造方式，鲜为人知的是字典推导。不过，目的都是一样的：使用容易理解的推导语法构造字典。

#### 2.2.2.1 不好的风格

```python
user_email = {}
for user in users_list:
    if user.email:
        user_email[user.name] = user.email
```

####2.2.2.2 python的风格

```python
user_email = {user.name: user.email
                for user in users_list if user.email}
```

# 2.3 字符串

### 2.3.1 倾向使用format函数构造字符串

有三种方式格式化字符串（就是创建一个由硬编码字符串和字符串变量混合而成的字符串）。最容易不过不好的方式是通过+运算符连接静态字符串和变量。使用“老式”的字符串格式化方法相对好一点。它就其它语言的printf一样，使用格式字符串和%运算符来填充值。格式化字符串的最清晰和最习惯的方式是使用format函数。就像老式的格式化一样，它利用了格式字符串并用值来替换其中的占位符。两者相似之处仅此而言。通过format函数，代码可以使用具名占位符、访问其中的属性、控制填充空白和字符串宽度，以及其它内容等。format函数使得字符串格式化更加清晰和简洁。

#### 2.3.1.1 不好的风格

```python
def get_formatted_user_info_worst(user):
    # Tedious to type and prone to conversion errors
    return 'Name: ' + user.name + ', Age: ' + \
            str(user.age) + ', Sex: ' + user.sex
def get_formatted_user_info_slightly_better(user):
    # No visible connection between the format string placeholders
    # and values to use. Also, why do I have to know the type?
    # Don't these types all have __str__ functions?
    return 'Name: %s, Age: %i, Sex: %c' % (
            user.name, user.age, user.sex)
```

#### 2.3.1.2 python的风格

```python
def get_formatted_user_info(user):
    # Clear and concise. At a glance I can tell exactly what
    # the output should be. Note: this string could be returned
    # directly, but the string itself is too long to fit on the
    # page.
    output = 'Name: {user.name}, Age: {user.age}'
        ', Sex: {user.sex}'.format(user=user)
    return output
```

### 2.3.2 使用.join函数将列表元素转换为单一字符串

这种方式更快，占用更少内存，很多地方都在使用。请注意，两个引号代表想要连接成字符串的列表元素之间的分隔符。''表示连接的列表元素之间没有任何字符内容。

#### 2.3.2.1 不好的风格

```python
result_list = ['True', 'False', 'File not found']
result_string = ''
for result in result_list:
    result_string += result
```
    
#### 2.3.2.2 python的风格

```python
result_list = ['True', 'False', 'File not found']
result_string = ''.join(result_list)
```

### 2.3.3 使用链式的字符串函数是一些列的字符串转换更加清晰

当在一些数据上应用一些简单的数据变换时，单一表达式的链式调用相比那些通过临时变量进行一步步转换的方式更加清晰。不过，太多的链接可能使得代码难以掌控。一个比较好的规则是函数之间的连接不超过三个。

#### 2.3.3.1 不好的风格

```python
book_info = ' The Three Musketeers: Alexandre Dumas'
formatted_book_info = book_info.strip()
formatted_book_info = formatted_book_info.upper()
formatted_book_info = formatted_book_info.replace(':', ' by')
```

#### 2.3.3.2 python的风格

```python
book_info = ' The Three Musketeers: Alexandre Dumas'
formatted_book_info = book_info.strip().upper().replace(':', ' by')
```

# 2.4 类

### 2.4.1 在函数名和变量名中使用下划线来帮助标记为私有数据

Python类中的所有属性，不论是数据还是函数，本质上都是公有的。用户可以在类定义完成后自动地添加属性。此外，如果这个类是为了继承而来的，子类可能在不知不觉中更改基类的属性。最后，告知用户关于类的某些部分逻辑上公开的（不会变得向后不兼容），而其它属性是纯粹的内部实现，不应该被使用本类的客户端代码直接使用，关于这一点是非常有用的。

一些被广泛遵循的约定已经出现，使得作者的意图更加明确，有助于避免无意的命名冲突。于这两种用法，虽然普遍认为是公共约定，但是事实上在使用中会使解释器也产生不同的行为。

首先，用单下划线开始命名的表明是受保护的属性，用户不应该直接访问。其次，用两个连续地下划线开头的属性，表明是私有的，即使子类都不应该访问。当然了，这些更多的是约定，并不能真正阻止用户访问到这些私有的属性，但这些约定在整个Python社区中被广泛使用，不太可能遇到那些故意不遵循的开发者。从某种角度上来说这也是Python里用一种办法完成一件事情的哲学体现。

之前，本文提到过单下划线和双下划线不仅仅是使用习惯问题，一些开发者意识到这种写法是有实际作用的。以单下划线开头的变量在import *时不会被导入。以双下划线开头的变量则会触发Python中name mangling，如果Foo是一个类，那么Foo中定义的\_\_bar()方法将会被展开成_classname__attributename.

#### 2.4.1.1 不好的风格

```python
class Foo():
    def __init__(self):
        self.id = 8
        self.value = self.get_value()
    def get_value(self):
        pass
    def should_destroy_earth(self):
        return self.id == 42

class Baz(Foo):
    def get_value(self, some_new_parameter):
        """Since 'get_value' is called from the base class's
        __init__ method and the base class definition doesn't
        take a parameter, trying to create a Baz instance will
        fail.
        """
        pass
class Qux(Foo):
    """We aren't aware of Foo's internals, and we innocently
    create an instance attribute named 'id' and set it to 42.
    This overwrites Foo's id attribute and we inadvertently
    blow up the earth.
    """
    def __init__(self):
        super(Qux, self).__init__()
        self.id = 42
        # No relation to Foo's id, purely coincidental
q = Qux()
b = Baz() # Raises 'TypeError'
q.should_destroy_earth() # returns True
q.id == 42 # returns True
```

#### 2.4.1.2 python的风格

```python
class Foo():
    def __init__(self):
        """Since 'id' is of vital importance to us, we don't
        want a derived class accidentally overwriting it. We'll
        prepend with double underscores to introduce name
        mangling.
        """
        self.__id = 8
        self.value = self.__get_value() # Our 'private copy'
    def get_value(self):
        pass

    def should_destroy_earth(self):
        return self.__id == 42
    # Here, we're storing an 'private copy' of get_value,
    # and assigning it to '__get_value'. Even if a derived
    # class overrides get_value is a way incompatible with
    # ours, we're fine
    __get_value = get_value
class Baz(Foo):
    def get_value(self, some_new_parameter):
            pass
class Qux(Foo):
    def __init__(self):
        """Now when we set 'id' to 42, it's not the same 'id'
        that 'should_destroy_earth' is concerned with. In fact,
        if you inspect a Qux object, you'll find it doesn't
        have an __id attribute. So we can't mistakenly change
        Foo's __id attribute even if we wanted to.
        """
        self.id = 42
        # No relation to Foo's id, purely coincidental
        super(Qux, self).__init__()
q = Qux()
b = Baz() # Works fine now
q.should_destroy_earth() # returns False
q.id == 42 # returns True
with pytest.raises(AttributeError):
    getattr(q, '__id')        

```

### 2.4.2 在类中定义__str__方法用于可读性显示

当定义可能被print()方法使用的类时，默认的Python呈现方法帮助不大。通过定义__str__方法，可以控制类实例打印呈现出来的内容。

#### 2.4.2.1 不好的风格

```python
class Point():
    def __init__(self, x, y):
        self.x = x
        self.y = y
p = Point(1, 2)
print (p)
# Prints '<__main__.Point object at 0x91ebd0>'
```

2.4.2.2 python的风格

```python
class Point():
    def __init__(self, x, y):
        self.x = x
        self.y = y
    def __str__(self):
        return '{0}, {1}'.format(self.x, self.y)
p = Point(1, 2)
print (p)
# Prints '1, 2'
```

# 2.5 集合

### 2.5.1 使用集合来消除可迭代容器中的重复项

字典或列表中有重复项很常见。大公司所有员工的姓氏中，同样的姓氏会不止一次地出现。如果用列表来处理唯一的姓氏，工作量将很大。集合三个方面的特性可以完美回答上述问题：

1. 集合仅包含唯一的元素
2. 集合中添加已存在的元素会被忽略
3. 可迭代容器中可以hash的元素都可以构建到集合中

继续上面的例子，有一个接受序列参数的display函数，可以以某一格式显示参数中的元素。从原始列表创建集合后，是否需要改变display函数？

答案是：否。假定我们的display函数实现是合理的，那么集合可以直接替换列表。这得益于集合事实上像列表一样，是可迭代的，可以在循环、列表推导中等使用。

2.5.1.1 不好的风格

```python
unique_surnames = []
for surname in employee_surnames:
    if surname not in unique_surnames:
        unique_surnames.append(surname)
def display(elements, output_format='html'):
    if output_format == 'std_out':
        for element in elements:
            print(element)
    elif output_format == 'html':
        as_html = '<ul>'
    for element in elements:
        as_html += '<li>{}</li>'.format(element)
        return as_html + '</ul>'
    else:
        raise RuntimeError('Unknown format {}'.format(output_format))
```

2.5.1.2 python的风格

```python
unique_surnames = set(employee_surnames)
def display(elements, output_format='html'):
    if output_format == 'std_out':
        for element in elements:
            print(element)
    elif output_format == 'html':
        as_html = '<ul>'
        for element in elements:
            as_html += '<li>{}</li>'.format(element)
            return as_html + '</ul>'
    else:
    raise RuntimeError('Unknown format {}'.format(output_format))
```

### 2.5.2 使用集合推导可以很简洁地生成集合

集合的推导是Python中相对比较新的概念，应此，常常被忽略。就像通过列表推导产生列表一样，集合也可以由集合推导产生。事实上，两种语法几乎是相同的，除了分界符。

#### 2.5.2.1 不好的风格

```python
users_first_names = set()
for user in users:
    users_first_names.add(user.first_name)
```

#### 2.5.2.2 python的风格

```python
users_first_names = {user.first_name for user in users}
```

### 2.5.3 理解和使用数学集合操作

集合是非常容易理解的数据结构。集合类似有索引没有值的字典，集合类实现了Iterable和Container接口。因此，集合可以在for循环和in语句中使用。

之前不了解集合数据类型的程序员，可能不能灵活运用。了解它们的关键是了解他们的数学起源。集合论是学习集合set的数学基础，了解基本的数学集合操作是发挥集合set能力的关键。

不用担心，无需深入理解和使用数学上的集合。仅需要记住简单的几个操作：

**并集**       集合A和B中的元素，或者说A和B中所有的元素（Python中写法：A | B）

**交集**       A和B中同时包含的元素（Python中写法：A & B）

**补集(差集)**  A中但是不再B中的元素（Python中写法：A - B）

注意：这里A和B之间的顺序是有影响的，A - B 不一定和 B - A相同。

**对称差集**   A或B中的元素，但是不能同时在A和B中（Python中写法：A ^ B）

当处理数据列表时，一个常见的任务就是找出同时出现在所有列表中的元素。任何时候，当需要基于序列之间的关系，从两个或多个序列中挑选元素时，请使用集合。

下面，探讨一些典型的例子：

#### 2.5.3.1 不好的风格

```python
def get_both_popular_and_active_users():
    # Assume the following two functions each return a
    # list of user names
    most_popular_users = get_list_of_most_popular_users()
    most_active_users = get_list_of_most_active_users()
    popular_and_active_users = []
    for user in most_active_users:
        if user in most_popular_users:
            popular_and_active_users.append(user)
    return popular_and_active_users
```

#### 2.5.3.2 python的风格

```python
def get_both_popular_and_active_users():
    # Assume the following two functions each return a
    # list of user names
    return(set(
        get_list_of_most_active_users()) & set(
            get_list_of_most_popular_users()))
```

# 2.6 生成器

#2.6.1 使用生成器惰性加载无穷序列

通常能够提供一种可以迭代无穷序列的方式是很有用的，否则，需要提供一个异常昂贵开销的接口来实现，并且用户还需花时间来等待列表的构建。

面临这些情况，使用生成器就很有帮助。生成器是协程的一种特殊类型，它返回iterable类型。生成器的状态会被保存，当再次调用生成器时，可以继续上次离开时的状态。下面的例子，介绍了如何生用生成器处理上面提到的情况。

#### 2.6.1.1 不好的风格

```python
def get_twitter_stream_for_keyword(keyword):
    """Get's the 'live stream', but only at the moment
    the function is initially called. To get more entries,
    the client code needs to keep calling
    'get_twitter_livestream_for_user'. Not ideal.
    """
    imaginary_twitter_api = ImaginaryTwitterAPI()
    if imaginary_twitter_api.can_get_stream_data(keyword):
        return imaginary_twitter_api.get_stream(keyword)
current_stream = get_twitter_stream_for_keyword('#jeffknupp')
for tweet in current_stream:
    process_tweet(tweet)
# Uh, I want to keep showing tweets until the program is quit.
# What do I do now? Just keep calling
# get_twitter_stream_for_keyword? That seems stupid.
def get_list_of_incredibly_complex_calculation_results(data):
    return [first_incredibly_long_calculation(data),
            second_incredibly_long_calculation(data),
            third_incredibly_long_calculation(data),
            ]
```

#### 2.6.1.2 python的风格

```python
def get_twitter_stream_for_keyword(keyword):
    """Now, 'get_twitter_stream_for_keyword' is a generator
    and will continue to generate Iterable pieces of data
    one at a time until 'can_get_stream_data(user)' is
    False (which may be never).
    """
    imaginary_twitter_api = ImaginaryTwitterAPI()
    while imaginary_twitter_api.can_get_stream_data(keyword):
        yield imaginary_twitter_api.get_stream(keyword)
# Because it's a generator, I can sit in this loop until
# the client wants to break out
for tweet in get_twitter_stream_for_keyword('#jeffknupp'):
    if got_stop_signal:
        break
    process_tweet(tweet)
def get_list_of_incredibly_complex_calculation_results(data):
    """A simple example to be sure, but now when the client
    code iterates over the call to
    'get_list_of_incredibly_complex_calculation_results',
    we only do as much work as necessary to generate the
    current item.
    """
    yield first_incredibly_long_calculation(data)
    yield second_incredibly_long_calculation(data)
    yield third_incredibly_long_calculation(data)
```


### 2.6.2 倾向使用生成器表达式到列表推导中用于简单的迭代

在处理序列时，通常需要在轻微修改的序列版本上迭代一次。譬如，你可以需要以首字母大写的方式打印所有用户的姓。

第一本能可能是就地构建并迭代序列。列表推导可能是理想的，不过Python有个内建的更好的处理方式：生成器表达式。

有什么不同？列表推导构建列表对象并立即填充所有的元素。对于大型的列表，可能非常耗资源。生成器返回生成器表达式，换句话说，按需产生每一个元素。你想要大写字母转换的列表？问题不大。但是，如果你想要写出国会图书馆中已知的每本书的标题时，可能在列表推导的过程中就会耗尽系统内存，而生成器表达式可以泰然处之。生成器表达式工作的逻辑扩展方式使得我们可以把它们应用到无穷序列中。

#### 2.6.2.1 不好的风格

```python
for uppercase_name in [name.upper() for name in get_all_usernames()]:
    process_normalized_username(uppercase_name)
```

#### 2.6.2.2 python的风格

```python
for uppercase_name in (name.upper() for name in get_all_usernames()):
    process_normalized_username(uppercase_name)
```

# 2.7 上下文管理

### 2.7.1 使用上下文管理器确保资源的合理管理

类似C++和D语言的RAII原则，上下文管理器(和with语句一起使用)可以让资源的管理更加安全和清晰。一个典型的例子就是文件IO操作。

看一下下面不好的代码。如果发生了异常会，怎么样？因为代码中没有捕获异常，异常会向上传递。代码中的退出点可能被跳过，导致没法关闭已经打开的文件。

标准库中有很多支持和使用上下文管理器的类。此外，通过定义__enter__和__exit__方法，用户自定义类也可以很容易地支持上下文管理器。函数可以通过contextlib模块，用上下文管理器来包装。

### 2.7.1.1 不好的风格

```python
file_handle = open(path_to_file, 'r')
for line in file_handle.readlines():
    if raise_exception(line):
        print('No! An Exception!')
```

#### 2.7.1.2 python的风格

```python
with open(path_to_file, 'r') as file_handle:
for line in file_handle:
    if raise_exception(line):
        print('No! An Exception!')
```

# 2.8 元组

### 2.8.1 使用元组解包(unpack)数据

在Python中，可使用解包数据实现多重赋值。这类似于LISP中的desctructuring bind。

### 2.8.1.1 不好的风格

```python
list_from_comma_separated_value_file = ['dog', 'Fido', 10]
animal = list_from_comma_separated_value_file[0]
name = list_from_comma_separated_value_file[1]
age = list_from_comma_separated_value_file[2]
output = ('{name} the {animal} is {age} years old'.format(
    animal=animal, name=name, age=age))
```

#### 2.8.1.2 python的风格

```python
list_from_comma_separated_value_file = ['dog', 'Fido', 10]
(animal, name, age) = list_from_comma_separated_value_file
output = ('{name} the {animal} is {age} years old'.format(
    animal=animal, name=name, age=age))
```

### 2.8.2 在元组中使用_占位符表示要忽略的值

当将元组中内容顺序赋值到一些变量中时，很多时候并不是所有的数据都是需要的。与其创建令人困惑的废弃变量，不如使用_占位符告诉读者，该数据是没用的。

###2.8.2.1 不好的风格

```python
(name, age, temp, temp2) = get_user_info(user)
if age > 21:
    output = '{name} can drink!'.format(name=name)
# "Wait, where are temp and temp2 being used?"
```

#### 2.8.2.2 python的风格

```python
(name, age, _, _) = get_user_info(user)
if age > 21:
    output = '{name} can drink!'.format(name=name)
# "Clearly, only name and age are interesting"
```

# 2.9 变量

### 2.9.1 在交互数值时避免使用临时变量

Python中交互数值时没有使用临时变量的理由。可以使用元组使得互换更加清晰。

#### 2.9.1.1 不好的风格

```python
foo = 'Foo'
bar = 'Bar'
temp = foo
foo = bar
bar = temp
```

#### 2.9.1.2 python的风格
```python
foo = 'Foo'
bar = 'Bar'
(foo, bar) = (bar, foo)
```

# 2. 组织代码

# 3.1 模块和包

Python语言支持面向对象编程，不过不是必须要使用的。很多有经验的Python程序员相对较少使用类和多态。有很多这方面的原因。

大部分存储在类中的属性可以用列表、字典和集合类型来表示。Python有很多优化（设计和实现层面）的内置函数和标准库来处理这些数据。类仅当需要的时候才使用，并且永远不要超过出API边界。

在Java语言中，类是封装的基本单元。每个文件都代表一个Java类，不管解决的问题是否有意义。如果有个方便的工具函数，那么将会封装到Utility类中。即使我们不能直观地理解Utility对象代表什么意思，也没有关系。当然，我夸大了，但重点是明确的，一旦习惯把所有内容封装到一个类中，就会很容易把这个概念带到其他的编程语言。

在Python中，一组相关的函数和数据通常封装在模块中。如果使用MVC的WEB框架来构建"Chirp"应用，可能会存在一个包含模型、视图和控制器的chirp包。如果"Chirp"是一个特别宏大的项目，代码库很大，这些模块可以很容易自己打包。 控制器包可能有一个持久性模块和处理模块。除了直觉上属于控制者之外，这些都不需要以任何方式相关。

如果所有这些模块都变成了类，那么互操作性立即就会变成了一个问题。 我们必须认真准确地确定我们将要对外公开的方法，状态如何更新，以及类支持的测试方法。不像一个字典或列表，我们必须编写代码来处理和持久性化对象。

请注意，在"Chirp"的描述中，没有任何需要使用类的地方。简单的导入语句使代码共享和封装变得简单。将状态显式作为参数传递给函数让一切变得松耦合。系统更容易接收，处理和转换数据流。

可以确切的说，类可能是更清晰或自然地表示一种东西的方式。在许多情况下，面向对象编程是一个很方便的范例，只是不要让它成为使用的唯一范例。


# 3.2 格式化

### 3.2.1 使用大写字母来声明全局常量

为了区分从定义在模块级别中(或者单个文件中的全局变量)导入的常量名字，都使用大写字母。

#### 3.2.1.1 不好的风格

```python
seconds_in_a_day = 60 * 60 * 24
# ...
def display_uptime(uptime_in_seconds):
    percentage_run_time = (
        uptime_in_seconds/seconds_in_a_day) * 100
    # "Huh!? Where did seconds_in_a_day come from?"
    return 'The process was up {percent} percent of the day'.format(
        percent=int(percentage_run_time))
# ...
uptime_in_seconds = 60 * 60 * 24
display_uptime(uptime_in_seconds)
```

#### 3.2.1.2 Python的风格

```python
SECONDS_IN_A_DAY = 60 * 60 * 24
# ...
def display_uptime(uptime_in_seconds):
    percentage_run_time = (
        uptime_in_seconds/SECONDS_IN_A_DAY) * 100
    # "Clearly SECONDS_IN_A_DAY is a constant defined
    # elsewhere in this module."
    return 'The process was up {percent} percent of the day'.format(
        percent=int(percentage_run_time))
    # ...
uptime_in_seconds = 60 * 60 * 24
display_uptime(uptime_in_seconds)
```

### 3.2.2 避免在同一行上放置多条语句

尽管语言定义允许放置多条，没有理由地使用，使得代码很难读，无法较好表述语句。当同一行上出现像if, else 或 elif 多条语句时，情形变得更加困惑。

#### 3.2.2.1 不好的风格

```python
if this_is_bad_code: rewrite_code(); make_it_more_readable();
```

#### 3.2.2.2 Python的风格

```python
if this_is_bad_code:
    rewrite_code()
    make_it_more_readable()
```

### 3.2.3 依据PEP8规则调整代码风格

Python定义了一套代码风格规则，俗称PEP8。如果你浏览Python工程的commit信息时，你将会发现，里面散落对PEP8的引用。原因很简单：如果我们都同意通用的命名和格式约定，那么所有的Python代码对新手和经验丰富的开发者而言，可读性更好。PEP8也许是Python社区中习语最明显的例子。阅读PEP，需要为编辑器安装PEP8的样式检查插件（一般编辑器都有），用其它程序员都欣赏的风格编写代码。下面列举几个例子：

<table>
    <tr>
        <th>标识符类型</th>
        <th>格式</th>
        <th>示例</th>
    </tr>
    <tr>
        <td>类</td>
        <td>首字母大写的Camel风格</td>
        <td>class StringManipulator():</td>
    </tr>
    <tr>
        <td>变量</td>
        <td>单词之间使用_连接</td>
        <td>joined_by_underscore = True</td>
    </tr>
    <tr>
        <td>函数</td>
        <td>单词之间使用_连接</td>
        <td>def multi_word_name(words):</td>
    </tr>
    <tr>
        <td>常量</td>
        <td>所有字母大写</td>
        <td>SECRET_KEY = 42</td>
    </tr>
</table>

一般其它没有列出的遵循变量和函数命名习惯：单词之间使用下划线连接。

# 3.3 可执行脚本

### 3.3.1 在脚本中使用sys.exit函数返回恰当的错误码

Python脚本应该是一个好的shell公民。在if \_\_name\_\_ == '\_\_main\_\_'后面执行一段代码，不返回任何结果，看起来挺神奇的。要避免这种做法。

在包含代码的运行脚本中创建一个主函数。如果有错误，在主函数中使用sys.exit返回错误码，否则，返回0。if \_\_name\_\_ =='\_\_main\_\_'下的唯一代码语句应该调用sys.exit作为主函数的返回值参数。

通过这样做，允许脚本在Unix管道中被使用，进行监控，失败的时候不需要自定义规则，可以被其他程序安全地调用。

#### 3.3.1.1 不好的风格

```python
if __name__ == '__main__':
    import sys
    # What happens if no argument is passed on the
    # command line?
    if len(sys.argv) > 1:
        argument = sys.argv[1]
        result = do_stuff(argument)
        # Again, what if this is False? How would other
        # programs know?
        if result:
            do_stuff_with_result(result)
```

#### 3.3.1.2 python的风格

```python
def main():
    import sys
    if len(sys.argv) < 2:
        # Calling sys.exit with a string automatically
        # prints the string to stderr and exits with
        # a value of '1' (error)
        sys.exit('You forgot to pass an argument')
    argument = sys.argv[1]
    result = do_stuff(argument)
    if not result:
        sys.exit(1)
        # We can also exit with just the return code
    do_stuff_with_result(result)
    # Optional, since the return value without this return
    # statment would default to None, which sys.exit treats
    # as 'exit with 0'
    return 0
# The three lines below are the canonical script entry
# point lines. You'll see them often in other Python scripts
if __name__ == '__main__':
    sys.exit(main())
```

### 3.3.2 在文件中使用if __name__ == '__main__'使得文件即可以被导入也可以直接运行

不像某些语言中的main()函数，Python没有内建的主入口点。相反，Python依赖加载的文件来执行语句。如果想要文件既可以作为模块导入，也可以作为一个独立的脚本，请使用 if \_\_name\_\_== '\_\_main\_\_'。

#### 3.3.2.1 不好的风格

```python
import sys
import os
FIRST_NUMBER = float(sys.argv[1])
SECOND_NUMBER = float(sys.argv[2])
def divide(a, b):
    return a/b
# I can't import this file (for the super
# useful 'divide' function) without the following
# code being executed.
if SECOND_NUMBER != 0:
    print(divide(FIRST_NUMBER, SECOND_NUMBER))
```

#### 3.3.2.2 Python的风格

```python
import sys
import os
def divide(a, b):
return a/b
# 仅当脚本直接运行的时候执行，作为模块导入的时候不执行
if __name__ == '__main__':
    first_number = float(sys.argv[1])
    second_number = float(sys.argv[2])
    if second_number != 0:
        print(divide(first_number, second_number))    
```

# 3.4 导入

### 3.4.1 相比相对导入，最好使用绝对导入

有两种导入模块的风格：绝对导入和相对导入。绝对导入依据sys.path指定导入模块位置（像\<package\>.\<module\>.\<submodule\>）。

相对模块导入的模块相对于当前模块在文件系统中的位置。假设有模块package.sub_package.module，想要导入package.other_module,可以使用.风格的相对导入语法：from ..other_module import foo。单一.表示当前模块包含的包。每一个额外的.意味着包的父亲一级，一级一个点。请注意，相对导入必须使用from ... import ...风格。import foo通常会被认为是绝对导入。

相应地，使用绝对导入应该这样写：import package.other_module（有可能使用子句给模块命名一个短的名字）。

那么，为什么更倾向于使用绝对导入呢？相对导入会搞混模块的命名空间。from foo import bar语句将bar绑定到自己模块的命名空间下。对于那些阅读你代码的读者而言，不太清楚bar来自哪里，尤其是当应用在复杂函数或大模块中。然而，boo.bar，很清晰地知道bar在哪里定义。Python编程常见问题甚至可以这样说：永远不要使用相对包导入。

#### 3.4.1.1 不好的风格

```python
# My location is package.sub_package.module
# and I want to import package.other_module.
# The following should be avoided:
from ...package import other_module
```

#### 3.4.1.2 Python的风格

```python
# My location is package.sub_package.another_sub_package.module
# and I want to import package.other_module.
# Either of the following are acceptable:
import package.other_module
import package.other_module as other
```

### 3.4.2 不要使用from foo import * 导入模块中的内容

考虑之前的习惯，该规则就会很明显。在导入的时候(from foo import \*)，使用\*号很容易混乱命名空间。如果自己定义和包中定义的名称出现冲突，可能会导致问题。

不过，如果想要从foo包中导入大量的名字怎么办？很简单。利用括号来组合导入语句。你没有必要从一个模块中写10行导入语句，命名空间仍然相对比较干净。

更好的是，只需简单使用绝对导入。如果包/模块名称很长，用as从句命名一个短点的变量名。

#### 3.4.2.1 不好的风格

```python
from foo import *
```

#### 3.4.2.2 Python的风格

```python
from foo import (bar, baz, qux, quux, quuux)
# or even better
import foo
```

#### 3.4.3 使用标准的顺序排列导入语句

随着工程规模的增长（尤其是那些WEB框架），导入语句的数量也会变大。让所有的导入语句在文件的开头，使用标准的顺序给import语句排序，并坚持下去。不过实际的顺序并不是太重要，以下是Python编程常见问题推荐的顺序：

1. 标准库模块
2. 安装在site-packages的第三方库
3. 本工程中的本地模块

许多人选择按照（大致）按字母顺序排列导入语句。其他人会认为这是可笑的。 实际上，这并不重要。 重要的是你选择一个标准的顺序，并遵守它。

#### 3.4.3.1 不好风格

```python
import os.path
# Some function and class definitions,
# one of which uses os.path
# ....
import concurrent.futures
from flask import render_template
# Stuff using futures and Flask's render_template
# ....
from flask import (Flask, request, session, g,
    redirect, url_for, abort,
    render_template, flash, _app_ctx_stack)
import requests
# Code using flask and requests
# ....
if __name__ == '__main__':
    # Imports when imported as a module are not so
    # costly that they need to be relegated to inside
    # an 'if __name__ == '__main__'' block...
    import this_project.utilities.sentient_network as skynet
    import this_project.widgets
    import sys
```
#### 3.4.3.2 Python的风格

```python
Easy to see exactly what my dependencies are and where to
# make changes if a module or package name changes
import concurrent.futures
import os.path
import sys
from flask import (Flask, request, session, g,
    redirect, url_for, abort,
    render_template, flash, _app_ctx_stack)
import requests
import this_project.utilities.sentient_network as skynet
import this_project.widgets
```

# 4. 一般建议

# 4.1 避免重造轮子

# 4.1.1 学习Python标准库

编写优雅代码的一部分是自由使用标准库。不知觉地重新编码实现标准库中的功能，可能是新手常做的时。Python通常被认为包含一切。标准库中的包涵盖很广泛的领域。

使用标准库有两个明显的好处。最明显的是，给你节省了很多时间，不需要从零开始。对那些阅读和维护代码的人同样重要，如果你使用他们都熟悉的内容，将会节省很多时间。

请记住，学习和编写优雅Python的目的是编写清晰，可维护和无缺陷的代码。没有什么比重用Python核心开发人员编写的代码更能确保你代码的质量了。

在标准库中发现并修复了的错误，你的代码将随着每一个Python版本的发布而得到改善，不需要动一根手指头。

### 4.1.2 学会使用PyPI（Python包索引）

如果Python标准库中没有你需要的包，PyPI将会派上用场。本文写作的时候，索引库中大约有27000个包。如果你想完成一个特殊的任务，不能在PyPI找到一个相关的包，很可能不会存在。

该索引是全文搜索的，包含Python2 和 Python 3。当然，并不是所有的软件包都是相同的（或同等维护）所以一定要检查包最后更新的时间。一个包含文档的包在像ReadTheDocs这样的网站上外部托管是一个好兆头，就像一个在GitHub或Bitbucket等网站上可以找到源代码。现在你发现了一个看起来很有前途的软件包，你怎么安装它？通过目前最流行的管理第三方软件包的工具pip。简单的pip install <软件包名称>命令将会下载最新的软件包，并安装到你的site-packages目录。如果需要包的bleeding edge版本，pip也能够直接从git或mercurial中安装。

如果你创建一个通用的有用的包，强烈建议你通过PyPI发布到Python社区。后来的Python开发者会感谢你的。

# 4.2 再提模块

### 4.2.1 学习itertools模块中的内容

如果经常访问像StackOverflow这些网站，你会注意到诸如“为什么Python没有以下明显有用的库函数”这类问题的答案时，几乎总是引用itertools模块。itertools提供的稳定性程序功能应该被视为基础建筑模块。更重要的是，itertools的文档有一个“食谱”提供了部分常用功能编码的优雅实现，所有这些都是使用itertools模块创建的。因为某些原因，很少有Python开发者知道“食谱”部分，实际上，itertools确实这么做的（Python文档中常常有这类宝典）。编写优雅代码的一部分内容是你要熟悉什么时候重零开始编写代码。

### 4.2.2 使用os.path模块处理目录路径

当编写一个简单的命令行脚本时，Python新手经常操作字符串处理文件路径。Python有完整的模块专门用于处理路径名称：os.path。使用os.path减少通用错误的风险，让代码可移植，更易懂。

#### 4.2.2.1 不好的风格

```python
from datetime import date
import os
filename_to_archive = 'test.txt'
new_filename = 'test.bak'
target_directory = './archives'
today = date.today()
os.mkdir('./archives/' + str(today))
os.rename(filename_to_archive, target_directory + '/' + str(today) + '/' + new_filename))
```

#### 4.2.2.2 优雅的风格

```python
from datetime import date
import os
current_directory = os.getcwd()
filename_to_archive = 'test.txt'
new_filename = os.path.splitext(filename_to_archive)[0] + '.bak'
target_directory = os.path.join(current_directory, 'archives')
today = date.today()
new_path = os.path.join(target_directory, str(today))
    if (os.path.isdir(target_directory)):
        if not os.path.exists(new_path):
            os.mkdir(new_path)
    os.rename(
        os.path.join(current_directory, filename_to_archive),
        os.path.join(new_path, new_filename))
```


写作不易，如果觉得本文帮助到了您，请打赏点咖啡钱。

# 微信扫描

![](/images/wxpay.jpg)