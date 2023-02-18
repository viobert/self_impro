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
