---
layout:     post
title:      "python字符编码和字符串"
subtitle:   "编码与解码、字符与字节浅析"
date:       2017-09-12 17:21:00
author:     "Bryancy"
header-img: "img/post-bg-ios9-web.jpg"
tags:
    - python
---

## 前言

背景：我司代码都是基于python2

今天接到一个需求，三方登录时，对第三方的昵称如果大于20个字符，进行过滤截取，毫不迟疑，写了以下代码：
```python
if len(nickname) > 20:
    # 截取
```

我转念一想，事情不大对，我这算的是字符数（这个表述是正确的，是字符）啊
```python
len("abcd")  # 等于4
len("I love python")  # 等于13
```

上面的例子显而易见，小学生水平，再看下面的
```python
len("我爱北京天安门")  # 等于21
len(u"我爱北京天安门")  # 等于7
```

其实我想要的是下面这个，但是我在代码里直接那么写，我操作的字符串前面到底有没有u呢，这个u我知道是unicode字符串，它和没有u的字符串到底区别在哪里？

几年前我曾经下决心弄懂这个问题，可能当时也确实明白了，但是现在差不多已经全忘了。我自己总结了一下，一是之前一直是用python3，python3做了一些修改。二是这个问题确实不应该交给程序员来解决（我承认我的记性确实不好）。

然而，旧的系统那么多，难保你下次跳槽的公司没有一些上了年纪的代码，我们还是来梳理一下吧。


## 字符与字节

一个字符不等价于一个字节，字符是人类能够识别的符号，而这些符号要保存到计算机的存储中就需要用计算机能够识别的字节来表示。一个字符往往有多种表示方法，不同的表示方法会使用不同的字节数。这里所说的不同的表示方法就是指字符编码，比如字母A-Z都可以用`ASCII`码表示（占用一个字节），也可以用`UNICODE`表示（占两个字节），还可以用`UTF-8`表示（占用一个字节）。字符编码的作用就是将人类可识别的字符转换为机器可识别的字节码，以及反向过程。

`UNICDOE`才是真正的字符串，而用ASCII、UTF-8、GBK等字符编码表示的是字节串。关于这点，我们可以在Python的官方文档中经常可以看到这样的描述
> “Unicode string” , “translating a Unicode string into a sequence of bytes”。
> <br/>（注意：英文中string是我们说的字符串，bytes是字节串）

我们写`代码`是写在`文件`中的，而`字符`是以`字节`形式保存在`文件`中的，因此当我们在`文件`中定义个`字符串`时被当做`字节串`也是可以理解的。但是，我们需要的是`字符串`，而不是`字节串`（我们写代码处理的是我们能想象的数据，也就是字符串，应该不会有人想象字节串吧）。一个优秀的编程语言，应该严格区分两者的关系并提供巧妙的完美的支持。JAVA语言就很好，我认识的JAVA程序员从来没有考虑过这些不应该由程序员来处理的问题（我一直这么认为）。遗憾的是，很多编程语言试图混淆`“字符串”`和`“字节串”`，他们把字节串当做字符串来使用，PHP和Python2都属于这种编程语言。最能说明这个问题的操作就是取一个包含中文字符的字符串的长度：

- 对`字符串`取长度，结果应该是所有`字符`的个数，无论中文还是英文
- 对`字符串`对应的`字节串`取长度，就跟编码(encode)过程使用的字符编码有关了(比如：UTF-8编码，一个中文字符需要用3个字节来表示；GBK编码，一个中文字符需要2个`字节`来表示)

```ipython
In [8]: a = u"中国"
In [9]: print len(a), type(a)
2 <type 'unicode'>

In [10]: b = a.encode('utf-8')
In [11]: print len(b), type(b)
6 <type 'str'>

In [12]: c = a.encode('gbk')
In [13]: print len(c), type(c)
4 <type 'str'>
```

## 编码与解码

UNICODE字符编码，也是一张字符与数字的映射，但是这里的数字被称为代码点(code point), 实际上就是十六进制的数字。

