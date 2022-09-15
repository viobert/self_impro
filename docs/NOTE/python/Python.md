# python

还是一样，随手记录学习中的一些细节。

## 字符编码

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