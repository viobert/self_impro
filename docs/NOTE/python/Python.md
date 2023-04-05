# python

还是一样，记录学习中的一些细节。

## 字符串

### 1 字符编码

计算机只处理数字，所以我们平常使用的字符串、字符就需要计算机进行编码转化成数字。世界上这么多语言，各国有各国的标准：计算美国是最早，他们编译了127个字符到`ASCII`；日本把日文编到`Shift_JIS`里；韩国把韩文编到`Euc-kr`，中国把中文编到`GB2312`编码......不同的语言混在一起，没有统一的标准就会出现乱码。那么如何统一？

因此，Unicode字符集应运而生。Unicode把所有语言都统一到一套编码里，这样就不会再有乱码问题了。

> 如果说“各个国家都在为自己文字独立编码”是百家争鸣，那么“建立世界统一的字符编码”则是一统江湖，谁都想来做这个武林盟主。早前就有两个机构试图来做这个事：
> (1) [国际标准化组织（ISO）](https://en.wikipedia.org/wiki/International_Organization_for_Standardization)，他们于1984年创建ISO/IEC JTC1/SC2/WG2工作组，试图制定一份“通用字符集”（Universal Character Set，简称UCS），并最终制定了[ISO 10646](https://en.wikipedia.org/wiki/Universal_Coded_Character_Set)标准。
> (2) [统一码联盟](https://en.wikipedia.org/wiki/Unicode_Consortium)，他们由Xerox、Apple等软件制造商于1988年组成，并且开发了[Unicode](https://en.wikipedia.org/wiki/Unicode)标准（The Unicode Standard，这个前缀Uni很牛逼哦---Unique, Universal, and Uniform）。
>
> 1991年前后，两个项目的参与者都认识到，世界不需要两个不兼容的字符集。于是，它们开始合并双方的工作成果，并为创立一个单一编码表而协同工作。从Unicode 2.0开始，Unicode采用了与ISO 10646-1相同的字库和字码；ISO也承诺，ISO 10646将不会替超出U+10FFFF的UCS-4编码赋值，以使得两者保持一致。两个项目仍都独立存在，并独立地公布各自的标准。不过由于Unicode这一名字比较好记，因而它使用更为广泛。
>
> 中国的设计史：GB2312集收纳了6763个字符，发现不够，设计了GBK，后来又加入了少数民族的字符成了GB18030

Unicode是一个字符集，同时也定义了编码规则（字符集大约是收纳的意思，每一个字符对应一个编号；而编码是指如何使用对应的编号和计算机二进制直接的转化存储）

一开始，Unicode使用UCS-2，发现还是不够就使用了UCS-4。UCS-4有个缺点，她太大了，导致当时并不流行。后来的后来，UTF-8出现，她是一种可变长的编码，解决了空间上的问题，如今十分流行。

其实还有很多种编码规则UTF-16，UTF-32......包括他们的编码规则是什么？感兴趣可以自己了解一下

对于单个字符的编码，Python提供了`ord()`函数获取字符的整数表示，`chr()`函数把编码转换为对应的字符：

```python
>>> ord('A')
65
>>> ord('中')
20013
>>> chr(66)
'B'
>>> chr(25991)
'文'

# 知道了字符的整数编码，使用整数的十六进制然后可以直接写出对应字符
>>> '\u6587'
'文'
```

由于Python的字符串类型是`str`，在内存中以Unicode表示，一个字符对应若干个字节。如果要在网络上传输，或者保存到磁盘上，就需要把`str`变为以字节为单位的`bytes`。

Python对`bytes`类型的数据用带`b`前缀的单引号或双引号表示：

```python
x = b'ABC'
```

要注意区分`'ABC'`和`b'ABC'`，前者是`str`，后者虽然内容显示得和前者一样，但`bytes`的每个字符都只占用一个字节。

以Unicode表示的`str`通过`encode()`方法可以编码为指定的`bytes`，例如：

```python
>>> 'ABC'.encode('ascii')
b'ABC'
>>> '中文'.encode('utf-8')
b'\xe4\xb8\xad\xe6\x96\x87'
>>> '中文'.encode('ascii')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)
```

纯英文的`str`可以用`ASCII`编码为`bytes`，内容是一样的，含有中文的`str`可以用`UTF-8`编码为`bytes`。含有中文的`str`无法用`ASCII`编码，因为中文编码的范围超过了`ASCII`编码的范围，Python会报错。

在`bytes`中，无法显示为ASCII字符的字节，用`\x##`显示。

反过来，如果我们从网络或磁盘上读取了字节流，那么读到的数据就是`bytes`。要把`bytes`变为`str`，就需要用`decode()`方法：

```python
>>> b'ABC'.decode('ascii')
'ABC'
>>> b'\xe4\xb8\xad\xe6\x96\x87'.decode('utf-8')
'中文'
```

如果`bytes`中包含无法解码的字节，`decode()`方法会报错：

```python
>>> b'\xe4\xb8\xad\xff'.decode('utf-8')
Traceback (most recent call last):
  ...
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xff in position 3: invalid start byte
```

如果`bytes`中只有一小部分无效的字节，可以传入`errors='ignore'`忽略错误的字节：

```python
>>> b'\xe4\xb8\xad\xff'.decode('utf-8', errors='ignore')
'中'
```

要计算`str`包含多少个字符，可以用`len()`函数：

```python
>>> len('ABC')
3
>>> len('中文')
2
```

`len()`函数计算的是`str`的字符数，如果换成`bytes`，`len()`函数就计算字节数：

```python
>>> len(b'ABC')
3
>>> len(b'\xe4\xb8\xad\xe6\x96\x87')
6
>>> len('中文'.encode('utf-8'))
6
```

可见，1个中文字符经过UTF-8编码后通常会占用3个字节，而1个英文字符只占用1个字节。

在操作字符串时，我们经常遇到`str`和`bytes`的互相转换。为了避免乱码问题，应当始终坚持使用UTF-8编码对`str`和`bytes`进行转换。

### 字符串格式化

1. `%`

    Python中`%`就是用来格式化字符串的，在字符串内部，`%s`代表字字符串替换，`%d`代表整数替换...

    ```python
    ## 占位符顺序对应
    
    print('%2d-%02d' % (3, 1))
    '''
     3-01
    3.14
    '''
    
    print('%.2f' % 3.1415926)
    '''
    3.14
    '''
    ```

2. `format()`

    另一种格式化字符串的方法是使用字符串的`format()`方法，它会用传入的参数依次替换字符串内的占位符`{0}`、`{1}`，可以自己制定顺序了。

    ```python
    print('Hello, {1}, 你的幸运数字是{0}'.format('小明', 7))
    # Hello, 7, 你的幸运数字是小明
    ```

3. `f-string`

最后一种格式化字符串的方法是使用以`f`开头的字符串，称之为`f-string`，它和普通字符串不同之处在于，字符串如果包含`{xxx}`，就会以对应的变量替换：

```py
>>> r = 2.5
>>> s = 3.14 * r ** 2
>>> print(f'The area of a circle with radius {r} is {s:.2f}')
The area of a circle with radius 2.5 is 19.62
```

上述代码中，`{r}`被变量`r`的值替换，`{s:.2f}`被变量`s`的值替换，并且`:`后面的`.2f`指定了格式化参数（即保留两位小数），因此，`{s:.2f}`的替换结果是`19.62`。

## List 和 Tuple

```python
# List  
classmates = ['Michael', 'Bob', 'Tracy']

classmates.append('H')    # 追加末尾
classmates.insert(1, 'H') # 追加到指定位置
classmates.pop(0)         # 删除指定位置元素，留空表示尾部


# Tuple  无法改动（本质是指向不变）
classmates = ('Michael', 'Bob', 'Tracy')

```

## Dict 和 Set

```python
# Dict  key-value
d = {'Michael': 95, 'Bob': 75, 'Tracy': 85}

if 'White' in d:      # 查找key值是否存在
d.get('White', value) # 查找key值是否存在，若不存在返回value,留空返回None
d.pop('Bob')          # 删除key

# Set 存储不重复元素，无序（内部不指定顺序）
>>> s = {1, 1, 2, 2, 3, 3}
>>> s
{1, 2, 3}
s.add(4)    # 添加元素
s.remove(4) # 删除元素
```

## 函数的参数

### 位置参数

按位置顺序对应

```python
def power(x, n):
    s = 1
    while n > 0:
        n = n - 1
        s = s * x
    return s
  
power(5, 2)
```

### 默认参数

将变化大的参数放在前面，变化小的参数放在后面。变化小的参数在传入时如果留空就接受默认值。

```python
def power(x, n=2):
    s = 1
    while n > 0:
        n = n - 1
        s = s * x
    return s
  
  
power(5) # 等价于 power(5, 2)
power(5, 3)
```

请注意，默认参数一定要是整型，字符串等不可变对象。

> 不可变对象，其本质是指向不变。字符串的`replace`函数看似能替换修改`str`，但其实他只是重新创建了一个`str`变量赋值。
>
> 其实不可变对象调用自身任意方法都不会修改自身内容，而是创建新的对象并返回。

```python
# 默认参数是可变对象时产生的问题
def add_end(L=[]):
    L.append('END')
    return L
  
  
add_end([1, 2, 3])
# [1, 2, 3, 'END']
add_end(['x', 'y', 'z'])
# ['x', 'y', 'z', 'END']


# 默认参数启用后，会出下面情况：
>>> add_end()
['END', 'END']
>>> add_end()
['END', 'END', 'END']
```

Python函数在定义的时候，默认参数`L`的值就被计算出来了，即`[ ]`，因为默认参数`L`也是一个变量，它指向对象`[ ]`，每次调用该函数，如果改变了`L`的内容，则下次调用时，默认参数的内容就变了，不再是函数定义时的`[ ]`了。

要修改上面的例子，我们可以用`None`这个不变对象来实现：

```python
def add_end(L=None):
    if L is None:
        L = []
    L.append('END')
    return L
```

现在，无论调用多少次，都不会有问题：

```python
>>> add_end()
['END']
>>> add_end()
['END']
```

为什么要设计`str`、`None`这样的不变对象呢？因为不变对象一旦创建，对象内部的数据就不能修改，这样就减少了由于修改数据导致的错误。**此外，由于对象不变，多任务环境下同时读取对象不需要加锁，同时读一点问题都没有。**我们在编写程序时，如果可以设计一个不变对象，那就尽量设计成不变对象。

### 可变参数 *args

不确定参数的数量，使用可变参数更加方便。

当我们想讲多个数当做参数传入，普通参数：

```python
def calc(numbers):
    sum1 = 0
    for n in numbers:
        sum1 = sum1 + n * n
    return sum1
  
 
calc([1, 2, 3]) # 此时参数需要传入list或者tuple
```

如果使用了可变参数：

```python
def calc(*numbers):
    sum2 = 0
    for n in numbers:
        sum2 = sum2 + n * n
    return sum2
  
  
  calc(1, 2, 3)
```

在函数内部，参数`numbers`接收到的是一个`tuple`，因此，函数代码完全不变。但是，调用该函数时，可以传入任意个参数，包括0个参数。

当然即便是你已经使用`list`，同样也使用了可变参数的函数，这时python允许再`list`或`tuple`前添加`*`作为可变参数传入函数。表示把`list`或`tuple`**中的全部元素传入**（如果是不加`*`则会将整个`list`作为元素装入可变参数的`tuple`中），此写法十分常见。

```python
def calc(*numbers):
    sum2 = 0
    for n in numbers:
        sum2 = sum2 + n * n
    return sum2
  
  
	x = [1, 2, 3]
	calc(*x)
```

### 关键字参数 **kw

#### 1 使用

可变参数允许你传入0个或任意个参数，这些可变参数在函数调用时自动组装为一个tuple。而关键字参数允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict。请看示例：

```python
def person(name, age, **kw):
    print('name:', name, 'age:', age, 'other:', kw)
```

函数`person`除了必选参数`name`和`age`外，还接受关键字参数`kw`。在调用该函数时，可以只传入必选参数：

```python
person('Michael', 30)
# name: Michael age: 30 other: {}
```

也可以传入任意个数的关键字参数：

```python
person('Bob', 35, city='Beijing')
# name: Bob age: 35 other: {'city': 'Beijing'}


person('Adam', 45, gender='M', job='Engineer')
# name: Adam age: 45 other: {'gender': 'M', 'job': 'Engineer'}
```

关键字参数有什么用？它可以扩展函数的功能。比如，在`person`函数里，我们保证能接收到`name`和`age`这两个参数，但是，如果调用者愿意提供更多的参数，我们也能收到。试想你正在做一个用户注册的功能，除了用户名和年龄是必填项外，其他都是可选项，利用关键字参数来定义这个函数就能满足注册的需求。

和可变参数类似，也可以先组装出一个dict，然后，把该dict转换为关键字参数传进去：

```python
extra = {'city': 'Beijing', 'job': 'Engineer'}
person('Jack', 24, city=extra['city'], job=extra['job'])
# 关键字参数传入一定要传入参数名字

# name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}
```

当然，上面复杂的调用可以用简化的写法：

```python
extra = {'city': 'Beijing', 'job': 'Engineer'}
person('Jack', 24, **extra)


# name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}
```

`**extra`表示把`extra`这个dict的所有key-value用关键字参数传入到函数的`**kw`参数，`kw`将获得一个dict，注意`kw`获得的dict是`extra`的一份拷贝，对`kw`的改动不会影响到函数外的`extra`。

#### 2 检查关键字存在

kw相当于组成了一个`tuple`，那么检查用`in`即可。

```python
def person(name, age, **kw):
    if 'city' in kw:
        # 有city参数
        pass
    if 'job' in kw:
        # 有job参数
        pass
    print('name:', name, 'age:', age, 'other:', kw)
```

#### 3 升级：命名关键字参数

命名关键字用于限制关键字参数的名字。

如，只接受`city`和`job`关键字的参数：

```python
def person(name, age, **kw):
  
  
# 改成  
def person(name, age, *, city, job):
```

当前面已经有可变参数时，则不需要加`*`的特殊分割符号。

```python
def person(name, age, *args, city, job):
    print(name, age, args, city, job)


lis = [1, 2]
person('Jack', 24, *lis, city='Beijing', job='x')
```

命名关键字可以有默认值

```python
def person(name, age, *, city='Beijing', job):
    print(name, age, city, job)
    
    
person('Jack', 24, job='Engineer')
# Jack 24 Beijing Engineer
```

### 参数组合使用

使用时前后顺序需要满足：位置参数（必选），默认参数，可变参数，命名关键字参数，关键字参数

但是实际这样组合并不好，很影响理解。

## 奇招

### 切片

记住左开右闭，

```python
L = ['Michael', 'Sarah', 'Tracy', 'Bob', 'Jack']

L[:3] == L[0:3]
# ['Michael', 'Sarah', 'Tracy']

L[1:3]
# ['Sarah', 'Tracy']

L[-2:]
# ['Bob', 'Jack']
# [-2:0] 没有效果，也就是说，如果你想输出最后n个元素，这是唯一的招数。虽然L[-1]可以去最后一个，但是切片是左开右闭的。

L[-2:-1]
# ['Bob']

# 前7；后7；全部；每2个取1个
L[:7]
L[-7:]
L[:]
L[:7:2]
```

tuple也是一种list，唯一区别是tuple不可变。因此，tuple也可以用切片操作，只是操作的结果仍是tuple：

```python
>>> (0, 1, 2, 3, 4, 5)[:3]
(0, 1, 2)
```

字符串`'xxx'`也可以看成是一种list，每个元素就是一个字符。因此，字符串也可以用切片操作，只是操作结果仍是字符串：

```python
>>> 'ABCDEFG'[:3]
'ABC'
>>> 'ABCDEFG'[::2]
'ACEG'
```

### 迭代

c语言的迭代，使用`for`配合使用指针，数组使用下标

python中针对迭代，用关键字`in`。这样即可不在乎下标，只拿到我想要的。

```python
d = {'a': 1, 'b': 2, 'c': 3}
for key in d:
	print(key)

'''
a
c
b
'''

# 当然，上面这样的dict，你可能想要的东西有三种key, value, key and value
for key in dict:
  pass
for value in dict.values():
  pass
for key, value in dict.items():
  pass

# list 
for i, value in enumerate(['a', 'b', 'c']):
  pass

# enumerate /ɪˈnjuːməreɪt/ v. 枚举
```

字符串等也可以用`in`迭代。想知道一个对象能不能迭代，python中给了方法：

```python
from collections.abc import Iterable

isinstance('hello', Iterable)
# True
isinstance([1, 2, 3], Iterable)
# True
isinstance(123, Iterable)
# False




# ps isinstance
x = 'abc'
y = 123
isinstance(x, str)
# True
isinstance(y, str)
# False
```

### 列表生成式

所谓列表生成式，就是简介的生成列表

```python
[x * x for x in range(1, 11)]
# [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

甚至可以加`if`判断

```python
[x * x for x in range(1, 11) if x % 2 == 0]
# [4, 16, 36, 64, 100]
```

甚至可以使用两层循环

```python
[m + n for m in 'ABC' for n in 'XYZ']
# ['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
```

甚至甚至可以用`if ... else`

用`if else`就有讲究了，你如果这样使用：

```python
[x for x in range(1, 11) if x % 2 == 0 else 0]
# ERROR!
```

这样使用`else`就是错误的，其实很好理解。跟在`for`后面的`if`作用是 筛选过滤，只有符合`if`的才会取出来完成`for`前面的表达式，加上了`else`就一定需要表达式来承接`else`的内容了，这可就不叫筛选了。

所以说，想写`else`就得写在表达式里，像这样：

```python
>>> [x if x % 2 == 0 else -x for x in range(1, 11)]
# [-1, 2, -3, 4, -5, 6, -7, 8, -9, 10]
```

### 生成器

> 通过列表生成式，我们可以直接创建一个列表。但是，受到内存限制，列表容量肯定是有限的。而且，创建一个包含100万个元素的列表，不仅占用很大的存储空间，如果我们仅仅需要访问前面几个元素，那后面绝大多数元素占用的空间都白白浪费了。
>
> 所以，如果列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢？这样就不必创建完整的list，从而节省大量的空间。在Python中，这种**一边循环一边计算的机制**，称为生成器：generator。

#### 创建generator

第一种方法很简单，只要把一个列表生成式的`[]`改成`()`：

```python
L = [x * x for x in range(10)]
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
g = (x * x for x in range(10))
# <generator object <genexpr> at 0x1022ef630>
```

创建`L`和`g`的区别仅在于最外层的`[]`和`()`，`L`是一个list，而`g`是一个generator。

我们可以直接打印出list的每一个元素，但我们怎么打印出generator的每一个元素呢？

如果要一个一个打印出来，可以通过`next()`函数获得generator的下一个返回值：

```python
>>> next(g)
0
>>> next(g)
1
>>> next(g)
4
>>> next(g)
9
>>> next(g)
16
>>> next(g)
25
>>> next(g)
36
>>> next(g)
49
>>> next(g)
64
>>> next(g)
81
>>> next(g)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

**我们讲过，generator保存的是算法，每次调用`next(g)`，就计算出`g`的下一个元素的值，直到计算到最后一个元素，没有更多的元素时，抛出`StopIteration`的错误。**

当然，上面这种不断调用`next(g)`实在是太变态了，正确的方法是使用`for`循环，因为generator也是可迭代对象：

```python
>>> g = (x * x for x in range(10))
>>> for n in g:
...     print(n)
... 
0
1
4
9
16
25
36
49
64
81
```

所以，我们创建了一个generator后，基本上永远不会调用`next()`，而是通过`for`循环来迭代它，并且不需要关心`StopIteration`的错误。

#### 何时使用generator

**generator非常强大。如果推算的算法比较复杂，用类似列表生成式的`for`循环无法实现的时候，还可以用函数来实现。**

比如，著名的斐波拉契数列（Fibonacci），除第一个和第二个数外，任意一个数都可由前两个数相加得到：

1, 1, 2, 3, 5, 8, 13, 21, 34, ...

斐波拉契数列用列表生成式写不出来，但是，用函数把它打印出来却很容易：

```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        print(b)
        a, b = b, a + b
        n = n + 1
    return 'done'
```

*注意*，赋值语句：

```python
a, b = b, a + b

# 上面相当于
t = (b, a + b) # t是一个tuple
a = t[0]
b = t[1]
```

上面的函数可以输出斐波那契数列的前N个数：

```python
>>> fib(6)
1
1
2
3
5
8
'done'
```

仔细观察，可以看出，`fib`函数实际上是定义了斐波拉契数列的推算规则，可以从第一个元素开始，推算出后续任意的元素，这种逻辑其实非常类似generator。

也就是说，上面的函数和generator仅一步之遥。要把`fib`函数变成generator函数，只需要把`print(b)`改为`yield b`就可以了：

```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
    return 'done'
```

这就是定义generator的另一种方法。如果一个函数定义中包含`yield`关键字，那么这个函数就不再是一个普通函数，而是一个generator函数，调用一个generator函数将返回一个generator：

```python
>>> f = fib(6)
>>> f
<generator object fib at 0x104feaaa0>
```

这里，最难理解的就是generator函数和普通函数的执行流程不一样。**普通函数是顺序执行，遇到`return`语句或者最后一行函数语句就返回。而变成generator的函数，在每次调用`next()`的时候执行，遇到`yield`语句返回，再次执行时从上次返回的`yield`语句处继续执行。**

举个简单的例子，定义一个generator函数，依次返回数字1，3，5：

```python
def odd():
    print('step 1')
    yield 1
    print('step 2')
    yield(3)
    print('step 3')
    yield(5)
```

调用该generator函数时，首先要生成一个generator对象，然后用`next()`函数不断获得下一个返回值：

```python
>>> o = odd()
>>> next(o)
step 1
1
>>> next(o)
step 2
3
>>> next(o)
step 3
5
>>> next(o)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

可以看到，`odd`不是普通函数，而是generator函数，在执行过程中，遇到`yield`就中断，下次又继续执行。执行3次`yield`后，已经没有`yield`可以执行了，所以，第4次调用`next(o)`就报错。

**请务必注意：调用generator函数会创建一个generator对象，多次调用generator函数会创建多个相互独立的generator。**



回到`fib`的例子，我们在循环过程中不断调用`yield`，就会不断中断。当然要给循环设置一个条件来退出循环，不然就会产生一个无限数列出来。

同样的，把函数改成generator函数后，我们基本上从来不会用`next()`来获取下一个返回值，而是直接使用`for`循环来迭代：

```python
>>> for n in fib(6):
...     print(n)
...
1
1
2
3
5
8
```

但是用`for`循环调用generator时，发现拿不到generator的`return`语句的返回值。如果想要拿到返回值，必须捕获`StopIteration`错误，返回值包含在`StopIteration`的`value`中：

```python
>>> g = fib(6)
>>> while True:
...     try:
...         x = next(g)
...         print('g:', x)
...     except StopIteration as e:
...         print('Generator return value:', e.value)
...         break
...
g: 1
g: 1
g: 2
g: 3
g: 5
g: 8
Generator return value: done
```

关于如何捕获错误，后面的错误处理还会详细讲解。

### 迭代器

我们上面说过，使用`isintance(*, Iterable)`可以检验一个对象是否是可迭代对象。

而迭代器不同，迭代器对象往往是一类数据流，它在需要下一个数据时才惰性的计算。它甚至可以表示一个无限大的数据流，这点`list`做不到。

+ 可以被`next()`函数调用并不断返回下一个值的对象称为迭代器：`Iterator`。
    + 可以被`for()`使用的对象都是`Iterable`类型，但是`Iterable`类型未必是`Iterator`类型。

我们上面讲过`generator`，它就是第一个典型的迭代器。不但可以作用于`for`循环，还可以被`next()`函数不断调用并返回下一个值，直到最后抛出`StopIteration`错误表示无法继续返回下一个值了。生成器都是`Iterator`对象，但`list`，`dict`，`str`虽然是`Iterable`，却不是`Iterator`。

```python
from collections.abc import Iterator

isinstance((x for x in range(10)), Iterator)
# True
isinstance([], Iterator)
# False
isinstance({}, Iterator)
# False
isinstance('abc', Iterator)
# False



# 把list、dict、str等Iterable变成Iterator可以使用iter()函数：
isinstance(iter([]), Iterator)
# True
isinstance(iter('abc'), Iterator)
# True
```

## 函数式编程

函数式编程（functional programming）是一种编程范式

[函数式编程FP 与 面向对象编程OOP](https://zhuanlan.zhihu.com/p/158828668)

### 高阶函数

1. 变量可以指向函数

```python
>> abs
# <built-in function abs>

# 允许
x = abs
```

2. 函数名也是变量

那么函数名是什么呢？函数名其实就是指向函数的变量！对于`abs()`这个函数，完全可以把函数名`abs`看成变量，它指向一个可以计算绝对值的函数！

如果把`abs`指向其他对象，会有什么情况发生？

```python
>>> abs = 10
>>> abs(-10)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'int' object is not callable
```

把`abs`指向`10`后，就无法通过`abs(-10)`调用该函数了！因为`abs`这个变量已经不指向求绝对值函数而是指向一个整数`10`！

**当然实际代码绝对不能这么写，这里是为了说明函数名也是变量。要恢复`abs`函数，请重启Python交互环境。**

注：由于`abs`函数实际上是定义在`import builtins`模块中的，所以要让修改`abs`变量的指向在其它模块也生效，要用`import builtins; builtins.abs = 10`。

3. 函数也可以传参

```python
def g(x, y, f):
  return f(x) + f(y)

g(-5, 6, abs)
# 11
```

#### map()/reduce()

Google 的论文: [MapReduce: Simplified Data Processing on Large Clusters](http://research.google.com/archive/mapreduce.html)

##### map

`map()`函数接受两个参数，第一个是一个函数，第二个是一个`Iterable`

作用是：将传入的函数作用到序列的每一个元素，并且返回新的`Iterable`对象。

```python
def f(x):
  return x * x;

r = map(f, [1, 2, 3])
list(r)
# [1, 4, 9]

list(map(str, [1, 2]))
# ['1', '2']
```

##### reduce













