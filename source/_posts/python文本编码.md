---
title: Python 文本编码
tags: 编程语言 编码
---

<!-- more -->

> 转载请注明出处：
> [Python文本编码](https://zhuyuhe.github.io/2018/05/13/python%E6%96%87%E6%9C%AC%E7%BC%96%E7%A0%81/)

相信大家都碰到过令人头疼的python编码问题，比如：`'ascii' codec can't decode byte` `UnicodeDecodeError: 'utf-8' codec can't decode byte 0xe9 in position 2892: invalid continuation byte` 等很多类似的问题。由于博主平时使用python处理自然语言比较多，因此少不了与字符串及其编码打交道。每次碰到这种问题，都要去google个半天，然后按网上的解决方案一个一个试，直到解决为止。

终于有一天，我再也无法忍受这种重复的操作，下定决心一定要把这些问题背后的原因和解决方法搞清楚，于是就有了这片文章。力求搞懂python编码的原理，并总结了常见的问题和解决方案，方便以后查阅。

### 1. 编码基础
#### 1.1 什么是编码和解码
首先，我们需要明白，计算机只能处理二进制数字，如果要处理文本，需要把文本转换为二进制数字，然后才能进行处理。

同样，对于文本，计算机存储的是二进制数字，要想得到文本，需要将二进制数字转化为文本，才能呈现出我们看到的内容。

也就是说，我们看到的是文本，计算机处理的是二进制数字。这中间肯定是需要来回转换的。从文本到二进制数字就是编码，从二进制数字到文本就是解码。

#### 1.2 以 ASCII 编码为例
以我们最熟悉的 ASCII 编码为例，大写字母 `A` 的编码是 `65`，小写字母 `a` 的编码是 `97`。
```python
>>> list('A'.encode('ascii'))
[65]
>>> list('a'.encode('ascii'))
[97]
```
上面的代码意思是将 `A` 和 `a` 以 ASCII 编码方式编码后，得到的数字，这里以十进制表示之，换成二进制就是
>'A' -> 01000001
>'a' -> 01100001

就是说，其实我们看到的上述两个字母，在计算机中，是以上面的二进制形式存储的。当我们需要使用或将字母展示出来的时候，计算机需要将二进制数字转化为字母，这个过程就是解码。

ASCII 编码是计算机发展早期发明的，当时只将 127 个字符进行了编码（字符到二进制数字的映射），也就是大小写英文字母，数字和一些符号。ASCII 码采用 8 位 bit 进行编码，也就是一个字节，能表示的最大整数就是 255。当处理中文时，255个字符，显然就不够了。

#### 1.3 中文编码
要处理中文，一个字节显然是不够的（中文字符数量可远远不止 255 个）。所以，中国制定了 `GB2312` 编码，使用两个字节来编码中文。两个字节能够表示 65535 个整数，基本上能够囊括所有的汉字了。让我们来看一下：
```python
>>> list('中'.encode('gb2312'))
[214,208]
>>> list('a'.encode('gb2312'))
[97]
```
可以看到，`GB2312` 编码方式不仅可以将中文进行编码，同时兼容 `ASCII` 码，使用两者对 `ASCII` 的 127 个字符进行编码得到的结果是相同的。即
> '中' -> 11010110 11010000
> 'a' -> 00000000 01100001

我们常见的 `gbk` 编码方式，其实就是在 `gb2312` 编码的基础上，增加了一些中文字符。

现在，使用 `GB2312` 编码方式，我们能处理中英文混杂的文本了。那么问题来了，如果一段文本既有中文，又有英文，还有其他语言呢？

#### 1.4 大统一：unicode 编码
为每种语言制定一套编码方式实在是太蠢了！为什么不能把所有语言的所有字符一起编码呢？

把所有语言统一到一套编码里，这套编码就是 unicode 编码。使用 unicode 编码，无论处理什么文本都不会出现乱码问题了。

unicode 编码使用两个字节（16 位 bit）表示一个字符，比较偏僻的字符需要使用 4 个字节。

但是新的问题又来了，如果一段纯英文文本，用 `unicode` 编码存储会比用 `ASCII` 编码多占用一倍空间！无论是存储还是传输都很浪费！

#### 1.5 节约小能手：utf-8 编码
为了改进上述问题，又提出了 `utf-8` 编码。该编码将一个 unicode 字符编码成 1~6 个字节，常用的英文字母被编码成 1 个字节，汉字通常是 3 个字节，只有很生僻的字符才会被编码成 4~6 个字节。注意，从 unicode 到 utf-8 并不是直接的对应，而是通过一些算法和规则来转换的。 

来看一下具体编码例子吧：
```python
>>> list('中'.encode('utf-8'))
[228, 184, 173]
>>> list('a'.encode('utf-8'))
[97]
```
可以看出，`utf-8` 将汉字 ‘中’ 编码成了三个字节，将英文字母 ‘a’ 编码成了一个字节，且 `utf-8` 编码兼容 `ASCII` 编码。

### 2. Python编码
#### 2.1 Python2 or Python3 ?
从上面的知识，我们可以知道，字符的最佳定义应当是 Unicode 字符（存储及传输的时候再转为 utf-8）。

从 Python3 的 str 对象中获取的元素是 Unicode 字符，这相当于从 Python2 的 unicode 对象中获取的元素。而 Python2 的 str 对象获取的是原始字节序列（相信用过 Python2 的都见过 ‘\xe8\x32\xa6\xb2......’ 这种乱七八糟的字符吧）。

所以，我们的结论是：
> 人生苦短，我用Python3

#### 2.2 一个例子
让我们来结合Python实例来具体看一下编码的应用：
```python
# Python3 字符串，为 Unicode 字符
>>> a = '中文'
>>> import sys
# 查看当前系统默认编码方式（linux）， windows 默认为utf-8
>>> sys.getfilesystemencoding()
'utf-8'
>>> with open('test.txt', 'w', encoding = 'utf-8') as f:
···     f.write(a)
···
>>> with open('/home/zhuyuhe/test.txt', 'r'，encoding = 'utf-8') as f:
...     print(f.readlines())
...
['中文']
```
上面两个 open 发生了什么的，让我们看一下：

对于第一个 open 函数，它的执行流程是这样的：
```graphLR
[str对象获取 unicode 字符] -->|utf-8 编码| --> [字节序列] -->|二进制形式存储| --> [test.txt]
```

对于第二个 open 函数，它的执行流程是这样的：
```graphLR
[文件中存放的0101...] -->|utf-8 解码| --> [unicode 字符] --> [str 对象]
```

这里可能会有一点疑问，为什么 unicode 字符可以通过 utf-8 编码成二进制数字，二进制数字通过 utf-8 解码成 unicode 字符。

前面已经说过，utf-8 是在 unicode 的基础上改进而来，是针对传输和存储而设计的一种编码方式。 utf-8 编码针对的对象是 unicode 字符。

也就是说，在计算机内存中，统一使用 unicode 编码，当需要保存到硬盘或者需要传输的时候，就转换为 utf-8 编码。

#### 2.3 另一个例子
如果我们使用记事本打开上一小节保存的 `test.txt` 呢，这中间发生了什么呢？

让我们同样用流程图来看一下：

```
[在计算机硬盘以0101...形式存放的文件] --> |记事本所使用的编码方式|--> B[我们看到的字符]
```

我们的 `test.txt` 是以 utf-8 的编码方式保存的，如果记事本使用的编码方式为 `gb2312`，打开该文件时，记事本会尝试以 `gb2312` 的方式解码该文件的字节序列（二进制数字），由于两者的字符与字节对应关系不同，当然会解码失败。此时记事本会显示各种乱码。这个过程类似下面的代码：

```python
>>> '中文'.encode('utf-8').decode('gbk')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'gbk' codec can't decode byte 0xad in position 2: illegal multibyte sequence
```

如果碰到上述问题，可以尝试使用 [notepad++](https://notepad-plus-plus.org/) 打开该文件，尝试改变编辑器的编码方式，来尝试是否能正常打开文件。

是的，只能去尝试。因为在没有任何信息的情况下，给我们一串字节序列，我们是不知道它的编码方式的。

所以，在平时的文件存储和传输中，统一编码方式是很重要的。

#### 2.4 统一编码
在日常的使用中，避免乱码的重要方式就是统一你的编码方式。

一般我们将编码方式统一为 utf-8，下面给出了一些参考的建议：

 1. linux 使用 Python 时，可以使用如下代码查看默认编码：
 ```python
 >>> import sys
 >>> sys.getfilesystemencoding()
 
 ```
 查看是否为 utf-8，如果不是，改成 utf-8。
 2. windows 默认编码方式为 utf-8，不需要修改
 3. 在使用 open 函数及其他读写函数时，加上 `encoding=utf-8`，确保文件以 utf-8 方式编码和解码。
 4. 使用远程连接软件时，比如 XShell 或 MobaXterm ，将软件的编码选为 utf-8，这样可以保证远程连接显示正常。
 5. 不要依赖系统的默认编码，打开文件时应始终明确传入 encoding = 参数，因为不同设备使用的默认编码不同，即使是同一设备，也可能会发生变化。
 
其实，理解了为何需要做上述处理，也基本就理解了编码。碰到编码问题，结合错误信息，基本上就能很快找出发生错误的原因了。不过为了方便起见，我还是将常见的问题总结了下。

### 3. 常见问题原因及解决方案
#### 3.1 UnicodeEncodeError
多数非 UTF 编码器只能处理 Unicode 字符的一小部分子集。把文本转换为字节序列时，如果目标编码中没有定义某个字符，就会抛出 UnicodeEncodeError 异常。

看个例子：
```python
>>> '中文'.encode('ascii')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)

```

当我们尝试将中文字符以 ASCII 方式编码时，报出了 UnicodeEncodeError。这个原因很明显，ASCII 编码方式并没有定义中文字符，无法对中文字符进行编码。

碰到这种问题，可以使用 errors 参数进行处理：
```python
>>> '中文'.encode('ascii', errors='ignore')
b''
```
我们将错误的编码忽略掉，最后得到空的字节序列。通常，这样做是非常不妥的。

errors 参数还有许多可选项，大家可以自行探索。

比较妥当的解决方案，当然是选择合适的编码方式。这个需要根据具体情况而定，相信，理解了这个错误发生的原因，解决起来就很轻松啦！

#### 3.2 UnicodeDecodeError
不是每一个字节都包含有有效的 ASCII 字符，也不是每一个字符序列都是有效的 utf-8 。因此把二进制序列转换为文本时，遇到无法转换的字节序列就会抛出 UnicodeDecodeError。

看个例子：
```python
>>> b = b'\xe4\xb8\xad\xe6\x96\x87'
>>> b.decode('ascii')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 0: ordinal not in range(128)
```

b 是字符 '中文' 以 utf-8 方式编码形成的字节序列，属于 Python3 中的 bytes 类型。将该字节序列以 ASCII 方式解码时抛出了 UnicodeDecodeError。


### 4 总结
本文详细阐述了编码的原理及各种问题发生的原因，相信搞懂这些，以后碰到编码问题就再也不用求助 Google 了。知道了问题发生的原因，我们自己就能迅速解决了！

### 参考文献

[廖雪峰的官方网站-字符串和编码](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001431664106267f12e9bef7ee14cf6a8776a479bdec9b9000)
[大道至简：史上最易懂的『乱码』解决方案](http://www.10tiao.com/html/506/201803/2651615541/1.html)
《流畅的Python》