Python官方文档中对Unicode字符串、字节串与编码之间的关系有这样一段描述：
> a Unicode string is a sequence of code points, which are numbers from 0 to 0x10ffff.<br/>
> This sequence needs to be represented as a set of bytes (meaning, values from 0–255) in memory.<br/>
> The rules for translating a Unicode string into a sequence of bytes are called an encoding.

> Unicode字符串是一个代码点（code point）序列，代码点取值范围为0到0x10FFFF（对应的十进制为1114111。<br/>
> 这个代码点序列在存储（包括内存和物理磁盘）中需要被表示为一组字节(0到255之间的值)，<br/>
> 而将Unicode字符串转换为字节序列的规则称为编码。

这里说的编码不是指字符编码，而是指:**编码的过程以及这个过程中所使用到的Unicode字符的代码点与字节的映射规则**。
这个映射不必是简单的一对一映射，因此编码过程也不必处理每个可能的Unicode字符，例如：

将`Unicode`字符串转换为`ASCII`编码的规则很简单–对于每个代码点：
- 如果代码点数值<128，则每个字节与代码点的值相同
- 如果代码点数值>=128，则`Unicode`字符无法在此编码中进行表示（这种情况下，Python会引发一个`UnicodeEncodeError`异常）

将`Unicode`字符串转换为`UTF-8`编码使用以下规则：
- 如果代码点数值<128，则由相应的字节值表示（与Unicode转ASCII字节一样）
- 如果代码点数值>=128，则将其转换为一个2个字节，3个字节或4个字节的序列，该序列中的每个字节都在128到255之间。

简单总结：
- 编码(encode)：将`Unicode`字符串（中的代码点)转换特定字符编码对应的字节串的过程和规则
- 解码(decode)：将特定字符编码的字节串转换为对应的`Unicode`字符串(中的代码点)的过程和规则

可见，无论是编码还是解码，都需要一个重要因素，就是特定的字符编码。因为一个字符用不同的字符编码进行编码后的字节值以及字节个数大部分情况下是不同的，反之亦然。

而且很容易看到，编码和解码都是基于Unicode字符串，没有第二种字符串掺和，其他都是编码方式，一定要深刻地认识到这一点


## Python编码
#### Python源代码文件的执行过程
我们都知道，磁盘上的文件都是以二进制格式存放的，其中文本文件都是以某种特定编码的字节形式存放的。对于程序源代码文件的字符编码是由编辑器指定的，比如我们使用vim来编写Python程序时会指定文件编码为UTF-8，那么Python代码被保存到磁盘时就会被转换为UTF-8编码对应的字节（encode过程）后写入磁盘。当执行Python代码文件中的代码时，Python解释器在读取Python代码文件中的字节串之后，需要将其转换为UNICODE字符串（decode过程）之后才执行后续操作。

上面已经解释过，这个转换过程（decode，解码）需要我们指定文件中保存的字节使用的字符编码是什么，才能知道这些字节在UNICODE这张万国码和统一码中找到其对应的代码点是什么。这里指定字符编码的方式大家都很熟悉，一图胜千言：
```python
# -*- coding:utf-8 -*-
```
![](/img/in-post/post-encoding-20170919/python_source_file_execute.png)

#### 默认编码
那么，如果我们没有在代码文件开始的部分指定字符编码，Python解释器会使用哪种字符编码把从代码文件中读取到的字节转换为UNICODE代码点呢？就像我们配置某些软件时，有很多默认选项一样，Python解释器内部设置了默认的字符编码。因此大家所说的Python中文字符问题就可以总结为一句话：当无法通过默认的字符编码对字节进行转换时，就会出现解码错误(UnicodeEncodeError)。

Python2和Python3的解释器使用的默认编码是不一样的，我们可以通过sys.getdefaultencoding()来获取默认编码：
```python
# python2
In [1]: import sys
In [2]: sys.getdefaultencoding()
Out[3]: 'ascii'
# python3
In [1]: import sys
In [2]: sys.getdefaultencoding()
Out[3]: 'utf-8'
```

