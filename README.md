### 前言

本笔记借鉴于以下网址：
> [鱼C工作室](https://fishc.com.cn)
>
> <https://docs.python.org/zh-cn/3/library/re.html>

--------------------------------------------------

### 概述

正则表达式(称为RE，或正则，或正则表达式模式)本质上是嵌入在Python中的一种微小的、高度专业化的编程语言，可通过re模块获得。使用正则表达式，你需要指定一些规则来描述那些你希望匹配的字符串集合；此集合可能包含英语句子、电子邮件地址、TeX命令，或你喜欢的任何内容。

正则表达式语言相对较小且受限制，因此并非所有可能的字符串处理任务都可以使用正则表达式完成。还有一些任务可以用正则表达式完成，但表达式变得非常复杂。在这些情况下，你最好编写Python代码来进行处理；虽然Python代码比精心设计的正则表达式慢，但它也可能更容易理解。

----------------------------------------------------------------------------

### 简单模式

我们首先要了解最简单的正则表达式。由于正则表达式用于对字符串进行操作，因此我们将从最常见的任务开始：匹配字符。

-----------------------------------------------------------------------------

##### 匹配字符

大多数字母和字符只会匹配自己。例如：正则表达式`test`将完全匹配字符串`test`。(你可以启用不区分大小写的模式，让这个正则匹配`Test`或`TEST`，这个后面会详细说。)
当然这条规则有例外；一些字符是特殊的，称为元字符(metacharacters)，它们并不能匹配自身，它们定义了字符类、子组匹配和模式重复次数等。会用很大的篇幅讨论各种元字符及其功能。
> 这是元字符的完整列表：
> . ^ $ * + ? { } [ ] \ | ( )

方括号`[]`，用于指定字符类，可以单独列出字符，也可以通过给出两个字符并用`'-'`标记将它们分开来表示一系列字符。`[abc]`将匹配任何字符`a`、`b`或`c`；这与`a-c`相同，它使用一个范围来表示同一组字符。如果你只想匹配小写字母，你的正则是`[a-z]`。
在方括号中的元字符不生效，例如，`[akm$]`将匹配`'a'`、`'k'`、`'m'`或`'$'`中的任意字符；`'$'`通常是一个元字符，但在一个字符类中它被剥夺了特殊性。
还可以通过`^`匹配设置的字符类中未列出的字符，在类的开头添加`^`，`[^5]`会匹配除了`'5'`之外的任何字符。如果插入符出现在字符类的其他位置，则它没有特殊含义。例如：`[5^]`将匹配`'5'`或`'^'`。

反斜杠`\`是最重要的元字符，反斜杠后面可以跟各种字符，以指示各种特殊序列。它也用于转移所有元字符，如果你需要匹配`[`或`\`，你可以在它们前面加一个反斜杠来移除它们的特殊含义：`\[`或`\\`。

注：一些以`'\'`开头的特殊序列表示通常有用的预定义字符集，例如数字集、字母集或任何非空格的集合。让我们举个例子：`\w`匹配任何字母数字字符。如果正则表达式模式以字节类表示，这相当于类`[a-zA-Z0-9]`。如果正则表达式是一个字符串，`\w`将匹配由unicodedata模块提供的Unicode数据库中标记为字母的所有字符。通过在编译正则表达式时提供re.ASCII标志，可以在字符串模式中使用更为受限制的`\w`定义。

以下为一些特殊序列的表格：
| 特殊字符 | 含义 |
| :---: | :---- |
| `\d` | 匹配任何十进制数字；等价于类`[0-9]` |
| `\D` | 匹配任何非十进制数字字符；等价于类`[^0-9]` |
| `\s` | 匹配任何空白字符；等价于类`[\t\n\r\f\v]` |
| `\S` | 匹配任何非空白字符；等价于类`[^\t\n\r\f\v]` |
| `\w` | 匹配任何字母与数字字符；相当于类`[a-zA-Z0-9]` |
| `\W` | 匹配任何非字母与数字字符；相当于类`[^a-zA-Z0-9]` |

这些序列可以包含在字符类中。例如，`[\s,.]`是一个匹配任何空格字符的字符类或者`','`，或`'.'`。

`.`匹配除换行符之外的任何内容，并且有一个可选模式(re.DOTALL)甚至可以匹配换行符。`.`常用于你想匹配“任何字符”的地方。

重复
能够匹配不同的字符集合是正则表达式可以做的第一件事，这对于字符串可用方法来说是不可能的。但是，如果这是正则表达式的唯一额外功能，那么它们就不会有太大的优势。另一个功能是你可以指定正则的某些部分必须重复一定次数。

重复中我们要了解的第一个元字符是`*`。`*`与字面字符`'*'`不匹配；相反，它指定前一个字符可以匹配零次或多次，而不是恰好一次。
例如，`ca*t`将匹配`'ct'`(0个`'a'`字符)，`'cat'`(1个`'a'`)，`'caaat'`(3个`'a'`字符)，等等。

类似`*`这样的重复是贪婪的；当重复正则时，匹配引擎将尝试尽可能多地重复它。如果模式地后续部分不匹配，则匹配引擎将回退并以较少的重复次数再次尝试。

通过一个逐步的例子来讲解“贪婪”，让我们考虑`a[bcd]*b`。这个正则匹配字母`'a'`，类`'[bcd]'`中的零或多个字母，最后以`'b'`结尾。现在想象一下这个正则与字符串`'abcbd'`匹配。
| 步骤 | 匹配 | 说明 |
| :---- | :---- | :---- |
| 1 | `a` | 正则中的`a`匹配。 |
| 2 | `abcbd` | 引擎尽可能多地匹配`[bcd]*`，直到字符串结束。 |
| 3 | 失败 | 引擎尝试匹配`b`，但是当前位置位于字符串结束，所以匹配失败。 |
| 4 | `abcb` | 回退一次，`[bcd]*`少匹配一个字符。 |
| 5 | 失败 | 再次尝试匹配`b`，但是当前位置是最后一个字符`'d'`。 |
| 6 | `abc` | 再次回退，所以`[bcd]*`只匹配`bc`。 |
| 7 | `abcb` | 再试一次`b`。这次当前位置的字符是`'b'`，所以它成功了。 |

正则现在已经结束了，它已经匹配了`abcb`。这演示了匹配引擎最初如何进行，如果没有找到匹配，它将逐步回退并一次又一次地重试正则的其余部分。它将回退，知道它为`[bcd]*`尝试零匹配，如果随后失败，引擎将断定该字符串与正则完全不匹配。

另一个重复的元字符是`+`，它匹配一次或多次。 要特别注意`*` 和`+` 之间的区别；`*`匹配 零次 或更多次，因此重复的任何东西都可能根本不存在，而`+`至少需要一次。 使用类似的例子，`ca+t`将匹配`'cat'`(1 个`'a'`)，`'caaat'`(3 个`'a'`)，但不会匹配`'ct'`。

还有两个重复限定符。 问号字符`?`匹配一次或零次；你可以把它想象成是可选的。 例如，`home-?brew`匹配`'homebrew'`或`'home-brew'`。

最复杂的重复限定符是`{m,n}`，其中*m*和*n*是十进制整数。 这个限定符意味着必须至少重复*m*次，最多重复*n*次。 例如，`a/{1,3}b`将匹配`'a/b'`，`'a//b'`和`'a///b'`。 它不匹配没有斜线的`'ab'`，或者有四个的`'a////b'`。

你可以省略*m*或*n*; 在这种情况下，将假定缺失值的合理值。 省略*m*被解释为 0 下限，而省略*n*则为无穷大的上限。

还原论者的读者可能会注意到其他三个限定符都可以用这种表示法表达。`{0,}`与`*`相同，`{1,}`相当于`+`，`{0,1}`和`?`相同。 最好使用`*`，`+`或`?`，因为它们更短更容易阅读。

-------------------------------------------------------

### 使用正则表达式

现在我们开始写一下简单的正则表达式。re模块提供了正则表达式引擎的接口，允许你将正则编译为对象，然后用它们进行匹配。
> re模块是使用C语言编写，所以效率要比用普通的字符串方法高很多；将正则表达式编译(compile)也是为了进一步提高效率；后面会经常提到“模式”，指的是正则表达式被编译成的模式对象。

###### 编译正则表达式

正则表达式被编译为模式对象，模式对象具有各种操作的方法，例如搜索模式匹配或执行字符串替换。
```python
>>> import re
>>> p = re.compile('ab*')
>>> p
re.compile('ab*')
```

re.compile()也接受一个可选的***flags***，用于开启各种特殊功能和语法变化，我们会在后面一一介绍。
现在先看一个例子。
```python
>>> p = re.compile('ab*', re.IGNORECASE)
```

正则作为字符串传递给re.compile()。正则被处理为字符串，因为正则表达式并不是Python的核心部分，并没有创建用于表达它们的特殊语法。(有些应用程序根本不需要正则，因此不需要通过包含它们来扩展语言规范。)相反，re模块只是Python附带的C扩展模块，就类似于socket或zlib模块。

将正则放在字符串中可以使Python语言更简单，但也有一些负面影响，下面我们就来谈一谈。

###### 麻烦的反斜杠

如前所述，正则表达式使用反斜杠字符(`'\'`)来使得一些普通的字符拥有特殊的能力(例如`\d`表示匹配任何十进制数字)，或剥夺一些特殊字符的能力(例如`\[`表示匹配左方括号`'['`)。这会跟Python字符串中实现相同功能的字符发生冲突。

假设你想要在LaTex文件中使用正则表达式匹配字符串`'\section'`。因为反斜杠作为需要匹配的特殊字符，所以你需要在它前面加多一个反斜杠来剥夺它的特殊功能。所以我们会把正则表达式的字符写成`'\\section'`。

但不要忘了，Python在字符串中同样使用反斜杠来表示特殊意义。因此，如果我们想将`'\\section'`完整地传给re.compile()，我们需要再次添加两个反斜杠……

| 字符       | 匹配阶段       |
| ---------- | -------------- |
| `\section` | 被匹配地字符串 |
| `\\section` | 为re.compile()转义的反斜杠 |
|`'\\\\section'`| 为字符串字面转义的反斜杠 |

简而言之，要匹配文字反斜杠，必须将`'\\\\'`写为正则字符串，因为正则表达式必须是`\\`，并且每个反斜杠必须表示为`\\`在常规Python字符串字面中。在反复使用反斜杠的正则中，这会导致大量重复的反斜杠，并使得生成的字符串难以理解。

解决方案是使用Python的原始字符串表示法来表示正则表达式；反斜杠不以任何特殊的方式处理前缀为`'r'`的字符串字面，因此`r'\n'`是一个包含`'\'`和`'n'`的双字符字符串，而`'\n'`是一个包含换行符的单字符字符串。正则表达式通常使用这种原始字符串表示法用Python代码编写。

| 常规字符串 | 原始字符串 |
| :---- | :---- |
| `"ab*"` | `r"ab*"` |
| `"\\\\section"` | `r"\\section"` |
| `"\\w+\\s+\\l"` | `r"\w+\s+\l"` |

###### 应用匹配

一旦你有一个表示编译正则表达式的队形，你用它做什么？模式对象有几种方法和属性。下面列举最重要几个：

| 方法/属性 | 目的                             |
| --------- | -------------------------------- |
| `match()` | 确定正则是否从字符串的开头匹配。 |
| `search()` | 扫描字符串，查找此正则匹配的任何位置。 |
| `findall()` | 找到正则匹配的所有子字符串，并将它们作为列表返回。 |
| `finditer()` | 找到正则匹配的所有子字符串，并将它们返回为一个iterator |

如果没有找到匹配，`match()`和`search()`返回 `None` 。如果它们成功， 一个*匹配对象*实例将被返回，包含匹配相关的信息：起始和终结位置、匹配的子串以及其它。

示例：
```python
>>> import re
>>> p = re.compile('[a-z]+')
>>> p
re.compile('[a-z]+')
```
现在，你可以尝试使用正则表达式`[a-z]+`去匹配各种字符串。

例如：
```python
>>> p.match("")
>>> print(p.match(""))
None
```
因为`+`表示匹配一次或者多次，所以空字符串不能被匹配。因此，match()返回None。

现在，让我们尝试一下可以匹配的字符串：
```python
>>> m = p.match("tempo")
>>> m
<re.Match object; span(0, 5), match = 'tempo'>
```

match()返回一个匹配对象，我们可以把结果储存到一个变量中以供稍后使用。

现在你可以检查匹配对象以获取有关匹配字符串的信息。匹配对象包含了很多方法和属性；最重要的是：

| 方法/属性 | 目的                 |
| --------- | -------------------- |
| `group()` | 返回正则匹配的字符串 |
| `start()` | 返回匹配的开始位置 |
| `end()` | 返回匹配的结束位置 |
| `span()` | 返回包含匹配(start,end)位置的元组 |

举例：
```python
>>> m.group()
'tempo'
>>> m.start(), m.end()
(0, 5)
>>> m.span()
(0, 5)
```

由于match()方法只检查正则是否在字符串的开头匹配，所以start()始终为零。但是，模式的search()方法会扫描字符串，因此在这种情况下匹配可能不会从零开始。：
```python
>>> print(p.match('::: message'))
None
>>> m = p.search('::: message'); print(m)
<re.Match object; span=(4, 11), match='message'>
>>> m.group()
'message'
>>> m.span()
(4, 11)
```

在实际应用中，最常见的方式是在变量中存储匹配变量，然后检查它是否为None。通常形式如下：
```python
p = re.compile(...)
m = p.match('string goes here')
if m:
    print('Match found:', m.group())
else:
    print('No match')
```

有两种模式方法返回所有的匹配结果，一个是findall()，另一个是finditer()。

findall()返回匹配字符串的列表：
```python
>>> p = re.compile(r'\d+')
>>> p.findall('1个馒头，2个馒头，3个馒头')
['1', '2', '3']
```

finditer()则是将匹配对象作为一个迭代器(iterator)返回：
```python
>>> iterator = p.finditer('1个馒头，2个馒头，3个馒头')
>>> iterator
<callable_iterator object at 0x...>
>>> for match in iterator:
        print(match.span())
        
        
(0, 1)
(5, 6)
(10, 11)
```

-----------------------------------------