因此，对于Python2来讲，Python解释器在读取到中文字符的字节码尝试解码操作时，会先查看当前代码文件头部是否有指明当前代码文件中保存的字节码对应的字符编码是什么。如果没有指定则使用默认字符编码”ASCII”进行解码导致解码失败，导致如下错误：
```
SyntaxError: Non-ASCII character '\xc4' in file xxx.py on line 11, but no encoding declared; see http://python.org/dev/peps/pep-0263/ for details
```

对于Python3来讲，执行过程是一样的，只是Python3的解释器以”UTF-8”作为默认编码，但是这并不表示可以完全兼容中文问题。比如我们在Windows上进行开发时，Python工程及代码文件都使用的是默认的GBK编码，也就是说Python代码文件是被转换成GBK格式的字节码保存到磁盘中的。Python3的解释器执行该代码文件时，试图用UTF-8进行解码操作时，同样会解码失败，导致如下错误：
```
yntaxError: Non-UTF-8 code starting with '\xc4' in file xxx.py on line 11, but no encoding declared; see http://python.org/dev/peps/pep-0263/ for details
```

#### 再谈python2和python3中的字符串
其实Python3中对字符串支持的改进，不仅仅是更改了默认编码，而是重新进行了字符串的实现，而且它已经实现了对UNICODE的内置支持，从这方面来讲Python已经和JAVA等语言一样优秀。下面我们来看下Python2与Python3中对字符串的支持有什么区别：

##### python
Python2中对字符串的支持由以下三个类提供
```python
class basestring(object)
class str(basestring)
class unicode(basestring)
```
执行`help(str)`和`help(bytes)`会发现结果都是`str`类的定义，这也说明Python2中`str`就是字节串，而后来的`unicode`对象对应才是真正的字符串。
```ipython
In [1]: a = "中国"
In [2]: b = u"中国"
In [3]: print type(a), len(a)
<type 'str'> 6
In [4]: print type(b), len(b)
<type 'unicode'> 2
```

##### python3
Python3中对字符串的支持进行了实现类层次的上简化，去掉了`unicode`类，添加了一个`bytes`类。从表面上来看，可以认为Python3中的`str`和`unicode`合二为一了。
```python3
class bytes(object)
class str(object)
```
实际上，Python3中已经意识到之前的错误，开始明确的区分字符串与字节。因此Python3中的str已经是真正的字符串，而字节是用单独的bytes类来表示。也就是说，Python3默认定义的就是字符串，实现了对UNICODE的内置支持，减轻了程序员对字符串处理的负担。

```ipython
In [1]: a = "中国"
In [2]: b = u"中国"
In [3]: c = a.encode("gbk")
In [4]: print(type(a), len(a))
<class 'str'> 2
In [5]: print(type(b), len(b))
<class 'str'> 2
In [6]: print(type(c), len(c))
<class 'bytes'> 4
```

#### 字符编码转换
上面提到，UNICODE字符串可以与任意字符编码的字节进行相互转换，如图：
![](/img/in-post/post-encoding-20170919/unicode_string.png)
那么大家很容易想到一个问题，就是不同的字符编码的字节可以通过Unicode相互转换吗？答案是肯定的。

##### Python2中的字符串进行字符编码转换过程是：
字节串–>decode(‘原来的字符编码’)–>Unicode字符串–>encode(‘新的字符编码’)–>字节串
```ipython
In [1]: a = "中国"  # utf-8编码的字节串
In [2]: b = a.decode('utf-8').encode('gbk')  # 先转换成unicode字符串，再进行gbk编码
In [3]: print b.decode('gbk')  # 解码成unicode字符串
中国
```

##### Python3中定义的字符串默认就是unicode，因此不需要先解码，可以直接编码成新的字符编码：
字符串–>encode(‘新的字符编码’)–>字节串
```ipython
In [1]: a = "中国"
In [2]: b = a.encode("gbk")
In [3]: print(b.decode("gbk"))
中国
```


## 结语
python字符串编码差不多就是这些内容了，搞清楚原理其实记忆也不难。祝大家不会再掉到python字符编码的坑中，祝我司能尽快迁移到python3！

