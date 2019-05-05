---
layout:     post
title:      "Effective Python中文版"
date:       2016-05-04 19:00:00 +0800
author:     "Jay Cai"
header-style: text
catalog: true
tags:
    - Python
---

> 摘录自**Effective Python中文版**一书，详细请购买原本书籍。

##  第一章 用Pythonic方式来思考
###  第1条： 确认自己所用Python的版本
在命令行中输入默认的`python`命令来究竟是会执行哪个版本呢？可以`--version`标志来运行`python`命令，以了解所使用的python版本。
```python
$ python --version
Python 3.5.1
```
运行程序的时候，也可以在内置的`sys`模块里查询相关的值，以确定当前使用的python版本。
```python
>>> import sys
>>> print(sys.version_info)
sys.version_info(major=3, minor=5, micro=1, releaselevel='final', serial=0)
>>> print(sys.version)
3.5.1 (default, Dec 6 2015, 01:38:48) [MSC v.1900 64 bit (AMD64)]
>>>
```
*   **因为系统环境可能有多个python版本，确保所用python版本是自己想要使用的版本**

###  第2条： 遵循PEP8风格指南
PEP8风格详细指南参考[Python Enhancement Proposal #8](http://www.python.org/dev/peps/pep-0008)
#### 要点
* **空白**: Python中的空白(*whitespace*)会影响代码的含义以及代码清晰度，尤其需要注意
    * 使用*space*(空格)来表示缩进，而不要用*tab*(制表符)
    * 和语法相关的每一层缩进都用四个*space*(空格)来表示
    * 每行的字符数不要超过79
    * 对于需要占用多行的长表达式，首行之外的其余各行都应该在通常的缩进级别上再加4个空格
    * 文件中的函数与类之间应该用两个空行隔开
    * 同一个类中，各方法之间应该用一个空行隔开
    * 在使用下标来获取列表元素，调用函数或给关键字参数赋值的时候，不要在两旁加空格
    * 为变量赋值时，赋值符号`=`的左右两侧应该各自写上一个空格(*just one*)
* **命名**: PEP8提倡采用不同的命名风格来编写python代码中的各个部分，以便提高阅读性
    * 函数，变量及属性应该使用小写字母，单词之间以下划线连接，例如 `lowercase_underscore`
    * 受保护的实例属性，应该以单个下划线开头，例如 `_leading_underscore`
    * 私有的实例属性，应该以两个下划线开头，例如 `__double_leading_underscore`
    * 类与异常，应该以每个单词首字母均大写的形式来命名，例如 `CapitalizedWord`
    * 模块级别的常量，应该全部采用大写字母来命名，单词之间以下划线连接，例如 `ALL_CAPS`
    * 类中的实例方法(*instance method*)，应该把首个参数命名为`self`，以表示对象自身
    * 类方法(*class method)*的首个参数，应该命名为*cls*,以表示类自身
* **表达式和语句**: *\<The Zen of Python\>*(Python之禅)中说：“每件事都应该有直白的做法，而且最好只有一种。” PEP8体现了这种思想。
    * 采用内联形式的否定词，而不要把否定词放在整个表达式的前面，例如，应该写`if a is not b`而不是`if not a is b`
    * 不要通过检测长度的办法(如`if len(somelist) == 0`)来判断*somelist*是否为`[]`或`''`等空值，而是应该采用`if not somelist`这种写法来判断，他会假定：空值将自动评估为`False`
    * 检测*somelist*是否为`[1]`或者`'hi'`等非空值时，亦应如此，`if somelist`语句默认会把非空的值判断为`True`
    * 不要编写单行的`if`语句，`for`循环，`while`循环及`except`符合语句，而是应该把这些语句分成多行来书写，以示清晰
    * `import`语句应该重视放在文件开头
    * 引入模块的时候，重视应该使用绝对名称，而不应该根据当前模块的路径来使用相对名称。例如，引入`bar`包中的`foo`模块时，应该完整地写出`from bar import foo`，而不应简写为`import foo`
    * 如果一定要以相对名称来编写`import`语句，那就采用明确的写法：`from . import foo`
    * 文件中的那些`import`语句应该按顺序分成三个部分，分别表示标准库模块，第三方模块以及自定义模块。在每一部分中，各`import`语句应该按模块的字母顺序来排列

###  第3条： 了解bytes,str与unicode的区别
Python3有两种表示字符序列的类型：`bytes`和`str`。前者的实例包含原始的8位字节，后者的实例包含*Unicode*字符。
Python2也有两种表示字符序列的类型：`str`和`unicode`。与Python3不同的是，`str`的实例包含原始的8位字节，而`unicode`的实例则包含*Unicode*字符。

有许多方法来表示*Unicode*字符的二进制数据（原始的8位字节）。
最常用的编码方式为*UTF-8*编码。但是要记住，Python3中的`str`实例和Python2中的`unicode`都没有和特定的二进制编码相关联。
所以要想将*Unicode*字符转换成二进制数据，就必须使用`encode`方法，要想把二进制数据转换成*Unicode*字符，就必须使用`decode`方法。

编写Python程序的时候，一定要把编码和解码操作放在最外围来做。程序最核心的部分应使用Unicode字符类型（Python3中使用`str`,Python2中使用`unicode`）并且不要对字符编码做任何假设。
这种办法既可以令程序接收多种类型的文本编码（如*Latin-1,Shift JIS, Big5*），又可以保证输出的文本信息只采用一种编码形式（最好是使用*UTF-8*编码）。

由于字符类型有别，所以Python代码中经常会出现两种常见的使用情景。
*   开发者需要原始的8位字节值，这些8位字节值表示以*UTF-8*（或者其他的编码方式）编码的字符。
*   开发者需要操作没有特定编码的*Unicode*字符。 
所以我们需要两个辅助(`helper`)函数,以便在这两种情况之间转换，以此来确保转换后的输入数据符合开发者的预期。

在Python3中，我们需要编写如下两个处理编码转换的函数：
```python
# 接收str或者bytes, 并总是返回str
def to_str(bytes_or_str):
    if isinstance(bytes_or_str,bytes):
        value = bytes_or_str.encode('utf-8')
    else:
        value = bytes_or_str
    # str类型的数据
    return value

# 接收str或者bytes, 并总是返回bytes
def to_bytes(bytes_or_str):
    if isinstance(bytes_or_str,str):
        value = bytes_or_str.encode('utf-8')
    else:
        value = bytes_or_str
    # 字节类型的数据
    return value
```

在Python2中，我们需要像下面这样编写：
```python
# Python 2
# 接收str或者unicode，并总是返回unicode
def to_unicode(unicode_or_str):
    if isinstance(unicode_or_str,str):
        value = unicode_or_str.encode('utf-8')
    else:
        value = unicode_or_str
    # unicode类型的数据
    return value

# 接收str或者unicode，并总是返回str
def to_str(unicode_or_str):
    if isinstance(unicode_or_str,unicode):
        value = unicode_or_str.encode('utf-8')
    else:
        value = unicode_or_str
    # str类型的数据
    return value
```

在Python中使用原始的8位字节值与*Unicode*字符时，有两个问题要注意。

1.  第一个问题可能会出现在Python2中，如果`str`数据仅仅包含7位*ASCII码*字符，那么`unicode`和`str`实例似乎就成了同一种类型。
    *   可以使用`+`运算符把`str`和`unicode`连接起来。
    *   可以使用等价或者不等价运算符来比较`str`和`unicode`实例。
    *   在格式化字符串中，可以用`%s`等形式来代表`unicode`实例。
    这些行为意味着，在只处理7位*ASCII*的情景下，如果某函数接收`str`，那么可以给他传入`unicode`；如果某函数接受`unicode`，那么也可以给他传入`str`。
    **而在Python3中，`bytes`与`str`实例则绝对不会等价，即时是空字符串也不行。**所以，在传入字符序列的时候必须留意其类型。

2.  第二个问题可能会出现在Python3中，如果通过内置的`open`函数获取了文件句柄，那么要注意，该句柄默认会采用*UTF-8*编码格式来操作文件。
    而在Python2中，文件操作的默认编码格式格式这是二进制形式。这可能会导致程序出现奇怪的错误，对于习惯了Python2的程序员更是如此。
    <br>例如，现在要想文件中随机写入一些二进制数据。下面这种用法在Python2中可以正常运作，但在Python3中不行。
    ```python
    with open('/tmp/random.bin','w') as f:
        f.write(os.urandom(10))
    
    >>>
    TypeError: must be str,not bytes
    ```
    发生上述异常的原因在于，Python3给`open`函数添加了名为*encoding*的新参数，该参数的默认值是'UTF-8'。这样在文件句柄上进行`read`和`write`操作时，系统就要求开发和必须传入包含*Unicode*字符的`str`实例，而不接受包含二进制数据的`bytes`实例。
    <br>为了解决这个问题，我们必须用二进制写入模式'wb'来开启待操作的文件，而不能像原来那样，采用字符写入模式'w'。按照下面这种方式来使用`open`函数，即可同时适配Python2和Python3：
    ```python
    with open('/tmp/random.bin','wb') as f:
        f.write(os.urandom(10))
    ```
    从文件中读取数据的时候也有这种问题。解决办法与写入时相似：用'rb'模式打开文件，而不要用'r'模式。

#### 要点
*  Python3中，`bytes`是一种包含8位字节值的序列，`st`r是一种包含*Unicode*字符的序列。开发者不能以`>`或`+`等操作符来混同操作`bytes`和`str`实例
*  Python2中，`str`是一种包含8位字节值的里，`unicode`是一种包含*Unicode*字符的序列。如果`str`只含有7位*ASCII*字符，那么可以通过相关的操作符来同时使用`str`和`unicode`
*  在对输入数据进行操作之前，使用辅助函数来保证字符序列的类型与开发者的期望相符。
*  对于文件操作，读取或者写入二进制数据时，总应该使用'rb'或'wb'等二进制模式来开启文件

###  第4条： 用辅助函数来取代复杂的表达式
表达式如果比较复杂，就考虑将其拆解成小块，并把这些逻辑移入辅助函数中。这样会让代码更加易读。
<br>例如下面的表达式：
```python
# For Query string：'red=5&blue=0&green='
red = my_value.get('red', [''])[0] or 0
green = my_value.get('green', [''])[0] or 0
opacity = my_value.get('opacity', [''])[0] or 0
print('Red:%r' % red)
print('Green:%r' % green)
print('pacity:%r' % opacity)
>>>
Red: '5'
Green: 0
Opacity: 0
```
可以提炼出以下辅助函数来提高代码的阅读性：
```python
def get_first_int(values,key,default=0):
    found = values.get(key,[''])
    if found[0]:
        found = int(fount[0])
    else:
        found = default
    return found

green = get_first_int(my_values,'green')
```

#### 要点
*  开发者很容易过度运用Python的语法特性，从而写出发复杂并难以理解的单行表达式
*  请把复杂的表达式移入辅助函数之中，特别是反复使用的逻辑
*  使用`if/else`表达式，要比用`or`或者`and`这样的`Boolean`操作符写成的表达式更加清晰

###  第5条： 了解切割序列的办法
Python提供了方便的序列切片(*slice*)操作，使得开发者能够轻易访问由序列中的某些元素说构成的子集。
最简单的用法是对内置的`list`，`str`, `bytes`进行切割。切割操作还可以延伸到实现了`__getitem__`和`__setitem__`这两个特殊方法的Python类上。
切割操作的基本写法是`somelist[start:end]`，其中*start*表示起始索引，其所指元素会包含在切割结果内，*end*表示结束索引，其所指元素不包括在切割结果之中。
<br>示例如下：
```python
a[:]       # ['a','b','c','d','e','f','g','h']
a[:5]      # ['a','b','c','d','e']
a[:-1]     # ['a','b','c','d','e','f','g']
a[4:]      # ['e','f','g','h']
a[-3:]     # ['f'.'g','h']
a[2:5]     # ['c','d','e']
a[2:-1]    # ['c','d','e','f','g']
a[-3:-1]   # [ 'f','g']
```

**对原列表切割之后，会产生另外一份全新的列表**。系统仍然维护着指向原列表中各个对象的应用。在切割后得到的新列表上进行修改，不会影响原列表。
```python
a = ['a','b','c','d','e','f','g','h']
b = a[4:]
print("Before:", b)
b[1] = 99
print("After:",b)
print("Original:",a)
>>>
Before: ['e','f','g','h']
After: ['e',99,'g','h']
Original: ['a','b','c','d','e','f','g','h']
```

在赋值时对左侧列表使用切割操作时，会把该列表中处在指定范围内的对象替换为新值。与元组(*tuple*)的赋值`a,b = c[:2]`不同，此切片的长度无需新值的个数相等。位于切片范围之前及之后的值都保留不变。列表会根据新值的个数相应地扩展或者收缩。
```python
print("Before:",a)
a[2:7] = [99,22,14]
print("After:",a)
>>>
Before: ['a','b','c','d','e','f','g','h']
After: ['a','b',99,22,14,'h']
```

如果对赋值操作左侧的列表使用切片，而没有指定起止索引时，系统会把右侧的新值赋值一份，并用这份拷贝来替换左侧列表的全部内容，而不会重新分配新的列表。
```python
b = a
print("Before:",a)
a[:] = [101,102,103]
assert a is b
print("After:",a)
>>>
Before: ['a','b','c','d','e','f','g','h']
After: [101,102,103]
```

#### 要点
*  不要写多余的代码：当*start*为0或者*end*为序列长度时，应将其省略
*  切片操作不会计较*start*与*end*是否越界，这使得我们很容易就能从序列的前端或后端开始，对其进行范围固定的切片操作(如`a[:20]`或`a[-20:]`)
*  对`list`赋值的时候，如果使用切片操作，就会把原列表中处在相关范围内的值替换成新值，即便他们的长度不同也依然可以替换(自动扩展或收缩)。

###  第6条： 在单次切片操作内，不要同时指定start,end和stride
除了基本的切片操作之外，Python还提供了`somelist[start:end:stride]`形式的写法，以实现步进式切割，也就是从每*n*个元素里取*1*个出来。
<br>例如，可以指定步进值(*stride*)，把列表中位于偶数索引出和奇数索引出的元素分成两组：
```python
a = ['red','orange','yellow','green','blue','purple']
odds = a[::2]
evens = a[1::2]
print(odds)
print(evens)
>>>
['red','yellow','blue']
['orange','green','purple']
```
但是，问题在于，采用**stride**方式进行切片时，经常会出现不合符预期的结果。
<br>例如，Python中有一种常见的技巧，能够把以字节形式存储的字符串返还过来，这个技巧就是采用`-1`来做步进值。
```python
x = b'mongoose'
y = x[::-1]
print(y)
>>>
b'esoognom'
```
这种技巧对字符串和*ASCII*字符有效，但是对已经编码成`UTF-8`字符串的*Unicode*字符无效。
```python
w = '谢谢'
x = w.encode('utf-8')
y = a[::-1]
z = y.decode('utf-8')
>>>
UnicodeDecodeError: 'utf-8' codec can't decode byte 0x9d in position 0: invalid start byte.
```
除了`-1`之外，其他的负步进值有没有意义呢？
```python
a = ['a','b','c','d','e','f','g','h']
a[::2]        # ['a','c','e','g']
a[::-2]       # ['h','f','d','b']
```
上例中，`::2`表示从头部开始，每两个元素取一个。`::-2`则表示从尾部开始向前选取，每两个元素取一个。
<br>那么，`2:::2`,`-2::-2`,`-2:2:-2`,`2:2:-2`又分别代表什么意思呢？请看下面的例子：
```python
a = ['a','b','c','d','e','f','g','h']
a[2::2]     # ['c','e','g']
a[-2::-2]    # ['g','e','c','a']
a[-2:2:-2]   # ['g','e']
a[2:2:-2]    # []
```
通过上面例子可以看出：切割列表时，如果指定了*stride*，那么代码可能会变得相当费解，也难以阅读。这种写法使得*start*和*end*索引的含义变得模糊，尤其是*stride*为负值时。
<br>为了解决这种问题，我们不应该把*stride*与*start*和*end*一起使用，如果一定要用*stride*，尽量采用正值，同时省略*start*和*end*索引。如果一定要配合*start*或*end*索引来使用*stride*，那么请考虑先做步进式切片，把切割结果赋给某个变量，然后在那个变量上面做第二次切割。
```python
a = ['a','b','c','d','e','f','g','h']
b = a[::2]    # ['a','c','e','g']
c = b[1:-1]   # ['c','e']
```
上面这种先做步进切割，再做范围切割的办法，会多产生一份原数据的浅拷贝。执行第一次切割操作时，应该尽量缩减切割后的列表尺寸。如果你说开发的程序对执行时间或内存用量的要求非常严格，以致不能采用两阶段切割法，那就请考虑Python内置的`itertools`模块，该模块有个`islide`方法，这个方法不允许为*start*，*end*或*stride*指定负值（参见第46条）。

#### 要点
*  既有*start*和*end*，又有*stride*的切片操作，可能会令人费解
*  尽量使用*stride*为正数，且不带*start*或*end*索引的切片操作。尽量避免用负数做*stride*
*  在同一个切片操作内，不要同时使用*start*，*end*和*stride*。如果确实需要执行这种操作，考虑将其拆解为两条赋值语句，其中一条做范围切片，另一条做步进切片，或考虑使用内置的`itertools`模块中的`islice`。

###  第7条： 用列表推导来取代map,和filter
Python提供了一种精炼的写法来根据一份列表来制作另外一份，称为**List comprehension(列表推导)**。
```python
a = [1,2,3,4,5,6,7,8,9,10]
squares = [x*x for x in a]
print(squares)
>>>
[1,4,9,16,25,36,49,64,81,100]
```

除非是调用只有一个参数的函数，否则，对于简单的情况，列表推导比内置的`map`函数更加清晰。如果使用`map`就需要创建*lambda*函数，这会使代码看起来有些乱。`squares = map(lambda x: x ** 2, a)`
列表推导也可以直接过滤原列表的元素，只需要在循环后面添加条件表达式即可：
```python
even_squares = [x**2 for x in a if x%2==0]
print(even_squares)
>>>
[4,16,36,64,100]
```
上述代码如果采用`map`函数，则需要和`filter`函数结合使用。但是代码会写的比较难懂：
```python
alt = map(lambda x: x**2, filter(lambda x: x%2==0,a))
assert even_squares== list(alt)
```

字典(*dict*)与集合(*set*)也有和列表类似的推导机制。
```python
chile_rank = {'ghost':1,'habanero':2,'cayenne':3}
rank_dict = {rank:name for name,rank in child_rank.items()}
chile_len_set = {len(name) for name in rank_dict.values()}
print(rand_dict)
print(chile_len_set)
>>>
{1: 'ghost',2: 'habanero',3: 'cayenne'}
{8, 5, 7}
```

#### 要点
*  列表推导要比内置的`map`和`filter`函数清晰，因为它无需额外编写*lambda*表达式
*  列表推导可以过滤输入列表中的某些元素，如果用`map`来做，需搭配`fliter`函数方能实现
*  字典(`dict`)与集合(`set`)也支持列表推导

###  第8条： 不要使用含有两个以上表达式的列表推导
列表推导支持多重循环。
<br>例如，要把二维矩阵(即包含列表的列表)转化为一维列表，可以用包含两个`for`表达式的列表推导来实现。
```python
matrix = [[1, 2, 3],[4, 5, 6],[7, 8, 9]]
flat = [x for row in matrix for x in row]
print(flat)
>>>
[ 1, 2, 3, 4, 5, 6, 7, 8, 9]
```
这是一种合理的用法，还有一种包含多种循环的合理用法，那就是根据输入列表来创建有两层深度的新列表。
<br>例如，对二维矩阵中的每个元素取平方，然后构建新的矩阵。
```python
quared = [[ x**2 for x in row] for row in matrix]
print(squared)
>>>
[[1, 4, 9],[16, 25, 36],[49, 64, 81]]
```

如果表达式里有过多的循环，列表推导将变得很长，这时不得不把它分成多行来写才能保证代码的可读性。
```python
my_lists = [
    [[1, 2, 3],[4, 5, 6]],
    # ...
]
flat = [ x for sublist in my_lists
          for sublist2 in sublist
          for x in sublist2]

print(flat)
```
可以看出，这时列表推导并没有比普通写法简洁，这种时候应该用普通的循环语句写法来增加代码的可读性。
```python
flat = []
for sublist in my_lists:
    for sublist2 in sublist:
        flat.append(sublist2)
```

列表推导也支持`if`过滤条件，每一级都支持。处于同一循环级别的`if`条件默认形成`and`表达式。
```python
a = [1,2,3,4,5,6,7,8,9,10]
b = [x for x in a if x> 4 if x%2 ==0]    #默认会形成and表达式
c = [x for x in a if x > 4 and if x%2 ==0]    #显式指定and

#过滤出每行总和大于10且能被3整除的元素，列表推导可以实现单可读性较差
matrix = [[ 1, 2, 3],[ 4, 5, 6],[ 7, 8, 9]]
filtered = [[x for x in row if x%3==0]
            for row in matrix if sum(row) >= 10 ]
print(filtered)
>>>
[[6],[9]]
```

在列表推导中，最好不要使用两个以上的表达式。可以使用两个条件，两个循环或一个条件搭配一个循环，但不能再多了，否则可读性变差。
<br>如果要写的代码比较复杂，推荐使用普通的`if`和`for`语句，并编写辅助函数(参考第16条)。

#### 要点
*  列表推导支持多级循环，每一级循环也支持多个条件
*  超过两个表达式的列表推导是很难理解的，应该尽量避免

###  第9条： 用生成器表达式来改写数据量较大的列表推导
列表推导的缺点是，推导过程中，对于输入序列中的每个值来说，可能都要创建仅含一项元素的全新列表。输入数据较少时没有问题，但输入数据非常多时，可能会消耗大量内存甚至导致程序崩溃。
<br>例如，读取一份文件并返回每行的字符数。若使用列表推导来做，则需把文件每一行的长度都保存在内存中。如果这个文件特别大，或是通过无休止的*network socket(网络套接字)*来读取，那么这种列表推导就会出问题。下面的这段列表推导代码，只适合处理少量的输入值：
```python
value = [len(x) for x in open('./my_file.txt','rb')]
print(values)
>>>
[100, 57, 15, 1, 12, 75, 5, 86, 89,11]
```

为了解决此类问题，Python提供了生成器表达式(*generator expression*)，它是对列表推导和生成器的一种泛化(*generalization*)。生成器表达式在运行的时候，并不会把整个输出序列都呈现出来，而是会估值为迭代器(*iterator*)，这个迭代器每次可以根据生成器表达式产生一项数据。把实现列表推导所用的那种写法放在一对圆括号`()`中，就构成了生成器表达式。
<br>二者的区别在于，对生成器表达式求值时，它会立刻返回一个迭代器，而不会深入处理文件中的内容。
```python
it = ( len(x) for x in open('/tmp/my_file.txt'))
print(it)
>>>
<generator object <genexpr> at 0x101b81480>
```

可以逐次调用迭代器内置的`next`函数来输入下一个值。这样每次都只有一个值，不需要担心内存问题。
```python
print(next(it))
print(next(it))
>>>
100
57
```

生成器表达式还有个好处，就是可以相互组合。
<br>例如，`roots = ((x,x**0.5) for x in it)`会把刚才那个生成器表达式所返回的迭代器用作另外一个生成器表达式的输入值。
<br>外围的迭代器每次前进时，都会推动内部那个迭代器，这样就产生了连锁效应，使得执行循环，评估条件表达式，对接输入和输出等逻辑都组合在了一起。
```python
print(next(roots))
>>>
(15,3.872983346207417)
```
上面这种连锁生成器表达式，可以迅速在Python中执行。如果要把多种手法组合起来，以操作大批量的输入数据，最好使用生成器表达式来实现。
<br>**注意**：由生成器表达式所返回的迭代器是有状态的，用过一轮之后，就不能反复使用了(参考第17条)。

#### 要点
*  当输入的数据量较大时，列表推导可能会因为占用太多内存而出现问题
*  由生成器表达式所返回的迭代器，可以逐次产生输出值，从而避免了内存用量问题
*  把某个生成器表达式所返回的迭代器，放在另一个生成器表达式的`for`子表达式中，即可将二者组合起来
*  串在一起的生成器表达式执行速度很快


###  第10条： 尽量用enumerate取代range
在一系列整数上面迭代时，内置的`range`函数很有用。
```python
from random import *
random_bits = 0
for i in range(64):
    if randint(0,1):
        random_bits |= 1 << i
```
对于字符串列表，也可以直接在上面迭代。
```python
flavor_list = ['vanilla', 'chocolate', 'pecan', 'strawberry']
for flavor in flavor_list:
    print("%s is delicious."%flavor)
```
当迭代列表的时候，通常还想知道当前元素在列表中的索引。一种方法是用`range`来实现：
```python
for i in range(len(flavor_list)):
    flavor = flavor_list[i]
    print("%d:%s"%(i+1, flavor))
```
与单纯迭代`flavor_list`或者单纯使用`range`的代码来比较，上面这段代码有些生硬，因为我们必须获取列表长度，并通过下标来访问数组。
Python提供了内置的`enumerate`函数以解决此类问题。`enumerate`可以把各种迭代器包装成生成器，以便稍后产生输出值。生成器每次产生一对输出值，其中，前者表示循环下标，后者表示从迭代器中获取到的下一个序列元素。
```python
for i, flavor in enumerate(flavor_list):
    print("%d:%s"%(i+1,flavor))
>>>
1: vanilla
2: chocolate
3：pecan
4：strawberry
```
还可以指定`enumerate`函数开始计数时所用的值,默认为0.
```python
for i, flavor in enumerate(flavor_list,1):
    print("%d: %s"%(i,flavor))
>>>
1: vanilla
2: chocolate
3：pecan
4：strawberry
```

#### 要点
*  enumerate函数提供了一种精简的写法，可以在遍历迭代器时获知每个元素的索引
*  尽量用enumerate来改写那种将range与下标访问相结合的序列遍历代码
*  可以给enumerate提供第二个参数，以指定开始计数时所用的值(默认为0)


###  第11条： 用zip函数同时遍历两个迭代器
Python3中的`zip`函数可以把两个或两个以上的迭代器封装为生成器，以便稍后求值。这种`zip`生成器会从每个迭代器中获取该迭代器的下一个值，然后把这些值汇聚成元组(tuple)。
```python
names = ['Cecilia', 'Lise', 'Marie']
letters = [ len(n) for n in names]
for name,count in zip(names,letters):
    if count > max_letters:
        longest_name = name
        max_letters = count
```
但是内置的`zip`有两个问题需要注意。
第一个问题是，Python2中的`zip`并不是生成器，而是会把开发者所提供的那些迭代器都平行地遍历一次，在此过程中能够，它都会把那些迭代器所产生的值汇聚成元组，并把那些元素所构成的列表完整地返回给调用者。这可能会占用大量内存并导致程序崩溃。如果要在Python2里用`zip`来遍历数据量非常大的迭代器，那么应该使用`itertools`内置模块中的`izip`函数(参考第16条)
第二个问题是，如果输入的迭代器长度不同，那么`zip`会表现出奇怪的行为。例如，我们又给*names*里添加了一个名字，但却忘了吧这个名字的长度更新到*letters*之中。
```python
names.append("Rosalind")
for name, count in zip(names,letters):
  print(name)
>>>
Cecilia
Lise
Marie
```
新元素并没有出现在遍历结果中。这正是`zip`的运作方式。受封装的那些迭代器中，只要有一个耗尽，`zip`就不再产生元组了。如果待遍历的迭代器长度都相同，那么这种运作方式不会出现问题，由列表推导所推算出的派生列表一般都和源列表等长。如果待遍历的迭代器长度不同，那么`zip`会提前终止，这将导致意外的结果。若不能确定`zip`所封装的列表是否等长，则可考虑改用`itertools`内置模块中的`zip_longest`函数(*Python2*：`izip_longest`)

#### 要点
*  内置的zip函数可以平行地遍历多个迭代器
*  Python3中的zip相当于生成器，会在遍历过程中逐次产生元组，而Python2中哦哦那个的zip则是直接把这些元组完全生成好，并一次性地返回整份列表
*  如果提供的迭代器长度不等，那么zip就会自动提前终止
*  itertools内置模块中的zip_longest函数可以平行地遍历多个迭代器，而不用在乎他们的长度是否相等(参见第46条)


###  第12条： 不要在for和while循环后面写else块
Python提供了一种很多编程语言都不支持的功能，那就是可以在循环内部的语句块后面直接编写`else`块。
```python
for i in range(3):
    print("Loop in %d!"% i )
else:
    print("Else Block!")
>>>
Loop in 0
Loop in 1
Loop in 2
Else Block!
```
这种`else`块会在整个循环执行完毕之后立刻运行。既然如此，它为什么叫`else`呢？
在`if/else`语句中，`else`的意思是：如果不执行前面那个`if`块，就执行`else`块。在`try/finally`语句中的`except`也是类似：如果前面那个`try`块没有成功执行，那就执行`except`块。
同理，`try/except/else`也是如此(参考第13条)，该处`else`的含义是：如果前面的`try`块没有失败，那就执行`else`块。`try/finally`同样非常直观，这里的`finally`的意思是：执行过前面的`try`块之后总是执行`finally`块。
明白了`else`、`except`、`finally`的含义之后，刚接触python的程序员可能会把`for/else`结构中的`else`块理解为：如果循环没有正常执行完，那就执行`else`块。实际上刚好相反--在循环里用break语句提前跳出，会导致程序不执行`else`块。
```python
for i in range(3):
    print('Loop %d' % i)
    if i == 1:
        break
else:
    print('else block')
>>>    
Loop 0
Loop 1
```
还有一个奇怪的地方，如果`for`循环要遍历的序列是空的，那么就会立刻执行`else`块。
```python
for x in []:
    print('Never runs')
else:
    print('else block')
>>>
else block
```
初始条件为`false`的`while`循环，如果后面跟着`else`块，那它也会立刻执行。
```python
while False:
    print('Never runs')
else:
    print('For Else block!')
>>> For Else block!
```
知道了循环后面的`else`块所标新出的行为之后，我们会发现：在搜索某个事物的时候，这种写法是有意义的。例如，要判断两个数是否互质(也就是判断两者除了1之外，是否还有其他的公约数)，可以把有可能成为公约数的每个值都遍历一轮，逐个判断两数是否能以该值为公约数。尝试完每一种可能的值之后，循环就结束了。如果两个数确实互质，那么在执行循环的过程中，程序就不会因`break`语句而跳出，于是，执行完循环后，程序会紧接着执行`else`块。
```python
a = 4
b = 9
for i in range(2, min(a, b) + 1):
    print('Testing', i)
    if a % i == 0 and b % i == 0:
        print('not coprime')
        break
else:
    print('coprime')
>>>
Testing 2
Testing 3
Testing 4
coprime
```
实际上，我们不会这样写代码，而是会用辅助函数来完成计算。这样的辅助函数，有两种常见的写法。
*   第一种写法是，只要发现受测参数符合自己先要搜寻的条件，就尽早返回。如果整个循环都完整地执行了一遍，那就说明受测参数不符合条件，于是返回默认值。
    ```python
    def coprime(a, b):
        for i in range(2, min(a, b) + 1):
            if a % i == 0 and b % i == 0:
            return False
        return True
    ```
*   第二种写法是，用变量来记录受测参数是否符合自己想要搜寻的条件。一旦符合，就用`break`跳出循环。
    ```python
    def coprime2(a, b):
        is_coprime = True
        for i in range(2, min(a, b) + 1):
            if a % i == 0 and b % i == 0:
                is_coprime = False
                break
        return is_coprime
    ```
对于不熟悉`for/else`的人来说，这两种写法都要比早前那种写法清晰很多。`for/else`结构中的`else`块虽然也能够实现相应的功能，但是将来回顾这段程序的时候，却会令阅读代码的人(包括你自己)感到相当费解。像循环这种简单的语言结构，在Python程序中应该写的非常直白才对。所以，我们完全不应该在循环后面使用`else`块。

#### 要点
*  Python有种特殊语法，可以在for及while循环的内部语句块之后紧跟一个else块
*  只有当整个循环主体都没遇到break语句时，循环后面的else块才会执行
*  不要再循环后面使用else块，因为这种写法既不直观又容易引人误解

###  第13条： 合理利用try/except/else/finally结构中的每个代码块
Python程序的异常处理可能要考虑四种不同的时机。这些时机可以用`try`、`except`、`else`和`finally`块来表述。符合语句中的每个块都有特定的用途，他们可以构成很多种有用的组合方式(参考第51条)。
*   finally块
    如果既要将异常向上传播，又要在异常发生时执行清理工作，那就可以使用`try/finally`结构。这种结构有意向常见的用途，就是确保程序能够可靠地关闭文件句柄(另一种写法参考第43条)。
    ```python
    # 此处可能引发IO异常
    handle = open('/tmp/random_data.txt')
    try:
        # 可能引发UnicodeDEcode异常
        data = handle.read()
    finally:
        # 总是在try语句结束后关闭文件
        handle.close()
    ```
    `open`方法必须放在`try`块外面，因为如果打开文件时发生异常(例如，文件不存在)，那么程序应该跳过`finally`块
*   `else`块
    `try/except/else`结构可以清晰地描述出那些异常会由自己的代码来处理，哪些异常会传播到上一级。如果`try`块没有发生异常，那么就执行`else`块。有了这种`else`块，我们可以尽量缩减`try`块内的代码量，使其更加易读。例如，要从字符串中加载*JSON*字典数据，然后返回字典里某个键所对应的值。
    ```python
    def load_json_key(data, key):
    try:
        result_dict = json.loads(data)  # 可能引发ValueError异常
    except ValueError as e:
        raise KeyError from e
    else:
        return result_dict[key]       # 可能引发KeyError
    ```
    如果数据不是有效的*JSON*格式，那么用`json.loads`解码时，会产生`ValueError`。这个异常会有`except`块来捕获并处理。如果能够解码，那么`else`块里的查找语句就会执行，它会根据建来查处相关的值。查询时若有异常，则该异常会向上传播，因为查询语句并不在刚才那个`try`块的范围内。这种`else`子句，会把`try/except`后面的内容和`except`块本身区分开，使异常的传播行为变得更加清晰。
*   混合使用
    如果要在符合语句中把上面izhng机制都用到，那就编写完整的`try/except/else/finally`结构。例如，要从文件中读取某项事务的描述信息，处理该事务，然后就地更新该文件。为了实现此功能，我们可以用`try`块来读取文件并处理其内容，用`except`块来应对`try`块中可能发生的相关异常，用`else`块实时地更新文件并把更新中可能出现的异常回报给上级代码。然后用`finally`块来清理文件句柄。
    ```python
    UNDEFINED = object()
    def divide_json(path):
        handle = open(path,'r+')    # 可能抛出IOError异常
        try:
            data = handle.read()    # 可能引发UnicodeDecodeError异常
            op = json.loads(data)   # 可能引发ValueError异常
            value = (
                op['numeerator'] /   # 这里是除号！
                op['denominator']   # 可能引发ZeroDivisionError异常
        except ZeroDivisionError as e:
            return UNDEFINED
        else:
            op['result'] = value
            result = json.dumps(op)
            handle.seek(0)
            handle.write(result)    # 可能引发IOError异常
            return value
        finally:
            handle.close()        # 肯定能够被执行成功
    ```
    这种写法很有用，因为这四块代码互相配合得非常到位。例如，即使`else`块在写入`result`数据时发生异常，`finally`块中关闭文件句柄的那行代码，也依然能够执行。


#### 要点
*  无论try块是否发生异常，都可利用try/finally符合语句中的finally块来执行清理工作
*  else块可以用来缩减try块中的代码量，并把没有发生异常时所要执行的语句与try/except代码块隔开
*  顺利运行try块后，若想使某些操作能在finally块的清理代码之前执行，则可将这些操作写到else块中(必须要有except块)

##   第二章 函数
###  第14条： 尽量用异常来表示特殊情况，而不要返回None
```python
def divide(a, b):
  try:
      return a/b
  except ZeroDivisionError as e:
      return ValueError("Invalid inputs") from e # 抛出异常而不是返回None
```
调用时如下：
```python
x, y = 5, 2
try:
    result = divide(x, y)
except ValueError:
    print("Invalid inputs")
else:
    print("Result is %.1f"% result)

>>>
Result is 2.5
```
#### 要点
*  用None 这个返回值来表示特殊意义的函数，很容易使调用者犯错，因为None 和0 及空字符串之类的值，在条件表达式里都会评估为False
*  函数在遇到特殊情况时，应该抛出异常，而不要返回None。调用者看到该函数的文档中所描述的异常之后，应该就会编写相应的代码来处理它们了


###  第15条： 了解如何在闭包里使用外围作用域中的白能量
```python
def sort_priority(values, group):
    def helper(x):
        if x in group:
            return (0, x)
        return (1, x)
    values.sort(key=helper)

values = [1,5,3,9,7,4,2,8,6]
group = [7,9]
sort_priority(values,group)
print values
>>>
[7, 9, 1, 2, 3, 4, 5, 6, 8]
```
这个函数能够运作是基于如下三个原因：
*   Python支持闭包(*closure*)。闭包是一种定义在某个作用域中的函数，这种函数以农了那个作用域里面的变量。`helper`函数之所以能够访问`sort_priority`的`group`参数，原因就在于此。
*   Python的函数是一级对象(*first-class object*),也就是说，我们可以直接饮用函数，把函数赋给变量，把函数当成参数传给其他函数，并通过表达式及`if`语句对其进行比较和判断，等等。于是，我们可以把`helper`这个闭包函数，传给`sort`方法的`key`参数。
*   Python使用特殊的规则来比较两个元组。它首先比较个元组中下标为0的对应元素，如果相等，在比较下标为1的对应元素，如果还是相等，那就继续比较下标为2的对应元素，以此类推。

下面我们改进一下这个函数，让他可以返回一个是否有values的元素包含在group里的状态。
```python
def sort_priority2(values, group):
    found = False   # 作用域在sort_priority2函数
    def helper(x):
        if x in group:
            found = True  # 作用在helper函数
            return (0, x)
        return (1, x)
    values.sort(key=helper)
    return found

found = sort_priority2(values, group)
print("Found:",found)
print(values)
>>>
Found: False
[7, 9, 1, 2, 3, 4, 5, 6, 8]
```
可以看到，排序结果是对的，**但是found值不对**。为什么呢？
在表达式中引用变量是，Python解释器将按如下顺序遍历各作用域，以解析改引用：
1. 当前函数的作用域
2. 任何外围作用域(例如包含当前函数的其他函数)
3. 包含当前代码的那个模块的作用域(也叫全局作用域，*global scope*)
4. 内置作用域(也就是包含`len`和`str`等函数的那个作用域)

如果这些作用域内都没有定义过该变量，则抛出`NameError`异常。
给变量赋值时，规则有所不同。如果当前作用域内定义了这个变量，就给其赋新的值，如果当前作用域没有定义过这个变量，Python则会这次赋值视为是对该变量的定义。而变量的作用域就是赋值操作所在的作用域。这也是`sort_priority2`中返回错误的found值的原因--将found值赋值为True是在helper闭包中进行的，相当于在闭包中新定义了一个变量，作用域仅限于`helper`这个闭包函数中。所以，`sort_priority2`函数中的found值并没有被更新。

**获取闭包内的数据**
Python3中可以使用`nonlocal`语句来表明这样的意图：给相关变量赋值的时候，应该在上层作用域中查找该变量。但是，`nonlocal`的限制在于，为了防止污染全局作用域，它不能延伸到模块级别。
```python
def srt_priority3(numbers, group):
    found = False
    def helper(x):
        nonlocal found 
        if x in group:
            found = True
            return (0, x)
        return (1, x)
    numbers.sort(key=helper)
    return found
```
注意，不要滥用`nonlocal`语句，因为比较长的函数语句中，比较难以追踪。建议只在简短的函数中使用它。
如果使用`nonlocal`的代码已经越来越复杂，建议像下面这样将相关的状态封装成辅助类。
```python
class Sorter(object):
    def __init__(self, group):
        self.group = group
        self.found = False

    def __call__(self, x):
        if x in self.group:
            self.found = True
            return (0, x)
        return (1, x)

sorter = Sorter(group)
numbers.sort(key=sorter)
assert sorter is True
```

**Python2中的值**
Python2中不支持`nonlocal`关键字。为了实现类似的功能，可以利用Python的作用域规则来解决，这是一种看来其不太优雅的Python编程习惯。
```python
### Python2
def sort_priority(numbers, group):
    found = [False]
    def helper(x):
        if x in group:
            found[0] = True
            return (0, x)
        return (1, x)
    numbers.sort(sort=helper)
    return found[0]
```
运行上面这段代码时，Python要解析found变量的当前值，于是，它会按照刚才所讲的变量搜寻规则，在上级作用域中查找这个变量。上级作用域中的found变量是个列表，由于列表本身是可供修改的(mutable)，所以获取到这个found列表后就可以在闭包里面通过`found[0] = True`来修改found的状态。
上级作用域中的变量是字典(dict),集合(set)或者某个类的实例时，这个技巧也同样适用。

#### 要点
*  对于定义在某作用域内的闭包来说，它可以引用和谐作用域中的变量
*  使用默认方式对闭包内的变量赋值，不会影响外围作用域中的同名变量
*  在Python3中，程序可以在闭包内用nonlocal语句来修饰某个名称，使该闭包能够修改外围作用域中的同名变量
*  在Python2中，程序可以使用可变值(例如，包含单个元素的列表)来实现与nonlocal语句相仿的机制
*  除了那种比较简单的函数，尽量不要用nonlocal语句

###  第16条： 考虑用生成器来改写直接返回列表的函数
生成器是使用`yield`表达式的函数。调用生成器函数是，它并不会真的运行，而是会返回迭代器。每次在迭代器上面调用内置的`next`函数是，迭代器会把生成器推导下一个`yield`表达式哪里。生成器传给`yield`每一个值，都会由迭代器返回给调用者。
```python
def index_words_iter(text):
  if text:
      yield 0
  for index, letter in enumerate(text):
      if letter ==" ":
          yield index + 1

address = "Four score ad sever years ago..."
result = list(index_words_iter(address))
```
使用列表的问题在于，数据量大是可能会耗尽内存甚至导致程序崩溃，而生成器则不会。
使用生成器需要留意的是，迭代器是由状态的，遍历过就不能再用。

#### 要点
*  使用生成器比把收集到的结果放入列表里返回给调用者更加清晰
*  由生成器函数所返回的那个迭代器，可以把生成器函数体中，传给yield表达式的那些值，逐次产生出来
*  无论输入量有多大，生成器都能产生一系列输出，因为这些输入量和输出量，都不会影响它在执行时所耗的内存

###  第17条： 在参数上面迭代时，要多加小心
如果参数是迭代器，那么迭代一次之后，迭代器就清空了，所以要注意的是不能对迭代器进行多次迭代。

#### 要点
*  函数在输入的参数上面多次迭代式要当心：如果参数是迭代器，那么可能会导致奇怪的行为并错失某些值
*  Python的迭代器协议，描述了容器和迭代器应该如何与iter和next内置函数，for循环及相关表达式相互配合
*  把__iter__方法实现为生成器，即可定义自己的容器类型
*  想判断某个值是迭代器还是容器，可以拿该值作为参数两次调用iter函数，若结果相同则是迭代器，调用内置的next函数，可令该迭代器前进一步

###  第18条： 用数量可变的位置参数减少视觉杂讯
可变的星型参数`*args`可以让函数调用起来更加清晰，并能够减少视觉杂讯。`*`表示该参数可有可无。
位置参数有两个问题需要注意：
1.  可变参数在被传递给函数的时候经常会被转变成元组。这意味着如果调用方对生成器使用了`*`操作符，程序将会遍历整个序列直至完毕。如果结果集元组包含大量数据，可能会因为内存不足而导致宕机。
    例如：
    ```python
    def my_generator():
        for i in range(10):
            yield i

    def my__func(*agrs):
        print(*args)

    it my_generator()
    my_func(*it)
    >>>
    (0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
    ```
    所以，`*args`参数适合传入参数数据量比较小的情况。
2.  一个带有`*args`参数的函数，以后不能再增加其他的位置参数了，降低了移植性。
    <br>如果你想给函数增加一个位置参数，只能在`*args`前面添加，也必须修改原来调用该函数的既存代码。
    例如：
    ```python
    def log(sequence, message, *values):
        if not values:
            print('%s: %s' % (sequence, message))
        else:
            values_str = ', '.join(str(x) for x in values)
            print('%s: %s：%s' %(sequence, message, values_str))

    # 新的调用可以正常的工作
    log(1, 'Favorites', 7, 33)
    # 原来的函数调用方式不能正常的工作
    log('Favorite numbers', 7, 33)
    >>>
    1： Favorites: 7, 33
    Favorite numbers: 7: 33
    ```

#### 要点
*  在`def`语句中使用`*args`,即可令函数接收数量可变的位置参数
*  调用函数时，可以采用\*操作符，把序列中的元素当成位置参数，传给该函数
*  对生成器使用\*操作符，可能导致程序耗尽内存并崩溃
*  在已经接受`*args`参数的函数上面继续添加位置参数，可能会产生难以排查的bug

###  第19条： 用关键字参数来表达可选的行为
Python支持位置参数，也支持关键字参数。所有的位置参数都可以按照关键字来传递(即：`param=value`)。
位置参数必须保证顺序和函数定义的参数列表一致，关键字参数则不限制顺序，但混合使用的时候，必须先保证函数所要求的全部位置参数都已经指定好。
使用关键字参数，有以下三个好处：
1.  使代码清晰易读。
2.  可以在函数定义中给关键字参数提供默认值。
3.  使函数容易扩展，扩展之后的函数依然能与既存的调用代码保持兼容，移植性好。

#### 要点
*  函数参数可以按位置或关键字来指定
*  只使用位置参数来调用函数，可能会导致这些参数值的含义不够明确，而关键字参数则能阐明每个参数的意图
*  给函数添加新的行为时，可以使用带默认值的关键字参数，以便与原有的函数调用代码保持兼容
*  可选的关键字参数，总是应该以关键字形式来指定，而不应该以位置参数的形式来指定

###  第20条： 用None和文档字符串来描述具有动态默认值的参数
函数参数的默认值是在模块加载的时候求出并设置好的。包含这段代码的模块一旦被加载，参数的默认值就固定不变了。
如果是`{}`这种可变类型，容易引起奇怪的行为。一定要用`None`作为形式上的默认值，并在注释里说明。
反例：
```python
def decode(data, default={}):
    try:
        return json.loads(data)
    except ValueError:
        return default

foo = decode('bad data')
foo['stuff'] = 5
bar = decode('also bad')
bar['meep'] = 1
print('Foo:', foo)
print('Bar:', bar)
>>>
Foo: {'stuff': 5, 'meep': 1}
Bar: {'stuff': 5, 'meep': 1}
```
改成下面这样就OK了：
```python
def decode(data, default=None):
    """Load JSON data from string.

    Args:
        data: JSON data to be decoded.
        default: Value to return if decoding fails.
            Defaults to an empty dictionary.
    """

    if default is None:
        default = {}
    try:
        return json.loads(data)
    except ValueError:
        return default

foo = decode('bad data')
foo['stuff'] = 5
bar = decode('also bad')
bar['meep'] = 1
print('Foo:', foo)
print('Bar:', bar)
>>>
Foo: {'stuff': 5}
Bar: {'meep': 1}
```

#### 要点
*  参数的默认值，只会在程序加载模块并读到本函数的定义时评估一次。对于\{\}或\[\]等动态的值，这可能会导致奇怪的行为
*  对于以动态值作为实际默认值的关键字参数来说，应该把形式上的默认值写为None,并在函数的文档字符串里面描述该默认值所对应的实际行为

###  第21条： 用只能以关键字形式指定的参数来确保代码明晰
为了确保代码调用的清晰易读，强制调用者必须以关键字的形式来提供参数，而不能按照位置来提供参数。
如下面所示，`safe_division_c`函数，带有两个只能以关键字形式来指定的参数，参数列表里的`*`号，标志着位置参数就此终结，之后的那些参数，都只能以关键字形式来指定。
```python
def safe_division_c(number, division, *, ignore_overflow=False, ignore_zero_division=False):
    try:
        return number / divisor
    except OverflowError:
        if ignore_overflow:
            return 0
        else:
            raise
    except ZeroDivisionError:
        if ignore_zero_division:
            return float('inf')
        else:
            raise

# 现在使用位置参数来调用的话，函数就会报错。
safe_division_c(1, 10**500, True, False)
>>>
TypeError: safe_division_c() takes 2 position arguments but 4 ware given.

# 使用关键字参数来调用的话，是OK的。
safe_division_c(1, 0, ignore_zero_division=True)
try:
    safe_division_c(1, 0)
except ZeroDivisionError:
    pass
```

**Python2中实现只能以关键字来指定的参数**
Python2并没有明确的语法来定义这种只能以关键字形式指定的参数。不过，我们可以在参数列表中使用`**`操作符，并且令函数在遇到无效的调用时抛出`TypeErrors`,这样就可以实现与Python3相同的功能了。`**`操作符与`*`操作符类似，但区别在于，它不是用来接收数量可变的位置参数，而是用来接收任意数量的关键字参数。即便某些关键字参数没有定义在函数中，它也依然能够接受。
```python
# Python2
def print_args(*args, **kwargs):
    print 'Positional:', args
    print 'Keyword:', kwargs

print_args(1, 2, foo='bar', stuff='meep')
>>>
Positional: (1, 2)
Keyword: {'foo': 'bar', 'stuff': 'meep'}
```
为了使Python2版本的`safe_division`函数具备只能以关键字形式指定的参数，我们可以先令该函数接受`**kwargs`参数，然后，用`pop`方法吧期望的关键字参数从`kwargs`字典里取走，如果字典的键里没有那个关键字，那么`pop`方法的第二个参数就会成为默认值。最后，为了防止调用者提供无效的参数值，我们需要确认`kwargs`字典里面已经没有关键字参数了。
```python
# Python 2
def safe_division(number, divisor, **kwargs):
    ignore_overflow = kwargs.pop('ignore_overflow', False)
    ignore_zero_division = kwargs.pop('ignore_zero_division', False)
    if kwargs:
        raise TypeError("Unexpected **kwargs: %r"%kwargs)
    # ···

# 这样既可以用不带关键字参数的方式来调用，也可以用有效的关键字参数来调用
safe_division(1, 10)
safe_division(1, 0, ignore_zero_division=True)
safe_division(1, 10**500, ignore_overflow=True)

# 和Python3一样，也不能以位置参数的形式来指定关键字参数的值
safe_division(1, 0, False, True)
>>>
TypeError：safe_division() takes 2 positional arguments but 4 were given.

# 如果传入了不符合预期的关键字参数，也会触发类型错误
safe_division(0, 0, unexpected=True)
>>>
TypeErroe: Unexpected **kwargs: {'unexpected': True}
```

#### 要点
*  关键字参数能够使函数调用的意图更加明确
*  对于各参数之间很容易混淆的函数，可以声明只能以关键字形式指定的参数，以确保调用者必须通过关键字来指定他们。对于接收多个Boolean标志的函数，更应该这样做
*  在编写函数时，Python3有明确的语法来定义这种只能以关键字形式指定的参数
*  Python2的函数可以接受`**kwargs`参数，并手工抛出`TypeError`异常，以便模拟只能以关键字形式来指定的参数


##   第三章 类与继承
###  第22条： 尽量用辅助类来维护程序的状态，而不要用字典和元组
用来保存状态的数据结构一旦变得过于复杂，就应该考虑将其拆解为类，以便提供更为明确的借口，并更好地封装数据。这样也能够在借口与具体实现之间创建抽象层。
**把嵌套结构重构为类**
`collections`模块中的`namedtuple`(具名元组)很容易能定义出精简而又不可变的数据类。
```python
import collections
Grade = collections.namedtuple('Grade', ('score', 'weight'))

class Subject(object):
    def __init__(self):
        self._grades = []

    def report_grade(self, score, weight):
        self._grades.append(Grade(score, weight))

    def average_grade(self):
        total, total_weight = 0, 0
        for grade, in self._grades:
            total += grade.score * grade.weight
            total_weight += grade.weight
        return total / total_weight

class Stduent(object):
    def __init__(self):
        self._subjects = {}

    def subject(self, name):
        if name not in self._subjects:
            self._subjects[name] = Subject()
        return self._subjects[name]

    def average_grade(self):
        total, count = 0, 0
        for subject in self._subjects.values():
            total += subject.average_grade()
            count += 1
        return total / count

class Gradebook(object):
    def __init__(self):
        self._student = {}

    def student(self, name):
        if name not in self._students:
            self._students[name] = Student()
        return self._students[name]

##### 测试 ########
book = Gradebook()
albert = book.student('Albert Einstein')
math = albert.subject('Math')
math.report_grade(80, 0.10)
# ···
print(albert.average_grade())
>>>
81.5
```
构建这些具名元组时，既可以按位置指定其中各项，也可以采用关键字来指定。这些字段都可以通过属性名称访问。由于元组的属性都带有名称，所以当需求发生变化，以致要给简单的数据容器添加新的行为时，很容易就能从`namedtuple`迁移到自己定义的类。
但是，`namedtuple`具有以下局限性：
*   `namedtuple`类无法指定个参数的默认值。对于可选属性比较多的数据来说，`namedtuple`用起来很不方便。如果数据并不是一系列简单的属性，那还是定义自己的类比较好。
*   `namedtuple`实例的各项属性，依然可以通过下标及迭代来访问。这可能导致其他人以不符合设计者意图的方式使用这些元组，从而使以后很难把它迁移为真正的类，对于那种公布给外界使用的API来说，更要注意这个问题。如果没办法完全控制`namedtuple`实例的用法，那么最好是定义自己的类。

#### 要点
*  不要使用包含其他字典的字典，也不要使用过长的元组
*  如果容器中包含简单而又不可变的数据，那么可以先使用namedtuple来表示，待稍后有需要时，再修改为完整的类
*  保存内部状态的字典如果变得比较复杂，那就应该把这些代码拆解为多个辅助类

###  第23条： 简单的借口应该接受函数，而不是类的实例
Python有很多内置的API都允许调用者传入函数，以定制其行为。API在执行的时候，会通过这些*挂钩（hook）函数*来回调函数内的代码。
例如，`list`类型的`sort`方法接受可选的*key*参数,用以指定每个索引位置上的值应该如何排序。
```python
names = ['Socrates', 'Archimedes', 'Plato', 'Aristotle']
names.sort(key=lambda x: len(x))
print(names)
>>>
['Plato', 'Socrates', 'Aristotle', 'Archimedes']
```

假如，我们要定制`defaultdict`类(参考第46条)的行为。给`defaultdict`传入一个产生默认值的挂钩，并令其统计出该字典一共遇到了多少个缺失的键。
一种实现是使用带状态的闭包(可读性不太好)：
```python
def increment_with_report(current, increments):
    added_count = 0

    def missing():
        nonlocal added_count  # 状态闭包
        added_count  += 1
        return 0

    result = defaultdict(missing, current)
    for key, amount in crements:
    result[key] += amount

    return result, added_count

current = {'green': 12, 'blue': 3}
incremetns = [
    ('red', 5),
    ('blue', 17),
    ('orange', 9)
]
result, count = increment_with_report(current, increments)
assert count == 2
```
另一种实现是定义一个小型的类，把需要追踪的状态封装起来：
```python
class CountMissing(object):
    def __init__(self):
        self.added = 0

    def missing(self):
        self.added += 1
        return 0

counter = CountMissing()
result = defaultdict(counter.missing, current)  #python中的函数是一级函数，所以可以直接在CountMissing实例上面引用missing函数
for key, amount in increments:
    result[key] += amount
assert counter.added == 2
```
这样比使用闭包清晰易懂，但是如果单看这个类，我们依然不太容易理解`CountMissing`的意图。`CountMissing`的对象由谁来构建？`missing`方法由谁来调用？该类以后是否需要添加新的公共方法？这些问题，都必须等看过了`defaultdict`的用法之后才能明白。为了厘清这些问题，我们可以在Python代码中定义名为`__call__`的特殊方法。该方法使相关对象能够像函数那样得到调用。此外，如果把这样的实例传给内置的`callable`函数，那么`callable`函数会返回`True`。
```python
class BetterCountMissing(object):
    def __init__(self):
        self.added = 0

    def __call__(self):
        self.added += 1
        return 0

counter = BetterCountMissing()
counter()
assert callable(counter)

# 把BetterCountMissing实例作为defaultdict函数的默认值挂钩
counter = BetterCountMissing()
result = defaultdict(counter, current)  # Relies on __call__
for key, amount in increments:
    result[key] += amount
assert counter.added == 2
```

#### 要点
*  对于链接各种Python组件的简单借口来说，通常应该给其直接传入参数，而不是先定义某个类，然后再传入该类的实例
*  Python中的函数和方法都可以像一级类那样引用，因此，他们与其他类型的对象一样，也能够放在表达式里面
*  通过调用`__call__`的特殊方法，可以使类的实例能够像普通的Python函数那样得到调用
*  如果要用函数来保存状态，那就应该定义新的类，并令其实现`__call__`方法，而不要定义带状态的闭包(参考第15条)

###  第24条： 以@classmethod形式的多态去通用地构建对象
Python只允许名为`__init__`的构造器方法，即不支持构造器多态，所以不能要求每个子类都提供兼容的构造器。
为了解决这个问题，可以使用`@classmethod`形式的多态。这种多态形式针对的是整个类，而不是该类的对象。
例如，要构建一个mapreduce的流程：
```python
class GenericInputData(object):
    def read(self):
        raise NotImplementedError

    @classmethod
    def generate_inputs(cls, config):
        ''' 接收一份含有配置参数的字典，而具体的子类则可以解读这些参数。
        '''
        raise NotImplementedError

class PathInputData(GenericInputData):
    def ____init__(self, path):
        super().__init__()
        self.path = path

    def read(self):
        return open(self.path).read()

    @classmethod
    def generate_inputs(cls, config):
        data_dir = config['data_dir']
        for name in os.listdir(data_dir):
            yield cls(os.path.join(data_dir, name))

class GenericWorker(object):
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None

    def map(self):
        raise NotImplementedError

    def reduce(self, other):
        raise NotImplementedError

    @classmethod
    def create_workers(cls, input_class, config):
        workers = []
        # input_class.generate_inputs是个类级别的多态方法。
        # 此外，这里通过cls形式来构造GenericWorker对象，而不是直接使用init方法。
        for input_data in input_class.generate_inputs(config):
            workers.append(cls(input_data))
        return workers

# 具体的GenericWoker子类
class LineCountWorker(GenericWorker):
    def map(self):
        data = self.input_data.read()
        self.result = data.count('\n')

    def reduce(self, other):
        self.result += other.result


## 构建一个map-reduce的流程 ##############################################
def generate_inputs(data_dir):
    for name in os.listdir(data_dir):
        yield PathInputData(os.path.join(data_dir, name))

# 创建LineCountWorker实例来使用generate_inputs返回的InputData实例。
def create_workers(input_list):
    workers = []
    for input_data in input_list:
        workers.append(LineCountWork(input_data))
    return workers

# 执行上述Worker实例，以便将Map-Reduce流程中的map步骤分发到多个线程(参考第37条)。然后，反复执行reduce方法，将map步骤的结果合并成一个最终值。
def execute(workers):
    threads = [Thread(target=w.map) for w in workers]
    for thread in threads: thread.start()
    for thread in threads: thread.join()

    first, reset in rest:
        first.reduce(worker)
    return first.result

# 封装一个通用的函数来执行上述代码。
def mapreduce(worker_class, input_class, config):
    workers = worker_class.create_workers(input_class, config)
    return execute(workers)

## 测试一下
with TemporaryDirectory() as tmpdir:
    write_test_files(tmpdir)
    config = {'data_dir': tmpdir}
    result = mapreduce(LineCountWorker, PathInputData, config)

## 现在可以基于这些来构造其他的GenericInputData和GenericWorker子类，而且不用修改已经写好的那些拼接代码。
```

#### 要点
*  在Python程序中，每个类只能有一个构造器，也就是`__init__`方法
*  通过`@classmethod`机制，可以用一种与构造器相仿的方式来构造类的对象
*  通过类方法多态机制，我们能够以更加通用的方式来构建并拼接具体的子类

###  第25条： 用super初始化父类
Python2.2开始增加了内置的`super`函数，并且定义了*方法解析顺序(method resolution order, MRO)*来解决父类初始化问题。
MRO以标准的流程来安排超类之间的初始化顺序(例如：深度优先，从左至右)，它也保证钻石顶部(菱形)那个公共基类的`__init__`方法只会运行一次。
```python
# Python2
class MyBaseClass(object):
    def __init__(self, value):
        self.value = value

class TimesFiveCorrect(MyBaseClass):
    def __init__(self, value):
        super(TimesFiveCorrect, slef).__init__(value)
        self.value *= 5

class PlusTwoCorrect(MyBaseClass):
    def __init__(self, value):
        super(PlusTwoCorrect, self).__init__(value)
        self.value += 2


class GoodWay(TimesFiveCorrect, PlusTwoCorrect):
    def __init__(self, value):
        super(GoodWay, self).__init__(value)

foo = GoodWay(5)
print("Should be 5 * (5 + 2) = 35 and is " , foo.value)
>>>
Should be 5 * (5 + 2) = 35 and is 35
```
看来上面的运行结果之后，我们可能觉得程序的计算顺序和自己所想的刚好相反。应该先运行`TimesFiveCorrect.__init__`, 然后运行`PlusTwoCorrect.__init__`, 并得出`(5 * 5) + 2 = 27`才对啊。
但实际上却不是这样的。程序的运行顺序会与`GoodWay`类的*MRO*保持一致，这个*MRO*顺序可以通过名为`mro`的类方法来查询。
```python
from pprint import pprint
pprint(GoodWay.mro())
>>>
[<class '__main__.GoodWay'>,
<class '__main__.TimesFiveCorrect'>,
<class '__main__.PlusTwoCorrect'>,
<class '__main__.MyBaseClass'>,
<class 'object'>]
```
所以，调用`GoodWay(5)`的时候，它会调用`TimesFiveCorrect.__init__`, 而`TimesFiveCorrect.__init__`又会调用`PlusTwoCorrect.__init__`, `PlusTwoCorrect.__init__`再调用`MyBaseClass.__init__`。
到达了钻石体系的顶部之后，所有的初始化方法会按照与刚才那些`__init__`相反的顺序来运作。于是，`MyBaseClass.__init__`会先把value设为5， 然后`PlusTwoCorrect.__init__`为它加2是value变为7，最后，`TimesFiveCorrect.__init__`会将value乘以5，使其变为35.
Python2中的`super`内置函数有两个问题值得注意：
*   `super`语句写起来有点麻烦。必须指定但钱所在的类和`self`对象，而且还要指定相关的方法名称(`__init__`)以及那个方法的参数。
*   调用`super`时，必须写出当前类的名称。所以，以后修改类体系的时候如果类名发生变化，必须修改每一条`super`调用语句。

Python3没有这些问题，它提供了一种不带参数的`super`调用方式，该方式的效果与用`__class__`和`self`来调用`super`相同。
```python
class Explicity(MyBaseClass):
    def __init__(self, value):
        super(__class__, self).__init__(values * 2)


class Implicit(MyBaseClass):
    def __init__(self, value):
        super().__init__(value * 2)

assert Expilcit(10).value == Implicit(10).value
```
Python3可以使用`__class__`来准确地引用当前类；但是Python2中没有定义`__class__`,所以不行，`self.__class__`也是行不通的，因为Python2是用一种特殊方式来实现`super`的(![参考](http://stackoverflow.com/questions/18208683/when-calling-super-in-a-derived-class-can-i-pass-in-self-class))。

#### 要点
*  Python采用标准的方法解析顺序来解决超类初始化次序及钻石继承问题
*  总是应该使用内置的super函数来初始化父类

###  第26条： 只在使用Mix-in组件制作工具类时进行多重继承
**mix-in**是一种小型的类，它只定义了其他类可能需要提供的一套附加方法，而不定义自己的实例属性，此外，它也不要求使用使用者调用自己的`__init__`构造器。
在*mix-in*里面通过动态监测机制先编写一套通用的功能代码，稍后在将其应用到其他很多类上面。分层地组合*mix-in*类可以减少重复代码并提升复用度。
例如，要把内存中的Python对象转换为字典形式，以便将其序列化(serialization),我们可以将这个功能写成通用的代码，以供其他类使用。
```python
class ToDictMixin(object):
    """实现的细节是:通过hasattr方法动态访问属性，通过isinstance方法动态检测对象类型，并用__dict__访问实例内部的字典。
    """
    def to_dict(self):
        return self._traverse_dict(self.__dict__)

    def _traverse_dict(self, instance_dict):
        output = {}
        for key, value in instance_dict.items()
            output[key] = self._traverse(key, value)
        return output

    def _traverse(self, key, value):
       if instance(value, ToDictMixin):
           return value.to_dict()
       elif instance(value, dict):
           return self._traverse_dict(value)
       elif isinstance(value, list):
           return [self._traverse(key, i) for i in value]
       elif hasattr(value, '__dict__'):
           return self._traverse_dict(value.__dict__)
       else:
           return value
```
下面定义的这个类演示如何用*mix-in*把二叉树表示为字典：
```python
class BinaryTree(ToDIctMixin):
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right


# 现在可以把大量相互关联的Python对象轻松地转换成字典。
tree = BinaryTree(10, left=BinaryTree(7, right=BinaryTree(9)),
    right=BinaryTree(13, left=BinaryTree(11)))
print(tree.to_dict())
>>>
{'left': {'left': None,
         'right': {'left': None, 'right': None, 'value': 9},
         'value': 7},
 'right': {'left': {'left': None, 'right': None, 'value': 11},
         'right': None,
         'value': 13},
  'value': 10
}
```
*mix-in*的最大优势在于，使用者可以随时安插这些通用的功能，并且能在必要的时候重新它们。
例如，定义一个`BinaryTree`子类，持有指向父节点的引用，假如采用刚才的`ToDictMixin.to_dict`来处理，会陷入死循环。
解决办法是在这个子类里重写`ToDictMixin._traverse`方法，令其之处理与序列化有关的值，从而使*mix-in*的实现代码不会陷入死循环。
```python
class BinaryTreeWithParent(BinaryTree):
    def __init__(self, value, parent=None, left=None, right=None):
        super().__init__(value, left=left, right=right)
        self.parent = parent

    """重写_traverse方法，不再遍历父节点，而是只把父节点所对应的数值插入最终生成的字典里。
    """
    def _traverse(self, key, value):
        if (isinstance(value, BinaryTreeWithParent) and key == 'parent'):
            return value.value #防止循环
        else:
            return super()._traverse(key, value)

# 不会出现死循环。
root = BinaryTreeWithParent(10)
root.left = BinaryTreeWithParent(7, parent=root)
root.left.right = BinaryTreeWithParent(9, parent=root.left)
print(root.to_dict)
>>>
{'left': {'left': None,
         'parent': 10,
         'right': {'left': None,
                  'parent': 7,
                  'right': None,
                  'value': 9},
          'value': 7},
  'parent': None,
  'right': None,
  'value': 10

}

# 有了上面的重写，如果其他类的某个属性也是BinaryTreeWithParent类型，ToDictMixin也会自动处理好。
class NamedSubTree(ToDictMixin):
    def __init__(self, name, tree_eith_parent):
        self.name = name
        self.tree_with_parent = tree_with_parent


my_tree = NamedSubTree('foobar', root.left.right)
print(my_tree.to_dict())  # 不会导致循环
>>>
{'name': 'foobar',
 'tree_with_parent': {'left': None,
                    'parent': 7,
                    'right': None,
                    'value': 9}
}
```

多个*mix-in*之间也可以相互组合。例如，可以编写这样一个mix-in,它能够为任意类提供通用的*JSON*序列化功能。我们可以假定：继承了mix-in的那个类，会提供名为`to_dict`的方法(此方法可能是那个类通过多重继承`ToDictMixin`而具备的，也有可能是自定义的。)
```python
class JsonMixin(object):

    @classmethod
    def from_json(cls, data):
        kwargs = json.loads(data)
        return cls(**kwargs)

    def to_json(self):
        return json.dumps(self.to_dict())
```
`JsonMixin`既定义了实例方法，又定义类方法。这两种行为都可以通过mix-in来提供。在本例中，凡是想继承`JsonMixin`的类，只需要符合两个条件即可，第一，包含名为`to_dict`方法；第二，`__init__`方法接受关键字参数。
有了这样的mix-in之后，只需编写极少的代码，就可以通过继承体系轻松创建出相关的工具类，以便实现序列化数据以及从*JSON*中读取数据的功能。
例如，我们用下面这个继承了mix-in组件的数据类来表示数据中心的拓扑结构：
```python
class DatacenterRack(ToDictMixin, JsonMixin):
    def __init__(self, switch=None, machines=None):
        self.switch = Switch(**switch)
        self.machines = [ Machine(**kwargs) for kwargs in machines]

class Switch(ToDictMixin, JsonMixin):
    # ···

class Machine(ToDictMixin, JsonMixin):
    # ···


# 对上面这样的类进行序列化，以及从JSON中加载它都是比较简单的。
# 下面的代码，会重复执行序列化及反序列化操作，以验证这两个功能有没有正确地实现出来。
serialized = """{
    "switch": {"ports": 5, "speed": 1e9},
    "machines": [
        {"cores": 8, "ram": 32e9, "disk": 5e12},
        {"cores": 4, "ram": 16e9, "disk": 1e12},
        {"cores": 2, "ram": 4e9, "disk": 500e9}
     ]
   }"""

deserialized = DatacenterRack.from_json(serialized)
roundtrip = deserialized.to_json()
assert json.loads(serialized) == json.loads(roundtrip)
```

#### 要点
*  能用mixin组件实现的效果，就不要用多重继承来做
*  将各功能实现为可插拔的mix-in组件，然后令相关的类继承自己需要的那些组件，即可定制该类实例所应具备的行为
*  把简单的行为封装到mix-in组件里，然后就可以用多个mix-in组合出复杂的行为了

###  第27条： 多用public属性，少用private属性
Python类的属性可见度只有两种：*public*和*private*
任何人都可以在对象上过`.`操作符来访问*public*属性。
以两个下划线(`__`)开头的属性，是*private*字段。只有本类的方法可以直接访问他们。子类也无法访问父类的私有属性。
实际上，Python会对私有属性的名称做如下的简单变换来保证*private*字段的私密性。

> `__private_field`会被变换为`_ClassName__private_field`

了解了这套变换机制之后，就可以从任意类中访问相关类的私有属性了，但是不建议这么做。
关于访问受限的属性，最好遵循PEP的建议使用一种管用的命名方式来表示这种字段。即：以单下划线开头的字段，视为*protected*字段，本类之外的代码在使用这种字段时要多加小心。


#### 要点
*  Python编译器无法严格保证private字段的私密性
*  不要盲目地将属性设为private，而是应该从一开始就做好规划，并允许子类更多地访问超类的内部API
*  应该多用protected属性，并在文档中把这些字段的合理用法告诉子类的开发者，而不要试图用private属性来限制子类访问这些字段
*  只有当子类不受自己控制时，才可以考虑用private属性来避免名称冲突

###  第28条： 继承collections.abc以实现自定义的容器类型
Python中的每一个类，从某种程度上来说都是容器，他们都封装了属性与功能。Python也直接提供了一些管理数据所用的内置容器类型，
例如，*list(列表)*，*tuple(元组)*，*set(集)*，*dictionary(字典)*等。
可以通过继承Python内置的容器类型来定制自己的容器，例如，要创建一种自定义的列表类型并提供统计每个元素出现频率的方法：
```python
# 继承了list，并获得了由list提供的全部标准功能。还可以添加自定义的方法以定制期望的行为。
class FrequencyList(list):
    def __init__(self, members):
        super().__init__(members)

    def frequency(self):
        counts = {}
        for item in self:
            counts.setdefault(item, 0)
            counts[item] += 1
        return counts

foo = FrequencyList(['a', 'b', 'a', 'c', 'b', 'a', 'd'])
print('Length is', len(foo))
foo.pop()
print('After pop:', repr(foo))
print('Frequency:', foo.frequency())

>>
Length is 7
After pop: ['a', 'b', 'a', 'c', 'b', 'a']
Frequency: {'a': 3, 'b': 2, 'c': 1}
```

现在，假设要编写这么一种对象：它本身虽然不属于*list*子类，但是用起来却和*list*一样，也可以通过下标访问其中的元素。例如，我们要令下面的二叉树节点类，也能够像*list*或*tuple*等序列那样来访问。
```python
class BinaryNode(object):
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right
```
这个类怎么才能表现的和序列类型一样呢？我们可以通过特殊方法完成此功能。Python会给一些名称比较特殊的实例方法，来实现与容器有关的行为。用下标访问序列中的元素是：`bar = [1, 2, 3]; bar[0]`Python会把访问代码转译为：`bar.__getitem__(0)`,于是，我们提供自己定制的`__getitem__`方法，即可令`BinaryNode`类可以表现得和序列一样。下面是按深度优先的次序来访问二叉树中的对象：
```python
class IndexableNode(BinaryNode):
    def _search(self, count, index):
        found = None
        if self.left:
            found, count = self.left._search(count, index)
        if not found and count == index:
            found = self
        else:
            count += 1
        if not found and self.right:
            found, count = self.right._search(count, index)
        return found, count
        # Returns (found, count)

    def __getitem__(self, index):
        found, _ = self._search(0, index)
        if not found:
            raise IndexError('Index out of range')
        return found.value

# 和往常一样构建二叉树
tree = IndexableNode(
    10,
    left=IndexableNode(
        5,
        left=IndexableNode(2),
        right=IndexableNode(
            6, right=IndexableNode(7))),
    right=IndexableNode(
        15, left=IndexableNode(11)))

# 除了可以向普通的二叉树那样进行遍历之外，还可以使用与list相同的写法来访问树中的元素
print('LRR =', tree.left.right.right.value)
print('Index 0 =', tree[0])
print('Index 1 =', tree[1])
print('11 in the tree?', 11 in tree)
print('17 in the tree?', 17 in tree)
print('Tree is', list(tree))

>>>
LRR = 7
Index 0 = 2
Index 1 = 5
11 in the tree? True
17 in the tree? False
Tree is [2, 5, 6, 7, 10, 11, 15]
```
然而只实现`__getitem__`方法是不够的，它并不能使该类型支持我们想要的每一种序列操作。
必须实现其他很多*list*提供的类似`__getitem__`的特殊方法才能达到目的，不过很不容易。

为了避免这些麻烦，可以使用内置的`collections.abc`模块。该模块定义了一系列抽象基类，他们提供了每一种容器类型所应具备的常用方法。
从这样的基类中继承了子类之后，如果忘记实现某个方法，那么`collections.abc`模块就会指出这个错误。
如果子类实现了抽象基类所要求的每个方法，那么基类就会自动提供剩下的那些方法。
```python
class SequenceNode(IndexableNode):# 自动具备index,count等方法。
    def __len__(self):
        _, count = self._search(0, None)
        return count

from collections.abc import Sequence
class BetterNode(SequenceNode, Sequence):
    pass

tree = BetterNode(
    10,
    left=BetterNode(
        5,
        left=BetterNode(2),
        right=BetterNode(
            6, right=BetterNode(7))),
    right=BetterNode(
        15, left=BetterNode(11))
)

print('Index of 7 is', tree.index(7))
print('Count of 10 is', tree.count(10))

>>>
Index of 7 is 3
Count of 10 is 1
```

#### 要点
*  如果要定制的子类比较简单，那就可以直接从Python的容器类型(如list或dict)中继承
*  想正确实现自定义的容器类型，可能需要编写大量的特殊方法
*  编写自制的容器类型时，可以从collections.abc模块的抽象基类中继承，那些基类能够确保我们的子类具备适当的借口及行为


##   第四章 元类及属性
*metaclass*元类，是一种可以定制其他类的类，又称为*描述类*。简单来说，就是我们可以把Python的*class*语句转译为元类，并令其在每次定义具体的类时，都能提供独特的行为。
Python还有一个奇妙的特性，就是可以动态地定义对属性的访问操作。把这种动态属性机制与Python的面向对象机制相结合，就可以非常顺利地将简单的类逐渐变换为复杂的类。
但是，这些强大的功能也有弊端。动态属性可能会覆盖对象的某些行为从而产生令人意外的副作用。元类也可能会创建出极其古怪的程序，使得刚接触Python的程序员无所适从。使用这些特性是，要遵循*最小惊讶原则(rule of least surprise)*,也就是说，这些机制只适合用来实现那些广为人知的Python编程范式。

###  第29条： 用纯属性取代get和set方法
对于Python语言，基本上不需要手工实现`setter`或`getter`方法，应该先从简单的*public*开始写起。
```python
class Resistor(object):
    def __init__(self, ohms):
        self.ohms = ohms
        self.voltage = 0
        self.current = 0

r1 = Resistor(50e3)
r1.ohms = 10e3
print('%r ohms, %r volts, %r amps' %
      (r1.ohms, r1.voltage, r1.current))

>>>
10000.0 ohms, 0 volts, 0 amps
```
以后如果想要在设置属性的时候实现特殊行为，那么可以用`@property`修饰其和`setter`方法来做。下面这个类继承自`Resistor`，它在给*voltage*(电压)属性赋值的时候还会同时修改*current*(电流)属性。
注意：`setter`和`getter`方法的名称必须与相关属性相符，才能使这套机制正常运作。
```python
class VoltageResistance(Resistor):
    def __init__(self, ohms):
        super().__init__(ohms)
        self._voltage = 0

    @property
    def voltage(self):
        return self._voltage

    @voltage.setter
    def voltage(self, voltage):
        self._voltage = voltage
        self.current = self._voltage / self.ohms

r2 = VoltageResistance(1e3)
print('Before: %5r amps' % r2.current)
r2.voltage = 10
print('After:  %5r amps' % r2.current)

>>>
Before:     0 amps
After:   0.01 amps
```
为属性指定`setter`方法时，也可以在方法里做类型验证及数值验证。下面定义的类，可以保证传入的电阻值总是大于0欧姆：
```python
class BoundedResistance(Resistor):
    def __init__(self, ohms):
        super().__init__(ohms)

    @property
    def ohms(self):
        return self._ohms

    @ohms.setter
    def ohms(self, ohms):
        if ohms <= 0:
            raise ValueError('%f ohms must be > 0' % ohms)
        self._ohms = ohms
```
此时，传入无效值会引发异常。
```python
r3 = BoundedResistance(1e3)
r3.ohms = 0

>>>
ValueError: 0.000000 ohms must be > 0


BoundedResistance(-5)
>>>
ValueError: -5.000000 ohms must be > 0
# 此处之所以会引发异常，是因为BoundedResistance.__init__会调用Resistor.__init__，而Resistor.__init__会执行self.ohms = -5
# 这条赋值语句使得BoundedResistance中的@ohms.setter得到执行，于是引发异常。
```
甚至可以用`@property`来防止父类的属性遭到修改：
```python
class FixedResistance(Resistor):
    def __init__(self, ohms):
        super().__init__(ohms)

    @property
    def ohms(self):
        return self._ohms

    @ohms.setter
    def ohms(self, ohms):
        if hasattr(self, '_ohms'):
            raise AttributeError("Can't set attribute")
        self._ohms = ohms

r4 = FixedResistance(1e3)
r4.ohms = 2e3
>>>
AttributeError: Can't set attribute
```
`@property`的最大缺点在于:和属性相关的方法，只能在子类里面共享，而与之无关的其他类，则无法复用同一份实现代码。不过，Python也提供了描述符机制(参考第31条)，开发者可以通过它来复用与属性有关的逻辑。

#### 要点
*  编写新类时，应该用简单的public属性来定义其借口，而不要手工实现set和get方法
*  如果访问对象的某个属性时，需要表现出特殊的行为，那就用@property来定义这种行为
*  @property方法应该遵循最小惊讶原则，而不应该产生奇怪的副作用
*  @property方法需要执行的迅速一些，缓慢或复杂的工作应该放在普通的方法里面

###  第30条： 考虑用@property来代替属性重构
`@property`有一种高级用法是可以把简单的数值属性迁移为*实时计算(on-the-fly calculation,按需计算，动态计算)*的属性，这种用法比较常见。采用`@property`来迁移属性时，我们只需要给本类添加新的功能，原有的那些调用代码都不需要修改，因此，它是一种非常有效的编程手法。在持续完善借口的过程中，它是一种重要的缓冲方案。
例如，要用纯Python对象实现带有配额的漏桶。西面这段代码，把当前剩余的配额以及重置配额的周期，放在了`Bucket`类里面：
```python
import logging
from pprint import pprint
from sys import stdout as STDOUT
from datetime import datetime, timedelta

class Bucket(object):
    def __init__(self, period):
        self.period_delta = timedelta(seconds=period)
        self.reset_time = datetime.now()
        self.quota = 0

    def __repr__(self):
        return 'Bucket(quota=%d)' % self.quota

bucket = Bucket(60)
print(bucket)
>>>
Bucket(quota=0)
```
漏桶算法若要正常运作，就必须保证：无论向桶中加多少水，都必须在激怒下一个周期时将其清空。
```python
def fill(bucket, amount):
    now = datetime.now()
    if now - bucket.reset_time > bucket.period_delta:
        bucket.quota = 0
        bucket.reset_time = now
    bucket.quota += amount
```
每次在执行消耗配额的操作之前，都必须先确认桶里有足够的配额可供使用。
```python
def deduct(bucket, amount):
    now = datetime.now()
    if now - bucket.reset_time > bucket.period_delta:
        return False
    if bucket.quota - amount < 0:
        return False
    bucket.quota -= amount
    return True
```
使用方法：
```python
## 要先往桶里添水
bucket = Bucket(60)
fill(bucket, 100)
print(bucket)

>>>
Bucket(quota=60)

# 消耗自己的配额
if deduct(bucket, 99):
    print('Had 99 quota')
else:
    print('Not enough for 99 quota')
print(bucket)

>>>
Had 99 quota
Bucket(quota=1)

# 这样消耗下去，最终会导致待消耗的配额比剩余的配额还多，到了那时，Bucket对象就会阻止这一操作，而漏桶中剩余的配额则将保持不变。
if deduct(bucket, 3):
    print('Had 3 quota')
else:
    print('Not enough for 3 quota')
print(bucket)

>>>
Not enough for 3 quota
Bucket(quota=1)
```
这种实现方式的缺点是：以后无法得知漏桶的初始配额。配额会在每个周期内持续流失，如果降到0，那么deduct就总是返回False。此时，依赖deduct的那些操作就会受到阻塞，但是，我们却无法判断出这究竟是由于Bucket里面所剩的配额不足还是由于Bucket刚开始的时候就没有配额。
为了解决这一问题，我们在类中使用`max_quota`来记录本周期的初始配额，并且用`quota_consumed`来记录本周期内所消耗的配额。
```python
class Bucket(object):
    def __init__(self, period):
        self.period_delta = timedelta(seconds=period)
        self.reset_time = datetime.now()
        self.max_quota = 0
        self.quota_consumed = 0

    def __repr__(self):
        return ('Bucket(max_quota=%d, quota_consumed=%d)' %
                (self.max_quota, self.quota_consumed))
```
我们可以根据这两个新的属性，在`@property`方法里面实时地计算出当前所剩的配额。
```python
    @property
    def quota(self):
        return self.max_quota - self.quota_consumed
```
在设置`quota`属性的时候，`setter`方法应该采取一些措施，以保证`quota`能够与该类接口中的`fill`和`deduct`相匹配。
```python
    @quota.setter
    def quota(self, amount):
        delta = self.max_quota - amount
        if amount == 0:
            # Quota being reset for a new period
            self.quota_consumed = 0
            self.max_quota = 0
        elif delta < 0:
            # Quota being filled for the new period
            assert self.quota_consumed == 0
            self.max_quota = amount
        else:
            # Quota being consumed during the period
            assert self.max_quota >= self.quota_consumed
            self.quota_consumed += delta
```
运行测试代码，可以产生与刚才那种实现方案相同的结果。
这种写法最好的地方就在于：从前使用`Bucket.quota`的旧代码，既不需要做出修改，也不需要担心现在的`Bucket`类是如何实现的。而将来要使用`Bucket`的新代码，则可以直接访问`max_quota`和`quota_consumed`,以执行正确的操作。
你可能在想：为什么一开始不把`fill`和`deduct`直接设计成`Bucket`类的实力方法，而要把他们放在`Bucket`外面呢？这样想确实有道理(第22条)，但是实际工作中，我们经常接触到的对象，恰恰是这种借口设计的比较糟糕或仅仅能够充当数据容器的对象。总结就是前期设计问题。。
另外，如果你发现自己正在不停地编写各种`@property`方法，恐怕就意味着当前这个类的代码写的确实很差，此时应该考虑彻底重构该类，而不是继续缝缝补补这套糟糕的设计。

#### 要点
*  @property可以为现有的实例属性添加新的功能
*  可以用@property来逐步完善数据模型
*  如果@property用的太过频繁，那就应该考虑彻底重构该类并修改相关的调用代码

###  第31条： 用描述符来改写需要复用的@property方法
`@property`修饰器有个明显的缺点，就是不便于服用。受它修饰的这些方法，无法为同一个类中的其他属性所服用，而且与之无关的类也无法复用这些方法。
例如，要编写一个类来验证学生的家庭作业成绩都处在0-100.
```python
class Homework(object):
    def __init__(self):
        self._grade = 0

    @property
    def grade(self):
        return self._grade

    @grade.setter
    def grade(self, value):
        if not (0 <= value <= 100):
            raise ValueError('Grade must be between 0 and 100')
        self._grade = value

#由于有@property，这个类用起来非常简单
galileo = Homework()
galileo.grade = 95
print(galileo.grade)
>>>
95
```
加入，现在要把这个套验证逻辑放在考试成绩上面，而考试成绩又是由多个科目的小成绩组成的，每一科都要单独记分。
```python
class Exam(object):
    def __init__(self):
        self._writing_grade = 0
        self._math_grade = 0

    @staticmethod
    def _check_grade(value):
        if not (0 <= value <= 100):
            raise ValueError('Grade must be between 0 and 100')
    @property
    def writing_grade(self):
        return self._writing_grade

    @writing_grade.setter
    def writing_grade(self, value):
        self._check_grade(value)
        self._writing_grade = value

    # 下面会是非常枯燥的代码，因为每添加一项科目，
    # 就要重复编写一次@property方法，而且还要把相关验证逻辑也重做一遍。
```

Python会对访问操作进行一定的转译，而这种转译方式则是由*描述符(descriptor)*协议来确定的。描述符类可以提供`__get__`和`__set__`方法，使得开发者无需编写例行(重复)代码即可复用一些功能。由于描述符能够把同一套逻辑运用在类中的不同属性上面，所以从这个角度来看，描述符也要比*mix-in*好一些(第26条)。
```python
# 实现了描述符协议的Grade类
class Grade(object):
    def __get__(*args, **kwargs):
        pass

    def __set__(*args, **kwargs):
        pass

# Exam类将几个Grade实例用作自己的类属性
class Exam(object):
    # Class attributes
    math_grade = Grade()
    writing_grade = Grade()
    science_grade = Grade()

# 赋值
exam = Exam()
exam.writing_grade = 40
```
上述代码最后为属性赋值时，Python会将代码转译为：`Exam.__dict__['writing_grade'].__set__(exam, 40)`; 
而获取属性`print(exam.writing_grade)`时,Python也会将其转译为：`print(Exam.__dict__['writing_grade'].__get__(exam, Exam))`
之所以会有这样的转译，关键在于`object`类的`__getattribute__`方法(参考第32条)。简单来说，如果`Exam`实例没有名为`writing_grade`的属性，那么Python就会转向`Exam`类，并在该类中查找同名的类属性。这个类属性，如果是实现了`__get__`和`__set__`方法的对象，那么Python就认为此对象遵从描述符协议。
明白了这种转译方式之后，我们勀先按照下面这种写法，试着把`Homework`类里的`@property`分数验证逻辑，改用`Grade`描述符来实现。
```python
class Grade(object):
    def __init__(self):
        self._value = 0

    def __get__(self, instance, instance_type):
        return self._value

    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError('Grade must be between 0 and 100')
        self._value = value

class Exam(object):
    math_grade = Grade()
    writing_grade = Grade()
    science_grade = Grade()
```
不幸的是，上面这种实现方式是错误的，它会导致不符合预期的行为。在同一`Exam`实例上面多次操作器属性是，尚且看不出错误。
```python
first_exam = Exam()
first_exam.writing_grade = 82
first_exam.science_grade = 99
print('Writing', first_exam.writing_grade)
print('Science', first_exam.science_grade)
>>>
Writing 82
Science 99
```
但是如果在多个`Exam`实例上分别操作某一属性，就会导致错误的结果。
```python
second_exam = Exam()
second_exam.writing_grade = 75
print('Second', second_exam.writing_grade, 'is right')
print('First ', first_exam.writing_grade, 'is wrong')
>>>
Second 75 is right
First  75 is wrong
```
产生这种问题的原因是，对于`writing_grade`这个类属性来说，所有的`Exam`实例都共享宫一份`Grade`实例。而表示该属性的那个`Grade`实例，只会在程序的生命期中构建一次，也就是说：当程序定义`Exam`类的时候，它会把`Grade`实例构建好，以后创建`Exam`实例时就不再构建`Grade`了。

为了解决此问题，我们需要把每个`Exam`实例所对应的值记录到`Grade`中。下面这段代码，用字典来保存每个实例的状态。
```python
class Grade(object):
    def __init__(self):
        self._values = {}

    def __get__(self, instance, instance_type):
        if instance is None: return self
        return self._values.get(instance, 0)

    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError('Grade must be between 0 and 100')
        self._values[instance] = value
```
这种实现方式很简单而且能够正确运作，但它依然有个问题，那就是会泄漏内存。在程序的生命期内，对于传给`__set__`方法的每个`Exam`实例来说，`_values`字典都会保存指向该实例的一份引用。这就导致该实例的引用计数无法降为0，从而使垃圾收集器无法将其回收。

使用Python内置的`weakref`模块，即可解决此问题。该模块提供了名为*WeakKey-Dictionary*的特殊字典，它可以取代`_values`原来所用的普通字典。*WeakKeyDictionary*的特殊之处在于：如果运行期系统发现这种字典所持有的引用时整个程序里指向`Exam`实例的最后一份引用，那么，系统就会自动将该实例从字典的键中移除。Python会做好相关的维护工作，以保证当程序不再使用任何`Exam`实例时，`_values`字典会是空的。
```python
from weakref import WeakKeyDictionary

class Grade(object):
    def __init__(self):
        self._values = WeakKeyDictionary()
    def __get__(self, instance, instance_type):
        if instance is None: return self
        return self._values.get(instance, 0)

    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError('Grade must be between 0 and 100')
        self._values[instance] = value

class Exam(object):
    math_grade = Grade()
    writing_grade = Grade()
    science_grade = Grade()

first_exam = Exam()
first_exam.writing_grade = 82
second_exam = Exam()
second_exam.writing_grade = 75
print('First ', first_exam.writing_grade, 'is right')
print('Second', second_exam.writing_grade, 'is right')

>>>
First   82 is right
Second  75 is right
```
改用*WeakKeyDictionary*来实现`Grade`描述符，即可令程序的行为符合我们的需求。

#### 要点
*  如果想复用@property方法及其验证机制，那么可以自己定义描述符类
*  WeakKeyDictionary可以保证描述符类不会泄漏内存
*  通过描述符协议来实现属性的获取和设置操作时，不要纠结于__getattribute__的方法具体运行细节

###  第32条： 用__getattr__, getattribute__, 和__setattr__实现按需生成的属性
Python语言提供了一些挂钩，使得开发者很容易能编写出通用的代码，以便将多个系统粘合起来。
例如，要用Python对象表示数据库表的一行，数据库有自己的数据结构，是哦一在操作与行对应的对象时必须知道这个数据库的结构。然而，把Python对象与数据库连接的代码却不需要知道行的结构，所以这部分代码应该写的通用一些。
如何实现呢？普通的实例属性，`@property`方法和描述符，都不能够完成此功能，因为它们都必须原先定义好。而像这样的动态行为，则可以通过Python的`__getattr`特殊放啊来做。如果某个个类定义了`__getattr__`,同时系统在该类对象的实例字典中又找不到待查询的属性，那么，系统就会调用这个方法。
```python
class LazyDB(object):
    def __init__(self):
        self.exists = 5

    def __getattr__(self, name):
        value = 'Value for %s' % name
        setattr(self, name, value)
        return value

data = LazyDB()
print('Before:', data.__dict__)
# 访问对象所没有的foo属性，会导致Python调用类中定义的__getattr__方法，从而修改实例的__dict__字典
print('foo:   ', data.foo)
print('After: ', data.__dict__)
>>>
Before: {'exists': 5}
foo:    Value for foo
After:  {'exists': 5, 'foo': 'Value for foo'}
```
然后，给`LazyDB`添加记录功能，把程序对`__getattr__`的调用行为记录下来。请注意，为了避免无线递归，我们需要在`LoggingLazyDB`子类里面通过`super().__getattr__()`来获取真正的属性值。
```python
class LoggingLazyDB(LazyDB):
    def __getattr__(self, name):
        print('Called __getattr__(%s)' % name)
        return super().__getattr__(name)

data = LoggingLazyDB()
print('exists:', data.exists)
print('foo:   ', data.foo)
print('foo:   ', data.foo)
>>>
exists: 5
Called __getattr__(foo)
foo:    Value for foo
foo:    Value for foo
```
由于`exists`属性本身就在实例字典里面，所以访问它的时候，不会触发`__getattr__`。而`foo`属性刚开始并不在实例字典里，所以初次访问的时候会触发`__getattr__`。由于`__getattr__`又会调用`__setattr__`方法并把`foo`放在实例字典中，所以第二次访问`foo`时就不会触发`__getattr__`了。
这种行为非常适合实现*无结构数据(schemaless data)*的按需访问。初次执行`__getattr__`的时候进行一些操作，把相关的属性加载进来，以后再访问该属性时，只需从现有的结果之中获取即可。
现在假设我们还要在数据库系统中实现*事务(transaction)*处理。用户下次访问某属性是，我们要知道数据库中对应的行是否依然有效，以及相关事务是否依然处于开启状态。这样的需求，无法通过`__getattr__`挂钩可靠地实现出来，因为Python系统会直接从实例字典的现存属性中迅速查出该属性并返回给调用者。
为了实现此功能，我们可以使用Python中的另外一个挂钩，也就是`__getattribute__`。程序每次访问对象的属性时，Python系统都会调用这个特殊方法，即使属性字典里面已经有了该属性，也依然会触发`__getattribute__`方法。这样就可以在程序每次访问属性时，价差全局事务状态。下面定义的这个`ValidatingDB`类，会在`__getattribute__`方法里面记录每次调用的时间。
```python
class ValidatingDB(object):
    def __init__(self):
        self.exists = 5

    def __getattribute__(self, name):
        print('Called __getattribute__(%s)' % name)
        try:
            return super().__getattribute__(name)
        except AttributeError:
            value = 'Value for %s' % name
            setattr(self, name, value)
            return value

data = ValidatingDB()
print('exists:', data.exists)
print('foo:   ', data.foo)
print('foo:   ', data.foo)
>>>
Called __getattribute__(exists)
exists: 5
Called __getattribute__(foo)
foo:    Value for foo
Called __getattribute__(foo)
foo:    Value for foo
```
按照Python处理确实属性的标准流程，如果程序动态地访问了一个不应该有的属性，那么可以在`__getattr__`和`__getattribute__`里面抛出`AttributeError`异常。
```python
class MissingPropertyDB(object):
    def __getattr__(self, name):
        if name == 'bad_name':
            raise AttributeError('%s is missing' % name)
        value = 'Value for %s' % name
        setattr(self, name, value)
        return value

data = MissingPropertyDB()
data.foo  # Test this works
data.bad_name
>>>
AttributeError: bad_name is missing
```
实现这种通用的功能是，我们通常会在Python代码里使用内置的`hasattr`函数来判断对象是否已经拥有了该属性，并用内置的`getattr`函数来获取属性值。这些函数会现在实例字典中搜索待查询的属性，然后在调用`__getattr__`。
```python
data = LoggingLazyDB()
print('Before:     ', data.__dict__)
print('foo exists: ', hasattr(data, 'foo'))
print('After:      ', data.__dict__)
print('foo exists: ', hasattr(data, 'foo'))
>>>
Before:      {'exists': 5}
Called __getattr__(foo)
foo exists:  True
After:       {'exists': 5, 'foo': 'Value for foo'}
foo exists:  True
```
上例中，`__getattr__`方法只调用了一次。反之，如果本类实现的是`__getattribute__`方法，那么每次在对象上面调用`hasattr`或`getattr`函数时，此方法都会执行。
```python
data = ValidatingDB()
print('foo exists: ', hasattr(data, 'foo'))
print('foo exists: ', hasattr(data, 'foo'))
>>>
Called __getattribute__(foo)
foo exists:  True
Called __getattribute__(foo)
foo exists:  True
```
现在，假设当程序把值赋给Python对象之后，我们要以惰性的方式将其推回数据库。此功能可以用Python所提供的`__setattr__`挂钩来实现，它与前面所讲的那两个挂钩类似，可以拦截对属性的赋值操作。但是与`__getattr__`和`__getattribute__`不同的地方在于，我们不需要恩成两个方法来处理。只要对实例的属性赋值，无论是直接赋值还是通过内置的`setattr`函数赋值，都会触发`__setattr__`方法。
```python
class SavingDB(object):
    def __setattr__(self, name, value):
        # Save some data to the DB log
        super().__setattr__(name, value)


class LoggingSavingDB(SavingDB):
    def __setattr__(self, name, value):
        print('Called __setattr__(%s, %r)' % (name, value))
        super().__setattr__(name, value)

data = LoggingSavingDB()
print('Before: ', data.__dict__)
data.foo = 5
print('After:  ', data.__dict__)
data.foo = 7
print('Finally:', data.__dict__)
>>>
Before:  {}
Called __setattr__(foo, 5)
After:   {'foo': 5}
Called __setattr__(foo, 7)
Finally: {'foo': 7}
```
使用`__getattribute__`和`__setattr__`挂钩方法时要注意：每次访问对象属性是，他们都会触发，可这可能并不是你想要的效果。例如，我们想在查询对象的属性时，从对象内部的一份字典里面搜寻与待查找属性相关联的属性值。
```python
class BrokenDictionaryDB(object):
    def __init__(self, data):
        self._data = data

    def __getattribute__(self, name):
        print('Called __getattribute__(%s)' % name)
        return self._data[name]

data = BrokenDictionaryDB({'foo': 3})
data.foo
>>>
Called __getattribute__(_data)
Called __getattribute__(_data)
Called __getattribute__(_data)
# 此处陷入死循环，突破最大的栈深度并崩溃
```
此处的问题在于，`__getattribute__`会访问`self._data`,而这意味着需要再次调用`__getattribute__`,然后它又会调用`self._data`，开始无限循环。解决办法是，采用`super().__getattribute__`方法，从实例的属性字典里面直接获取`_data`属性值，以避免无线递归。
```python
class DictionaryDB(object):
    def __init__(self, data):
        self._data = data

    def __getattribute__(self, name):
        data_dict = super().__getattribute__('_data')
        return data_dict[name]

data = DictionaryDB({'foo': 3})
print(data.foo)
>>>
3
```
与之类似，如果要在`__setattr__`方法中修改对象的属性，也要通过`super().__setattr__`来完成。

#### 要点
*  通过__getattr__和__setattr__,我们可以用惰性的方式来加载并保存对象的属性
*  要理解__getattr__和__getattribute__的区别：前者只会在待访问的属性缺失时触发，而后者则会在每次访问属性时触发
*  如果要在__getattribute__和__setattr__方法中访问实例属性，那么应该直接通过super()(也就是object类的同名方法)来做，以避免无线递归

###  第33条： 用元类来验证子类
元类最简单的一种用途，就是验证某个类定义得是否正确。构建复杂的类体系时，我们可能需要确保类的更个协调一致，确保某些方法得到了覆写，或是确保类属性之间具备某些秧歌的关系。元类提供了一种可靠的验证方式，每当开发者定义新的类时，它都会运行验证代码，以确保这个新类符合预定的规范。
开发者一般会把验证代码放在本类的`__init__`方法里，这是由于程序在构建该类的对象时(第28条)，会调用本类型的`__init__`方法。但如果改用元类进行验证，我们还可以把验证时机再提前一些，以便尽早发现错误。
定义元类的时候，要从`type`中继承，而对于使用该元类的其他类来说，Python默认会把那些类的`class`语句体中所含的相关内容，发送给元类的`__new__`方法。于是，我们就可以在系统构建出那种类型之前，先修改那个类的信息。
```python
from pprint import pprint
class Meta(type):
    def __new__(meta, name, bases, class_dict):
        orig_print = __builtins__.print
        print = pprint
        print((meta, name, bases, class_dict))
        print = orig_print
        return type.__new__(meta, name, bases, class_dict)

class MyClass(object, metaclass=Meta):
    stuff = 123

    def foo(self):
        pass

## 元类可以获知那个类的名称，其所继承的父类，以及定义在class语句体中的全部类属性。
>>>
(<class '__main__.Meta'>,
 'MyClass',
 (<class 'object'>,),
 {'__module__': '__main__',
  '__qualname__': 'MyClass',
  'foo': <function MyClass.foo at 0x000001F331761AE8>,
  'stuff': 123})
```
Python2的写法稍有不同，它是通过名为`__metaclass__`的类属性来制定元类的。而`Meta.__new__`接口则一致。
```python
from pprint import pprint
class Meta(type):
    def __new__(meta, name, bases, class_dict):
        print(meta, name, bases, class_dict)
        return type.__new__(meta, name, bases, class_dict)

class MyClassInPython2(object):
    __metaclass__ = Meta
    stuff = 123

    def foo(self):
        pass
```
为了在定义某个类的时候，确保该类的所有参数都有效，我们可以把相关的验证逻辑添加到`Meta.__new__`方法中。例如，要用类来表示任意多边形。为此，我们可以定义一种特殊的验证类，使得多边形体系中的基类，把这个验证类当成自己的元类。要注意的是，元类中所编写的验证逻辑，针对的是该基类的子类，而不是基类本身。
```python
class ValidatePolygon(type):
    def __new__(meta, name, bases, class_dict):
        # Don't validate the abstract Polygon class
        if bases != (object,):
            if class_dict['sides'] < 3:
                raise ValueError('Polygons need 3+ sides')
        return type.__new__(meta, name, bases, class_dict)

class Polygon(object, metaclass=ValidatePolygon):
    sides = None  # Specified by subclasses

    @classmethod
    def interior_angles(cls):
        return (cls.sides - 2) * 180

class Triangle(Polygon):
    sides = 3

print(Triangle.interior_angles())
>>>
180
```
假如我们尝试定义一种边数少于3的多边形子类，那么class语句体刚一结束，元类中的验证代码立刻就会拒绝这个class。也就是说，如果开发者定义这样一种子类，那么程序根本就无法运行。
```python
print('Before class')
class Line(Polygon):
    print('Before sides')
    sides = 1
    print('After sides')
print('After class')
>>>
Before class
Before sides
After sides
ValueError: Polygons need 3+ sides
```

#### 要点
*  通过元类，我们可以在生成子类对象之前，先验证子类的定义是否合乎规范
*  Python2和Python3指定元类的语法略有不同
*  Python系统把子类的整个class语句体处理完毕之后，就会调用其元类的__new__方法

###  第34条： 用元类来注册子类
元类还有一个用途，就是在程序中自动注册类型。对于需要*反向查找(reverse lookup)*的场合，这种注册操作时很有用的，它使我们可以在简单的标识符与对应的类之间，建立映射关系。
例如，我们想按照自己的实现方式，将Python对象表示为*JSON*格式的序列化数据，那么，就需要用一种手段，把指定的对象转换成*JSON*字符串。下面这段代码，定义了一个通用的基类，它可以记录程序调用本类构造器时所用的参数，并将其转换为*JSON*字典：
```python
import json

class Serializable(object):
    def __init__(self, *args):
        self.args = args

    def serialize(self):
        return json.dumps({'args': self.args})
```
有了这个类，就可以把一些简单且不可变的数据结构，轻松地转换成字符串了。例如，下面这个`Point2D`类，很容易就能转为字符串。
```python
class Point2D(Serializable):
    def __init__(self, x, y):
        super().__init__(x, y)
        self.x = x
        self.y = y

    def __repr__(self):
        return 'Point2D(%d, %d)' % (self.x, self.y)

point = Point2D(5, 3)
print('Object:    ', point)
print('Serialized:', point.serialize())
>>>
Object:     Point2D(5, 3)
Serialized: {"args": [5, 3]}
```
现在，我们需要对这种*JSON*字符串执行*反序列化(deserialize)*操作，并构建出该字符串所表示的`Point2D`对象。下面定义的这个`Deserializable`类，继承自`Serializable`,它可以把`Serializable`所产生的*JSON*字符串还原为Python对象：
```python
class Deserializable(Serializable):
    @classmethod
    def deserialize(cls, json_data):
        params = json.loads(json_data)
        return cls(*params['args'])
```
有了`Deserializable`,我们就可以用一种通用的方式，对简单且不可变的对象执行序列化和反序列化操作。
```python
class BetterPoint2D(Deserializable):
    def __init__(self, x, y):
        super().__init__(x, y)
        self.x = x
        self.y = y

    def __repr__(self):
        return 'BetterPoint2D(%d, %d)' % (self.x, self.y)

point = BetterPoint2D(5, 3)
print('Before:    ', point)
data = point.serialize()
print('Serialized:', data)
after = BetterPoint2D.deserialize(data)
print('After:     ', after)
>>>
Before:     BetterPoint2D(5, 3)
Serialized: {"args": [5, 3]}
After:      BetterPoint2D(5, 3)
```
这种方案的缺点是，我们必须提前知道序列化的数据是什么类型(例如，是`Point2D`或`BetterPoint2D`等)，然后才能对其做反序列化操作。
<br>理想的方案应该是：有很多类都可以把本类对象转换为*JSON*格式的序列化字符串，但是只需要一个公用的反序列化函数，就可以将任意的*JSON*字符串还原成相应的Python对象。
为此，我们可以把序列化对象的类名写道*JSON*数据里面。
```python
class BetterSerializable(object):
    def __init__(self, *args):
        self.args = args

    def serialize(self):
        return json.dumps({
            'class': self.__class__.__name__,
            'args': self.args,
        })

    def __repr__(self):
        return '%s(%s)' % (
            self.__class__.__name__,
            ', '.join(str(x) for x in self.args))
```
然后，把类名与该类对象构造器之间的映射关系，维护到一份字典里面。这样凡是经由`register_class`注册的类，就都可以拿通用的`deserialize`函数做反序列化操作。
```python
registry = {}

def register_class(target_class):
    registry[target_class.__name__] = target_class

def deserialize(data):
    params = json.loads(data)
    name = params['class']
    target_class = registry[name]
    return target_class(*params['args'])
```
为了确保`deserialize`函数正常运作，我们必须用`register_class`把将来可能要执行反序列化操作的那些类，都注册一遍。
```python
class EvenBetterPoint2D(BetterSerializable):
    def __init__(self, x, y):
        super().__init__(x, y)
        self.x = x
        self.y = y

register_class(EvenBetterPoint2D)
```
接下来，就可以对任意的*JSON*字符串执行反序列化操作了，执行操作时，我们不需要知道该字符串表示的是哪种类型的数据。
```python
point = EvenBetterPoint2D(5, 3)
print('Before:    ', point)
data = point.serialize()
print('Serialized:', data)
after = deserialize(data)
print('After:     ', after)
>>>
Before:     EvenBetterPoint2D(5, 3)
Serialized: {"class": "EvenBetterPoint2D", "args": [5, 3]}
After:      EvenBetterPoint2D(5, 3)
```
这种方案也有缺点，那就是开发者可能会忘记调用`register_class`函数。
```python
class Point3D(BetterSerializable):
    def __init__(self, x, y, z):
        super().__init__(x, y, z)
        self.x = x
        self.y = y
        self.z = z

# 没有调用 register_class函数
point = Point3D(5, 9, -4)
data = point.serialize()
deserialize(data)
>>>
KeyError: 'Point3D'
```
如果写完`class`语句体之后，忘记调用`register_class`,那么即使从`BetterSerializable`中继承了子类，也依然无法利用`deserialize`函数对其执行反序列化操作。所以，这种写法很容易出错，而且对于编程新手尤其危险。
在Python3中使用类修饰器(class decorator)时，也会出现同样的问题。
我们应该想个办法，保证开发者在继承`BetterSerializable`的时候，程序会自动调用`register_class`函数，并将新的子类注册好。这个功能可以通过元类来实现。定义玩子类的`class`语句体之后，元类可以拦截这个新的子类(第33条)。于是，我们就能够在子类的`class`语句体得到处理之后，立刻注册这一新的类型。
```python
class Meta(type):
    def __new__(meta, name, bases, class_dict):
        cls = type.__new__(meta, name, bases, class_dict)
        register_class(cls)
        return cls

class RegisteredSerializable(BetterSerializable, metaclass=Meta):
    pass
```
现在，定义完`RegisteredSerializable`的子类之后，开发者可以确信：该类肯定已经通过`register_class`函数注册好了，于是`deserialize`函数也就可以正常运作了。
```python
class Vector3D(RegisteredSerializable):
    def __init__(self, x, y, z):
        super().__init__(x, y, z)
        self.x, self.y, self.z = x, y, z

v3 = Vector3D(10, -7, 3)
print('Before:    ', v3)
data = v3.serialize()
print('Serialized:', data)
print('After:     ', deserialize(data))
>>>
Before:     Vector3D(10, -7, 3)
Serialized: {"class": "Vector3D", "args": [10, -7, 3]}
After:      Vector3D(10, -7, 3)
```
只要类的继承体系正确无误，我们就可以用元类来实现类的注册，以确保每一个子类都不会遗漏。通过刚才的范例可以看出：这种方案，适用于序列化和反序列化操作。此外，它还适用于数据库的*对象关系映射(object-relationship mapping,ORM)*，插件系统和系统挂钩。

#### 要点
*  在构建模块化的Python程序时，类的注册时一种很有用的模式
*  开发者每次从基类中继承子类时，基类的元类都可以自动运行注册代码
*  通过元类来实现类的注册，可以确保所有子类都不会遗漏，从而避免后续的错误

###  第35条： 用元类来注解类的属性
元类还有一个更有用的功能，那就是可以在某个类刚定义好但是尚未使用的时候，提前修改或注解该类的属性。这种写法通常会与*描述符(descriptor)*搭配起来(第31条)，令这些属性可以更加详细地了解自己在外围类中的使用方式。
例如，要定义新的类，用来表示客户数据库里的某一行。同时，我们还希望在该类的相关属性与数据库表的每一列之间，建立对应关系。于是，用下面这个描述符类，把属性与列名联系起来。
```python
class Field(object):
    def __init__(self, name):
        self.name = name
        self.internal_name = '_' + self.name

    def __get__(self, instance, instance_type):
        if instance is None: return self
        return getattr(instance, self.internal_name, '')

    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)
```
由于列的名称已经保存到了`Field`描述符中，所以我们可以通过内置的e`setattr`和`getattr`函数，把每个实例的所有状态都作为`protected`字段，存放在该实例的字典里面。在本书前面的例子中，为了避免内存泄漏，我们曾经用`weakref`字典来构建描述符，而刚才的代码，目前看来，似乎要比`weakref`方案便捷得多。
接下来定义表示数据行为的`Customer`类，定义该类的时候，我们要为每个类属性指定对应的列名。
```python
class Customer(object):
    # Class attributes
    first_name = Field('first_name')
    last_name = Field('last_name')
    prefix = Field('prefix')
    suffix = Field('suffix')
```
`Customer`类用起来比较简单。通过下面这段演示代码可以看出，`Field`描述符能够按照预期，修改`__dict__`实例字典：
```python
foo = Customer()
print('Before:', repr(foo.first_name), foo.__dict__)
foo.first_name = 'Euclid'
print('After: ', repr(foo.first_name), foo.__dict__)
>>>
Before: '' {}
After:  'Euclid' {'_first_name': 'Euclid'}
```
问题在于，上面这种那个写法显得有些重复。在`Customer`类的`class`语句体中，我们既然要将构建好的`Field`对象赋给`Customer.first_name`,那为什么还要把这个字段名(`first_name`)再传给`Field`的构造器呢？
之所以还要把字段名传给`Field`构造器，是因为定义`Customer`类的时候，Python会议从右向左的顺序解读赋值语句，这与从左至右的阅读顺序恰好相反。首先，Python会以`Field('first_name')`的形式来调用`Field`构造器，然后，它把调用构造器所得的返回值，赋给`Customer.field_name`。从这个顺序来看，`Field`对象没有办法提前知道自己会赋给`Customer`类里的哪一个属性。
<br>为了消除这种重复代码，我们现在用元类来改写它。使用元类，就相当于直接在`class`语句上放置挂钩，只要`class`语句体处理完毕，这个挂钩就会立刻触发。于是，我们可以借助元类，为Field描述符自动设置其`Field.name`和`Field.internal_name`,而不用再像刚才那样，把列的名称手工传给`Field`构造器。
```python
class Meta(type):
    def __new__(meta, name, bases, class_dict):
        for key, value in class_dict.items():
            if isinstance(value, Field):
                value.name = key
                value.internal_name = '_' + key
        cls = type.__new__(meta, name, bases, class_dict)
        return cls
```
下面定义一个基类，该基类使用刚才定义好的`Meta`作为其元类。凡是代表数据库里面某一行的类，都应该从这个基类中继承，以确保他们能够利用元类所提供的功能：
```python
class DatabaseRow(object, metaclass=Meta):
    pass
```
采用元类来实现这套方案时，`Field`描述符类基本上是无需修改的。唯一要调整的地方就在于：现在不需要再给构造器传入参数了，因为刚才编写的`Meta.__new__`方法会自动把相关的属性设置好。
```python
class Field(object):
    def __init__(self):
        # These will be assigned by the metaclass.
        self.name = None
        self.internal_name = None
    def __get__(self, instance, instance_type):
        if instance is None: return self
        return getattr(instance, self.internal_name, '')

    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)
```
有了元类，新的`DatabaseRow`基类以及新的`Field`描述符之后，我们在为数据行定义`DatabaseRow`子类时，就不用再像原来那样，编写重复的代码了。
```python
class BetterCustomer(DatabaseRow):
    first_name = Field()
    last_name = Field()
    prefix = Field()
    suffix = Field()
```
新的`BetterCustomer`类的行为与旧的`Customer`类相同：
```python
foo = BetterCustomer()
print('Before:', repr(foo.first_name), foo.__dict__)
foo.first_name = 'Euler'
print('After: ', repr(foo.first_name), foo.__dict__)
>>>
Before: '' {}
After:  'Euler' {'_first_name': 'Euler'}
```

#### 要点
*  借助元类，我们可以在某个类完全定义好之前，率先修改该类的属性
*  描述符与元类能够有效地组合起来，以便对某种行为作出修饰，或在程序运行时探查相关信息
*  如果把元类与描述符相结合，那就可以在不是用weakref模块的前提下避免内存泄漏


##   第五章 并发及并行
*并发(concurrency)*的意思是说，计算机似乎(seemingly)是在同一时间作者很多不同的事。例如，某台电脑如果只有一个*CPU*核心，那么操作系统就会在各程序之间迅速切换，使其都有机会运行在这一个处理器上面。这种交错执行程序的方式，造成了一种假象，是我们以为这些程序可以同时运行。
<br>*并行(parallelism)*的意思则是说，计算机确实(acutally)实在同一时间做着很多不同的事。具备多个*CPU*核心的计算机，能够同时执行多个程序。各程序中的指令，
都分别运行在每个*CPU*内核赏面，于是，这些程序就能够在同一时刻向前推进。

在同一个程序内部，并发是一种工具，它使程序员可以更加方便地解决特定类型的问题。在并发程序中，不同的执行路径都能够以某种方式向前推进，而这种方式，使人感觉那些路径可以在同一时间独立地运行。

并发与并发的关键区别，在于能不能*提速(speedup)*。某程序若是并行程序，其中有两条不同的执行路径都在平行地向前推进，则总任务的执行时间会减半，执行速度会变为普通程序的两倍。反之，加入该程序是并发程序，那么它既是可以用看似平行的方式分别执行多条路径，也依然不会使总任务的执行速度得到提升。

用Python语言编写并发程序，是比较容易的。通过系统调用、子进程和C语言扩展等机制，也可以用Python平行地处理一些事务。但是，要想使并发式的Python代码以真正平行的方式来运行，却相当困难。所以，我们一定要明白：如何才能在这些有着微妙差别的情景之中，最为恰当地利用Python所提供的特性。

###  第36条： 用subprocess模块来管理子进程
由Python启动的多个子进程，是可以平行运作的，这是的我们能够在Python程序里充分利用电脑中的CPU核心，从而尽量提升程序的*处理能力(throughput,吞吐量)*。虽然Python解释器可能会受限于CPU(第37条)，但是开发者依然可以用Python顺畅地驱动并协调那些耗费CPU的工作任务。Python从诞生至今，演化出了很多中运行子进程的方式，如：`popen`、`popen2`、`os.exec*`等，
但现在最好用切最简单的子进程管理模块，应该是内置的`subporcess`模块。

用`subprocess`模块运行子进程是比较简单的。下面这段代码，用`Popen`构造器来启动进程。然后用`communicate`方法来读取子进程的输出信息，并等待其终止。
```python
import subprocess
proc = subprocess.Popen(
    ['echo', 'Hello from the child!'],
    stdout=subprocess.PIPE) # 注：Windows平台需加上shell=True
out, err = proc.communicate()
print(out.decode('utf-8'))
>>>
Hello from the child!
```
子进程将会独立于父进程而运行，这里的父进程，指的是Python解释器。在下面这个范例程序中，可以一边定期查询子进程的状态，一边处理其他事务。
```python
from time import sleep, time
proc = subprocess.Popen(['sleep', '0.3'])
while proc.poll() is None:
    print('Working...')
    # Some time consuming work here
    sleep(0.2)

print('Exit status', proc.poll())
>>>
Working...
Working...
Exit status 0
```
把子进程从父进程中剥离(decouple,解耦)，意味着父进程可以随意运行很多条平行的子进程。为了实现这一点，我们可以先把所有的子进程都启动起来。
```python
def run_sleep(period):
    proc = subprocess.Popen(['sleep', str(period)])
    return proc

start = time()
procs = []
for _ in range(10):
    proc = run_sleep(0.1)
    procs.append(proc)
```
然后，通过`communicate`方法，等待这些子进程完成其I/O工作并终结。
```python
for proc in procs:
    proc.communicate()

end = time()
print('Finished in %.3f seconds' % (end - start))
>>>
Finished in 0.199 seconds
```
开发者也可以从Python程序向子进程输送数据，然后获取子进程的输出信息。这使得我们可以利用其它程序来平行地执行任务。
<br>例如，要用命令行式的`openssl`工具加密一些数据。下面这段代码，能够以相关的命令行参数及I/O管道，轻松地创建出完成此功能所需的子进程。
```python
import os

def run_openssl(data):
    env = os.environ.copy()
    env['password'] = b'\xe24U\n\xd0Ql3S\x11'
    proc = subprocess.Popen(
        ['openssl', 'enc', '-des3', '-pass', 'env:password'],
        env=env,
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE)
    proc.stdin.write(data)
    proc.stdin.flush()  # Ensure the child gets input
    return proc
```
然后，把一些随机生成的字节数据，传给加密函数。请注意，在实际工作中，传入的应该是用户输入信息、文件句柄、网络套接字等内容：
```python
import os
procs = []
for _ in range(3):
    data = os.urandom(10)
    proc = run_openssl(data)
    procs.append(proc)
```
接下来，这些子进程就可以平行地运作并处理它们的输入信息了。此时，主程序可以等待这些子进程运行完毕，然后获取它们最终的输出结果：
```python
for proc in procs:
    out, err = proc.communicate()
    print(out[-10:])
>>>
b'\xefl\\\xd6F5\x07\x0fO\xe6'
b'X\x94\xfdb\x89~\x01\x00v\xf0'
b'c\xd3G\xd4qY\xd5\xf7\xe8\xde'
```
此外，我们还可以像UNIX管道那样，用平行的子进程来搭建平行的链条，所谓搭建链条(chain),就是把第一个子进程的输出，与第二个子进程的输入联系起来，并以此方式继续拼接下去。下面这个函数，可以启动一个子进程，而该进程会用命令行式的md5工具来处理输入流中的数据：
```python
def run_md5(input_stdin):
    proc = subprocess.Popen(
        ['md5'],
        stdin=input_stdin,
        stdout=subprocess.PIPE)
    return proc
```
现在，启动一套`openssl`进程，一边加密某些数据，同时启动另一套`md5`进程，一边根据加密后的输出内容来计算器哈希码(hash)。
```python
input_procs = []
hash_procs = []
for _ in range(3):
    data = os.urandom(10)
    proc = run_openssl(data)
    input_procs.append(proc)
    hash_proc = run_md5(proc.stdout)
    hash_procs.append(hash_proc)
```
启动起来之后，相关的子进程之间就会自动进行I/O处理。主程序只需等待这些子进程执行完毕，并打印最终的输出内容即可。
```python
for proc in input_procs:
    proc.communicate()

for proc in hash_procs:
    out, err = proc.communicate()
    print(out.strip())
>>>
b'(stdin)= 1e3ed070df31bbfefdb8b2cfda114063'
b'(stdin)= d41d8cd98f00b204e9800998ecf8427e'
b'(stdin)= d41d8cd98f00b204e9800998ecf8427e'
```
如果你担心子进程一直不终止，或担心它的输出管道及输出管道由于某些原因发生了阻塞，那么可以给`communicate`方法传入`timeout`参数。该子进程若在指定时间段内没有给出响应，`communicate`方法则会抛出异常，我们可以在处理异常的时候，终止出现意外的子进程。
```python
proc = run_sleep(10)
try:
    proc.communicate(timeout=0.1)
except subprocess.TimeoutExpired:
    proc.terminate()
    proc.wait()

print('Exit status', proc.poll())
>>>
Exit status -15
```
不幸的是，`timeout`参数仅在Python3.3及后续版本中有效。对于早前的Python版本来说，我们需要使用内置的`select`模块来处理`proc.stdin`、`proc.stdout`和`proc.stderr`,以确保I/O操作的超时机制能够生效。

#### 要点
*  可以用subprocess模块运行子进程，并管理其输入流与输出流
*  Python解释器能够平行地运行多条子进程，这是的开发者可以充分利用cpu的处理能力
*  可以给communicate方法传入timeout参数，以避免子进程死锁或失去响应(hanging,挂起)

###  第37条： 可以用线程来执行阻塞式I/O，但不要用它做平行计算
标准的Python实现叫做*CPython*。*CPython*分两步来运行Python程序。首先，把文本形式的源代码解析并编译成字节码。然后，用一种基于栈的解释器来运行这份字节码。执行Python程序时，字节码解释器必须保持协调一致的状态。Python采用**GIL(global interpreter lock,全局解释器锁)**机制来确保这种协调性。
<br>GIL实际上就是一把互斥锁(mutual-exclusion lock，又称为mutex，互斥体)，用以防止CPython受到占先式多线程切换(preemptive multithreading)操作的干扰。所谓占先式多线程切换，是指某个线程可以通过打断另外一个线程的方式，来获取程序控制权。加入这种干扰操作的执行实际不恰当，那就会破坏解释器的状态。而有了GIL之后，这些干扰操作就不会发生了，GIL可保证每条字节码指令均能够正确地与CPython实现及其C语言扩展模块协同运作。
<br>GIL有一种显著的负面影响。用C++或Java等语言写程序时，可以同时执行多条线程，以充分利用计算机所配备的多个CPU核心。Python程序尽管也支持多线程，但由于受到GIL保护，所以同一时刻，只有一条线程可以向前执行。这就意味着，如果我们想利用多线程做*平行计算(parallel computation)*，并希望借此为Python程序提速，那么结果会非常令人失望。
<br>例如，要用Python执行一项计算量很大的任务。为了模拟此任务，笔者编写了一种非常原始的因数分解算法。
```python
def factorize(number):
    for i in range(1, number + 1):
        if number % i == 0:
            yield i
```
如果逐个地分解许多数据，就会耗费比较长的时间。
```python
from time import time
numbers = [2139079, 1214759, 1516637, 1852285]
start = time()
for number in numbers:
    list(factorize(number))

end = time()
print('Took %.3f seconds' % (end - start))
>>>
Took 0.420 seconds
```
其他语言可以采用多线程来进行如上计算，那样做可以充分利用计算的CPU核心。但是Python却未必如此。
下面定义的Python线程，可以执行与刚才那段范例代码相同的运算：
```python
from threading import Thread

class FactorizeThread(Thread):
    def __init__(self, number):
        super().__init__()
        self.number = number

    def run(self):
        self.factors = list(factorize(self.number))
```
然后，为了实现平行计算，我们为`numbers`列表中的每个数字都启动一条线程。
```python
start = time()
threads = []
for number in numbers:
    thread = FactorizeThread(number)
    thread.start()
    threads.append(thread)

for thread in threads:
    thread.join()

end = time()
print('Took %.3f seconds' % (end - start))
>>>
Took 0.468 seconds
```
令人惊讶的是，这样做所耗费的时间竟然比逐个执行`factorize`所耗的还要长。由于每个数字都有专门的线程负责分解，所以假如改用其他编程语言来实现，那么扣除创建线程和协调线程所需的开销之后，程序的执行速度在理论上应该接近原来的4倍。笔者运行范例代码所用的计算机，拥有两个CPU核心，所以程序执行速度应该变为原来的2倍。我们本来打算利用多个CPU核心来提升程序的速度，但却没有料到多线程的Python程序执行的比单线程还要慢。这样的结果说明，标准CPython解释器中的多线程程序受到了GIL的影响。
<br>通过其他一些方式，我们确实可以令CPython解释器利用CPU的多个内核，但是那些方式所使用的并不是标准的`Thread`类(第41条)，而且还需要开发者编写较多的代码。明白了这些限制之后，你可能会问：既然如此，Python为什么还要支持多线程呢？下面有两个很好的理由：
*   首先，多线程使得程序看上去好像在同一时间做许多事情。如果要自己实现这种效果，并手工管理任务之间的切换，那就显得比较困难(第40条)。而借助多线程，则能够令Python程序自动以一种看似平行的方式，来执行这些函数。之所以能如此，是因为CPython在执行Python形成的时候，可以保证一定程度的公平。不过，由于受到GIL限制，所以同一时刻上只能有一个线程得到执行。
*   Python支持多线程的第二条理由，是处理阻塞式的I/O操作，Python在执行某些系统调用时，会触发此类操作。执行系统调用，是指Python程序请求计算机的操作系统与外界环境相交互，以满足程序的需求。读写文件、在网络间通信，以及与显示器等设备相交互等，都属于阻塞式的I/O操作。为了响应这种阻塞式的请求，操作系统必须花一些时间，而开发者可以借助线程，把Python程序与这些耗时的I/O操作隔离开。
例如，我们要通过串行端口(serial port)发送信号，以便远程控制一架直升飞机。笔者采用一个速度较慢的系统调用(select)来模拟这项活动。该函数请求操作系统阻塞0.1秒，然后把控制权还给程序，这种效果过于通过同步串口来发送信号是类似的。
```python
import select, socket

# Creating the socket is specifically to support Windows. Windows can't do
# a select call with an empty list.
def slow_systemcall():
    select.select([socket.socket()], [], [], 0.1)
```
如果逐个执行上面这个系统调用，那么程序所耗的总时间，就会随着调用的次数而增加。
```python
start = time()
for _ in range(5):
    slow_systemcall()

end = time()
print('Took %.3f seconds' % (end - start))
>>>
Took 0.059 seconds
```
这种写法的问题在于：主程序在运行`slow_systemcall`函数的时候，不能继续向下执行，程序的主线程会卡在`select`系统调用哪里。这种现象在实际的编程工作中是非常可怕的。因为发送信号的同时，程序必须算出直升飞机接下来要移动到的地点，否则飞机可能就会撞毁。如果要同时执行阻塞式I/O操作与计算操作，那就应该考虑把系统调用放到其他线程里面。
下面这段代码，把多个`slow_systemcall`调用分别放到多条线程中执行，这样写，使得程序既能够与多个串口通信(或是通过多个串口来控制许多架直升飞机)，又能够同时在主线程里执行所需的计算。
```python
start = time()
threads = []
for _ in range(5):
    thread = Thread(target=slow_systemcall)
    thread.start()
    threads.append(thread)
```
线程启动之后，我们先算出直升机接下来要移动到的地点，然后等待执行系统调用的线程都运行完毕。
```python
def compute_helicopter_location(index):
    pass

for i in range(5):
    compute_helicopter_location(i)

for thread in threads:
    thread.join()

end = time()
print('Took %.3f seconds' % (end - start))
>>>
Took 0.269 seconds
```
**注(jay)：上面是运行结果和书中作者结果相反，书中后者的运行速度是前者的5倍**
<br>与早前那种逐个执行系统调用的方案相比，这种平行方案的执行速度，接近于原来的5倍。这说明景观受制于GIL，但是用多个Python线程来执行系统调用的时候，这些系统调用可以平行地执行。GIL虽然使得Python代码无法并行，但它对系统调用却没有任何负面影响。由于Python线程在执行系统调用的时候会释放GIL，并且一直要等到执行完毕才会重新获取它，所以GIL是不会影响系统调用的。
<br>除了线程，还有其他一些方式，也能处理阻塞式的I/O操作，例如，内置的`asyncio`模块等。虽然那些方式都有着非常显著的优点，但他们要求开发者必须花些功夫，将代码重构够成另外一种执行模型(第40条)。如果既不像大幅度地修改该程序，又要平行地执行多个阻塞式I/O操作，那么使用多线程来实现会比较简单一些。

#### 要点
*  因为受到全局解释器锁(GIL)的限制，所以多条Python线程不能在多个cpu核心上面平行地执行字节码
*  尽管受制于GIL，但是Python的多线程功能依然很有用，它可以轻松地模拟出同一时刻执行多项任务的效果
*  通过Python线程，我们可以平行地执行多个系统调用，这使得程序能够在执行阻塞式I/O操作的同时，执行一些运算操作

###  第38条： 在线程中使用Lock来防止数据竞争
虽然GIL使得Python线程无法平行地运行在多个CPU核心上面，但它不会保护开发者自己编写的代码。同一时刻固然只能有一个Python线程得以运行，但是当这个线程正在操作某个数据结构时，其他线程可能会打断它。也就是说，Python解释器在执行两个连续的字节码指令时，其他线程可能会在中途突然插进来。如果开发者尝试吃哦你多个线程中同时访问某个对象，那么上述情形就会引发危险的结果。这种中断现象随时都可能啊生，一旦发生，就会破坏程序的状态，从而使相关的数据结构无法保持其一致性。
<br>例如，我们要编写一个程序，平行地统计许多事物。现在假设该程序要从一整套传感器网络中对光照级别进行采样，那么采集到的样本总数，就会随着程序的运行不断增多，于是，新建名为`Counter`的类，专门用来表示样本数量。
```python
class Counter(object):
    def __init__(self):
        self.count = 0

    def increment(self, offset):
        self.count += offset
```
在查询传感器读数的过程中，会发生阻塞式I/O操作，所以，我们要给每个传感器分配它自己的工作线程(work thread)。没采集到一次读书，工作线程就会给`Counter`对象的`value`值加1，然后继续采集，直至完成全部的采样操作。
```python
def worker(sensor_index, how_many, counter):
    # I have a barrier in here so the workers synchronize
    # when they start counting, otherwise it's hard to get a race
    # because the overhead of starting a thread is high.
    BARRIER.wait()
    for _ in range(how_many):
        # Read from the sensor
        counter.increment(1)
```
下面定义的这个`run_threads`函数，会为每个传感器启动一条工作线程，然后等待他们完成各自的采样工作：
```python
from threading import Barrier, Thread
BARRIER = Barrier(5)
def run_threads(func, how_many, counter):
    threads = []
    for i in range(5):
        args = (i, how_many, counter)
        thread = Thread(target=func, args=args)
        threads.append(thread)
        thread.start()
    for thread in threads:
        thread.join()
```
然后，平行地执行这5条线程。我们觉得：这个程序的结果，应该是非常明确的。
```python
how_many = 10**5
counter = Counter()
run_threads(worker, how_many, counter)
print('Counter should be %d, found %d' %
      (5 * how_many, counter.count))
>>>
Counter should be 500000, found 278328
```
从输出结果来看却相差甚远。这么简单的程序怎么会出这么大的错呢？正是Python解释器同一是可只能运行一个线程才发生了这种令人费解的错误。
<br>为了保证所有的线程都能被公平的执行，Python解释器会给每个线程分配大致相等的处理器时间。而，为了达成这样的分配策略，Python系统可能当某个线程正在执行的时候，将其暂停(suspend)，然后使另一个线程继续往下执行。问题在于，开发者无法准确地获知Python系统会在何时暂停这些线程。有一些操作，看上去好像是原子操作(atomic operation),但Python系统依然有可能在线程执行到一半的时候将其暂停。于是，就发生了上面那种情况。
<br>`Counter`对象的`increment`方法`counter.count += offset`看上去很简单，但是，在对象的属性上面使用`+=`操作符，实际上会令Python于幕后执行三项独立的操作。这条语句，可以拆分成下面这三条语句：
```python
value = getattr(counter, 'count')
result = value + offset
setattr(counter, 'count', result)
```
为了实现自增，Python线程必须一次执行上述三个操作，而在任意两个操作之间都有可能发生线程切换。这种交错执行的方式，可能会令线程把旧的`value`设置给`Counter`,从而使程序的运行结果出现问题。我们用A和B两个线程，来演示这种情况：
```python
# Running in Thread A
value_a = getattr(counter, 'count')
# Context switch to Thread B
value_b = getattr(counter, 'count')
result_b = value_b + 1
setattr(counter, 'count', result_b)
# Countext switch back to Thread A
result_a = value_a + 1
setattr(counter, 'count', result_a)
```
线程A执行到一半的时候，线程B插了进来，等线程B执行完整个递增操作之后，线程A有继续执行，于是，线程A就把线程B刚才对计数器所作的递增效果，完全抹去了。传感器采样程序所统计到的样本总数之所以会出错，正是这个原因。
<br>为了防止诸如此类的数据竞争(data race)行为，Python在内置的`threading`模块里提供了一套健壮的工具，使得开发者可以以保护自己的数据结构不受破坏。其中，最简单最有用的工具就是`Lock`类，该类相当于互斥锁。我们可以用互斥锁来保护`Counter`对象，使得多个线程同时访问`value`值的时候，不会将该值破坏。同一时刻，只有一个线程能够获得这把锁。下面这段范例代码，使用`with`语句来获取并释放互斥锁，这样写，能够使阅读代码的人更容易看出线程在拥有互斥锁时，执行的究竟是哪一部分代码(第43条)。
```python
from threading import Lock

class LockingCounter(object):
    def __init__(self):
        self.lock = Lock()
        self.count = 0

    def increment(self, offset):
        with self.lock:
            self.count += offset
```
接下来，就可以像往常一样启动工作线程，只不过这次改用`LockingCounter`来做计数器。
```python
BARRIER = Barrier(5)
counter = LockingCounter()
run_threads(worker, how_many, counter)
print('Counter should be %d, found %d' %
      (5 * how_many, counter.count))
>>>
Counter should be 500000, found 500000
```

#### 要点
*  Python确实有全局解释器锁，但是在编写自己的程序时，依然要设法防止多个线程争用同一份数据
*  如果在不加锁的前提下，允许多条线程修改同一个对象，那么程序的数据结构可能会遭到破坏
*  在Python内置的threading模块中，有个名叫Lock的类，它用标准的方式实现了互斥锁

###  第39条： 用Queue来协调各线程之间的工作
如果Python程序要同时执行许多事务，那么开发和经常需要协调这些事物。而在各种协调方式中较为高效的一种，则是采用函数管线(pipeline)。
<br>管线的工作原理，与制造业中的组装生产线(assembly line)相似。管线分为许多首尾相连的阶段(phase)，每个阶段都由一种具体的函数来负责。程序总是把待处理的新部件添加到管线的开端，每一种函数都可以在它所负责的那个阶段中，并发地处理位于该阶段的部件。等负责本阶段的那个函数，把某个部件处理好之后，该部件就会传送到管线中的下一个阶段，以此类推，知道全部阶段都经历一遍。涉及阻塞式I/O操作或子进程的工作任务，尤其适合用此办法来处理，因为这样的任务，很容易分配到多个Python线程或进程之中(第37条)。
<br>例如，要构建一个照片处理系统，该系统从数码相机里面持续获取照片，调整其尺寸，并将其添加到网络相册中。这样的程序，可以采用三阶段的管线来做。第一阶段获取新图片。第二阶段吧下载好的图片传给缩放函数。第三阶段把缩放后的图片交给上传函数。
<br>假设我们已经用Python代码，把负责这三个阶段的`download`、`resize`和`upload`函数都写好了。那么，如何将它们拼接为一条可以并发处理照片的管线呢？
<br>首先要做的，是设计一种任务传递方式，以便在管线的不同阶段之间传递工作任务。这种方式，可以用线程安全的生产者-消费者队列来建模(线程安全的重要性参考第38条，deque类的用法请参考第46条)。
```python

from threading import Lock
from collections import deque

def download(item):
    return item

def resize(item):
    return item

def upload(item):
    return item

class MyQueue(object):
    def __init__(self):
        self.items = deque()
        self.lock = Lock()

    def put(self, item):
        with self.lock:
            self.items.append(item)

    def get(self):
        with self.lock:
            return self.items.popleft()
```
数码相机在程序中扮演生产者的角色，它会把新的图片添加到`items`列表的末端，这个`items`列表，用来存放待处理的条目。
图片处理管线的第一阶段，在程序中扮演消费者的角色，它会从待处理的条目清单顶部移除图片。
<br>我们用Python线程来表示管线的各个阶段，这种`Worker`线程，会从`MyQueue`这样的队列中取出待处理的任务，并针对该任务运行相关函数，然后把运行结果放到另一个`MyQueue`队列里。此外，`Worker`下称还会记录查询新任务的次数，以及处理完的任务数量。
```python
from threading import Thread
from time import sleep

class Worker(Thread):
    def __init__(self, func, in_queue, out_queue):
        super().__init__()
        self.func = func
        self.in_queue = in_queue
        self.out_queue = out_queue
        self.polled_count = 0
        self.work_done = 0

    def run(self):
        while True:
            self.polled_count += 1
            try:
                item = self.in_queue.get()
            except IndexError:
                sleep(0.01)  # No work to do
            except AttributeError:
                # The magic exit signal
                return
            else:
                result = self.func(item)
                self.out_queue.put(result)
                self.work_done += 1

```
对于`Worker`线程来说，最棘手的部分，就是如何应对输入队列为空的情况。如果上一个阶段没有即时地把相关任务处理完，那就会引发此问题。`run`函数中，我们通过捕获`IndexError`异常来处理这种状况。你可以想象为生产线上的某个环节发生了阻滞。
<br>现在，创建相关的队列，然后根据队列与工作线程之间的对应关系，把整条管线的三个阶段拼接好。
```python
download_queue = MyQueue()
resize_queue = MyQueue()
upload_queue = MyQueue()
done_queue = MyQueue()
threads = [
    Worker(download, download_queue, resize_queue),
    Worker(resize, resize_queue, upload_queue),
    Worker(upload, upload_queue, done_queue),
]
```
启动这些线程，并将大量任务添加到管线的第一个阶段。下面的范例代码中，用简单的`object`对象，来模拟`download`函数所需下载的真实数据：
```python
for thread in threads:
    thread.start()
for _ in range(1000):
    download_queue.put(object())
```
最后，等待管线将所有条目都处理完毕。完全处理好的任务，会出现在`done_queue`队列里。
```python
import time
while len(done_queue.items) < 1000:
    # Do something useful while waiting
    time.sleep(0.1)
# Stop all the threads by causing an exception in their
# run methods.
for thread in threads:
    thread.in_queue = None
```
这个范例程序可以正常运行，但是线程在查询其输入队列并获取新的任务时，可能会产生一种副作用，这是值得我们注意的。`run`方法中有一段微妙的代码，用来捕获`IndexError`异常的，而通过下面的输出信息，可以得知：这段代码运行了很多次。
```python
processed = len(done_queue.items)
polled = sum(t.polled_count for t in threads)
print('Processed', processed, 'items after polling',
      polled, 'times')
>>>
Processed 1000 items after polling 3030 times
```
在管线中，每个阶段的工作函数，其执行速度可能会有所差别，这就使得前一阶段可能会拖慢后一阶段的进度，从而令整条管线迟滞。后一个阶段会在其循环语句中，反复查询输入队列，以求获取新的任务，而前一阶段又迟迟不能把任务交过来，于是就令最后一个阶段陷入了饥饿(starve)。这样做的结果是：工作线程会白白地浪费CPU时间，去执行一些没有用的操作，也就是说，他们会持续地抛出并捕获`IndexError`异常。
<br>上面那种实现方式有很多缺陷，刚才说的那个问题只是其中的一小部分而已。除此之外，还有三个较大的问题，也应该设法避免。首先，为了判断所有的任务是否都彻底处理完毕，我们必须再编写一个循环，持续判断`done_queue`队列中的任务数量。其次，`Worker`线程的`run`方法，会一直执行其循环。即便到了应该退出的时候，我们也没有办法通知`Worker`线程停止这一循环。第三个问题更严重：如果管线的某个阶段发生迟滞，那么随时都可能导致程序崩溃。若第一阶段的处理速度很快，而第二阶段的处理速度较慢，则连接这两个阶段的那个队列的容量就会不断增大。第二阶段始终没有办法跟上第一阶段的节奏。这种现象持续一段时间之后，程序就会因为收到大量的输入数据而耗尽内存，进而崩溃。
<br>这些问题并不能证明管线时一种糟糕的设计方式，它们只是在提醒大家：想要自己打造一种良好的生产者-消费者队列，是非常困难的。

**用Queue类来弥补自编队列的缺陷**
<br>内置的`queue`模块中，有个名叫`Queue`的类，该类能够彻底解决上面提出的那些问题。`Queue`类使得工作线程无需再频繁地查询输入队列的状态，因为他的`get`方法会持续阻塞，知道有新的数据加入。
<br>例如，我们启动一条线程，并令该线程等待`Queue`队列中的输入数据：
```python
from queue import Queue
queue = Queue()

def consumer():
    print('Consumer waiting')
    queue.get()                # Runs after put() below
    print('Consumer done')

thread = Thread(target=consumer)
thread.start()
```
线程虽然已经启动了，但它却并不会立刻就执行完毕，而是会卡在`queue.get()`那里，我们必须调用`Queue`实例的`put`方法，给队列中放入一项任务，方能使`queue.get()`方法得以返回。
```python
print('Producer putting')
queue.put(object())            # Runs before get() above
thread.join()
print('Producer done')
>>>
Consumer waiting
Producer putting
Consumer done
Producer done
```
为了解决管线的迟滞问题，我们用`Queue`类来限定队列中待处理的最大任务数量，使得相邻的两个阶段，可以通过该队列平滑地衔接起来。构造`Queue`时，可以指定缓冲区的容量，如果队列已满，那么后续的`put`方法就会阻塞。例如，定义一条线程，令该线程先等待片刻，然后再去消费`queue`队列中的任务：
```python
queue = Queue(1)               # Buffer size of 1

def consumer():
    time.sleep(0.1)            # Wait
    queue.get()                # Runs second
    print('Consumer got 1')
    queue.get()                # Runs fourth
    print('Consumer got 2')

thread = Thread(target=consumer)
thread.start()
```
之所以要令消费线程等待片刻，是想要给生产线程留出一定的时间，使其可以在`consumer()`方法调用`get`之前，率先通过`put`方法，把两个对象放到队列里面。然而，我们刚才在构建`Queue`的时候，把缓冲区的大小射程了1，这就意味着，生产线程在放入第一个对象之后，会卡在第二个`put`方法那里，它必须等待消费线程通过`get`方法将第一个对象消费掉，然后才能放入第二个对象。
```python
queue.put(object())            # Runs first
print('Producer put 1')
queue.put(object())            # Runs third
print('Producer put 2')
thread.join()
print('Producer done')
>>>
Producer put 1
Consumer got 1
Producer put 2
Consumer got 2
Producer done
```
我们还可以通过`Queue`类的`task_done`方法来追踪工作进度。有了这个方法，我们就不用再像原来那样，在管线末端的`done_queue`处进行轮询，而是可以直接判断：管线中的某个阶段，是否已将输入队列中的任务，全都处理完毕。
```python
in_queue = Queue()

def consumer():
    print('Consumer waiting')
    work = in_queue.get()      # Done second
    print('Consumer working')
    # Doing work
    print('Consumer done')
    in_queue.task_done()       # Done third

Thread(target=consumer).start()
```
现在，生产者线程的代码，既不需要在消费者线程上面调用`join`方法，也不需要轮询消费者线程。生产者只需在`Queue`实例上面调用`join`,并等待`in_queue`结束即可。即便调用`in_queue.join()`时队列为空，`join`也不会立刻返回，必须等消费者线程为队列中的每个条目都调用`task_done()`之后，生产者线程才可以从`join`出继续向下执行。
```python
in_queue.put(object())         # Done first
print('Producer waiting')
in_queue.join()                # Done fourth
print('Producer done')
>>>
Consumer waiting
Producer waiting
Consumer working
Consumer done
Producer done
```
我们把这些行为都封装到`Queue`的子类里面，并且令工作线程可以通过这个`ClosableQueue`类，判断出自己何时应该停止处理。这个子类定义了`close`方法，此方法会给队列中添加一个特殊的对象，用以表明该对象之后再也没有其他任务需要处理了,还为该类定义迭代器，此迭代器在发现特殊对象时，会停止迭代。`__iter__`方法也会在适当的实际调用`task_done`，使得开啊这可以追踪队列的工作进度。
```python
class ClosableQueue(Queue):
    SENTINEL = object()

    def close(self):
        self.put(self.SENTINEL)

    def __iter__(self):
        while True:
            item = self.get()
            try:
                if item is self.SENTINEL:
                    return  # Cause the thread to exit
                yield item
            finally:
                self.task_done()
```

现在，根据`ColseableQueue`类的行为，来重新定义工作线程。这一次，只要`for`循环耗尽，线程就会退出。
```python
class StoppableWorker(Thread):
    def __init__(self, func, in_queue, out_queue):
        super().__init__()
        self.func = func
        self.in_queue = in_queue
        self.out_queue = out_queue

    def run(self):
        for item in self.in_queue:
            result = self.func(item)
            self.out_queue.put(result)
```
接下来，用新的工作线程类，来重新创建线程列表。
```python
download_queue = ClosableQueue()
resize_queue = ClosableQueue()
upload_queue = ClosableQueue()
done_queue = ClosableQueue()
threads = [
    StoppableWorker(download, download_queue, resize_queue),
    StoppableWorker(resize, resize_queue, upload_queue),
    StoppableWorker(upload, upload_queue, done_queue),
]
```
然后，还是像从前那样，运行工作线程。把所有待处理的任务，都添加到管线第一阶段的输入队列之后，则给该队列发出终止信号。
```python
for thread in threads:
    thread.start()
for _ in range(1000):
    download_queue.put(object())
download_queue.close()
```
最后，我们针对管线中相邻两个阶段连接处的那些队列，分别调用`join`方法。也就是说，只要当前阶段处理完毕，我们就给下一个阶段的输入队列里面放入终止信号。等到这些队列全部完工之后，所有的产品都会输出到`done_queue`之中。
```python
download_queue.join()
resize_queue.close()
resize_queue.join()
upload_queue.close()
upload_queue.join()
print(done_queue.qsize(), 'items finished')
>>>
1000 items finished
```

#### 要点
*  管线是一种优秀的任务处理方式，它可以把处理流程划分为若干阶段，并使用多条Python线程来同时执行这些任务
*  构建并发式的管线时，要注意许多问题，其中包括：如何防止某个阶段陷入持续等待的状态之中，如何停止工作线程，以及如何防止内存膨胀等
*  Queue类所提供的机制，可以彻底解决上述问题，它具备阻塞式的队列操作，能够指定缓冲区尺寸，而且还支持join方法，这使得开发者可以构建出健壮的管线

###  第40条： 考虑用协程来并发地运行多个函数
Python的线程有三个显著的缺点：
*   为了确保数据安全，必须使用特殊的工具来协调这些线程(第38，39条)。这使得多线程的代码，要比单线程的过程式代码更加难懂。这种复杂的多线程代码，会逐渐令程序变得难于扩展和维护。
*   线程需要占用大量内存，每个正在执行的线程，大约占据8MB内存。如果只开始几个线程，多数计算机还还可以承受。但是如果要在程序中运行成千上万个函数，并且想用线程来模拟出同时运行的效果，那就会出现问题。在这些函数中，有的函数与用户发送给服务器的请求相对应，有的函数与屏幕上面的像素相对应，还有的函数与仿真程序中的例子想对应。如果没调用有一次函数，就要开一个线程，那么计算机显然无法承受。
*   线程启动时的开销比较大。如果程序不停地依靠创建新线程来同时执行多个函数，并等待这些线程结束，那么使用线程所引发的开销，会拖慢整个程序的速度。

Python的*协程(coroutine)*可以避免上述问题，使得Python程序看上去好像是在同时运行多个函数。协程的实现方式，实际上是对生成器(第16条)的一种扩展。启动生成器协程所需的开销，与调用函数的开销相仿。处于活跃状态的协程，在其耗竭之前，只会占用不到1KB的内存。
<br>协程的工作原理是这样的：每当生成器函数执行到`yield`表达式的时候，消耗生成器的那段代码，就通过`send`方法给生成器回传一个值。而生成器在收到了经由`send`函数所传进来的这个值之后，会将其视为`yield`表达式的执行结果。
```python
def my_coroutine():
    while True:
        received = yield
        print('Received:', received)

it = my_coroutine()
next(it)             # Prime the coroutine
it.send('First')
it.send('Second')
>>>
Received: First
Received: Second
```
在生成器上面调用`send`方法之前，我们要先调用一次`next`函数，以便将生成器推进到第一条`yield`表达式那里。此后，我们可以把`yield`操作与`send`操作结合起来，令生成器能够根据外界所输入的数据，用一套标准的流程来产生对应的输出值。
<br>例如，我们要编写一个生成器协程，并给它一次发送许多数值，而该协程每收到一个数值，就会给当前所统计到的最小值。下面这段范例代码中，第一条`yield`语句中的`yield`关键字，后面没有跟随其他内容，这条语句的意思是，把外界传进来的首个值，当成目前的最小值。此后，生成器会屡次执行`while`循环中的那条`yield`语句，以便将当前统计到的最小值告诉外界，同时等候外界传入下一个待考察的值。
```python
def minimize():
    current = yield
    while True:
        value = yield current
        current = min(value, current)
```
外界的代码在消耗该生成器时，可以每次将其推进一步，而生成器在收到外界发过来的值之后，就会给出当前所统计到的最小值。
```python
it = minimize()
next(it)            # Prime the generator
print(it.send(10))
print(it.send(4))
print(it.send(22))
print(it.send(-1))
>>>
10
4
4
-1
```
生成器函数似乎会一直运行下去，每次在它上面调用`send`之后，都会产生新的值。与线程类似，协程也是独立的函数，它可以消耗由外部环境所传进来的输入数据，并产生相应的输出结果。但与线程不同的是，协程会在生成器函数中的每个`yield`表达式那里暂停，等到外界再次调用`send`方法之后，它才会继续执行到下一个`yield`表达式。这就是协程的奇妙之处。
<br>这种奇妙的机制，使得消耗生成器的那段代码，可以在每执行完协程中的一条`yield`表达式之后，就进行相应的处理。例如，那段代码可以用生成器所产生的输出值来调用其他函数，并更新程序的数据结构。更为重要的是，它可以通过这个输出值，来推进其他的生成器函数，使得那些生成器函数也执行到他们各自的下一条`yield`表达式处。接连推进多个独立的生成器，即可模拟出Python线程的并发行为，令程序看上去好像是在同时运行多个函数。
1.  **生命游戏**
    <br>现在用一个例子，来演示协程的协同运作效果。我们用协程实现康威(John HortonConway)的生命游戏(The Game of Life)。
    <br>游戏规则很简单。在任意尺寸的二维网格中，每个细胞都处在生存(alive)或空白(empty)状态。
    ```python
    ALIVE = '*'
    EMPTY = '-'
    ```
    时钟每走一步，生命游戏就前进一步。向前推进时，我们要点算每个细胞周边的那八个单元格，看看该细胞附近有多少个存活的细胞。本细胞需要根据相邻细胞的存活量，来判断自己在下一轮是继续存活、死亡，还是再生(regenerate)。
    <br>下面从左至右列出五张5x5的生命游戏网格，他们演示了这些细胞在经历四个世代(generation)的变化之后，所呈现的情况。稍后会解释具体的规则。
    ```
      0   |   1   |   2   |   3   |   4
    ----- | ----- | ----- | ----- | -----
    -*--- | --*-- | --**- | --*-- | -----
    --**- | --**- | -*--- | -*--- | -**--
    ---*- | --**- | --**- | --*-- | -----
    ----- | ----- | ----- | ----- | -----
    ```
    我们用生成器协程来建模，把每个细胞都表示为一个协程，并令这些协程步调一致地向前推进。
    <br>为了实现这套模型，我们首先要定义一种方式，来获取相邻细胞的生存状态。用名为`count_neighbors`的协程制作该功能，这个协程会产生`Query`对象，而这个`Query`是自定义的。该类的作用，是向生成器协程提供一种手段，使得协程能够借此向外围环境查询相关的信息。
    ```python
    from collections import namedtuple
    Query = namedtuple('Query', ('y', 'x'))
    ```
    下面这个协程，会针对本细胞的每一个相邻细胞，来产生与之对应的`Query`对象。每个`yield`表达式的就诶过，要么是`ALIVE`,要么是`EMPTY`。这就是协程与消费代码之间的接口契约(interface contract)。其后，`count_neighbors`生成器会根据相邻细胞的状态来返回本细胞周边的存活细胞个数。
    ```python
    def count_neighbors(y, x):
        n_ = yield Query(y + 1, x + 0)  # North
        ne = yield Query(y + 1, x + 1)  # Northeast
        # Define e_, se, s_, sw, w_, nw ...
        e_ = yield Query(y + 0, x + 1)  # East
        se = yield Query(y - 1, x + 1)  # Southeast
        s_ = yield Query(y - 1, x + 0)  # South
        sw = yield Query(y - 1, x - 1)  # Southwest
        w_ = yield Query(y + 0, x - 1)  # West
        nw = yield Query(y + 1, x - 1)  # Northwest
        neighbor_states = [n_, ne, e_, se, s_, sw, w_, nw]
        count = 0
        for state in neighbor_states:
            if state == ALIVE:
                count += 1
        return count
    ```
    接下来，我们用虚构的数据测试这个`count_neighbors`协程。下面这段代码，会针对本细胞的每个相邻细胞，向生成器索要一个`Query`对象，并根据该对象，给出那个相邻细胞的存活状态。然后，通过`send`方法把状态发给协程，使`count_neighbors`协程可以收到上一个`Query`对象所对应的状态。最后，由于协程中的`return`语句会把生成器耗尽，所以程序届时将抛出`StopIteration`异常，而我们可以在处理该异常的过程中，得知本细胞周边的存活细胞总量。
    ```python
    it = count_neighbors(10, 5)
    q1 = next(it)                  # Get the first query
    print('First yield: ', q1)
    q2 = it.send(ALIVE)            # Send q1 state, get q2
    print('Second yield:', q2)
    q3 = it.send(ALIVE)            # Send q2 state, get q3
    print('...')
    q4 = it.send(EMPTY)
    q5 = it.send(EMPTY)
    q6 = it.send(EMPTY)
    q7 = it.send(EMPTY)
    q8 = it.send(EMPTY)
    try:
        it.send(EMPTY)     # Send q8 state, retrieve count
    except StopIteration as e:
        print('Count: ', e.value)  # Value from return statement

    >>>
    First yield:  Query(y=11, x=5)
    Second yield:  Query(y=11, x=6)
    Count:  2
    ```
    `count_neighbors`协程把相邻的存活细胞数量统计出来之后，我们必须根据这个数量来更新本细胞的状态，于是，就需要用一种方式来表示状态的迁移。为此，再定义另一个名叫`step_cell`的协程。这个生成器会产生`Transition`对象，用以表示本细胞的状态迁移。这个`Transition`类，与`Query`一样，这个类也是自定义的。
    ```python
    Transition = namedtuple('Transition', ('y', 'x', 'state'))
    ```
    `step_cell`协程会通过参数来接受当前细胞的网格坐标。它会针对此坐标产生`Query`对象，以查询本细胞的初始状态。接下里，它运行`count_neighbors`协程，以检视本细胞周边的其他细胞。此后，它运行`game_logic`函数，以判断本细胞在下一轮应该处于何种状态。最后，它生成`Transition`对象，把本细胞在下一轮所应有的状态，告诉外部代码。
    ```python
    def game_logic(state, neighbors):
        pass

    def step_cell(y, x):
        state = yield Query(y, x)
        neighbors = yield from count_neighbors(y, x)
        next_state = game_logic(state, neighbors)
        yield Transition(y, x, next_state)
    ```
    `step_cell`协程用`yield from`表达式来调用`count_neighbors`。在Python程序中，这种表达式可以把生成器协程组合起来，使开发者能够更加方便地复用小段的功能代码，并通过简单的协程来构建复杂的协程。`count_neighbors`协程耗竭之后，其最终的返回值(也就是return的值)会作为`yield from`表达式的结果，传给`step_cell`。
    <br>现在，我们终于可以来定义游戏的逻辑函数了，康威生命游戏的规则很简单，只有下面三条：
    ```python
    def game_logic(state, neighbors):
        if state == ALIVE:
            if neighbors < 2:
                return EMPTY     # Die: Too few
            elif neighbors > 3:
                return EMPTY     # Die: Too many
        else:
            if neighbors == 3:
                return ALIVE     # Regenerate
        return state
    ```
    我们现在用虚拟的数据来测试`step_cell`协程。
    ```python
    it = step_cell(10, 5)
    q0 = next(it)           # Initial location query
    print('Me:      ', q0)
    q1 = it.send(ALIVE)     # Send my status, get neighbor query
    print('Q1:      ', q1)
    print('...')
    q2 = it.send(ALIVE)
    q3 = it.send(ALIVE)
    q4 = it.send(ALIVE)
    q5 = it.send(ALIVE)
    q6 = it.send(EMPTY)
    q7 = it.send(EMPTY)
    q8 = it.send(EMPTY)
    t1 = it.send(EMPTY)     # Send for q8, get game decision
    print('Outcome: ', t1)
    >>>
    Me:       Query(y=10, x=5)
    Q1:       Query(y=11, x=5)
    Outcome:  Transition(y=10, x=5, state='-')
    ```
    生命游戏的目标，是要同时在网格中的每个细胞上面，运行刚才编写的那套游戏逻辑。为此，我们把`step_cell`协程组合到新的`simulate`协程之中。新的协程，会多次通过`yield from`表达式，来推进网格中的每一个细胞。把每个坐标点中的细胞都处理完之后，`simulate`协程会产生`TICK`对象，用以表示当前这代的细胞已经全部迁移完毕。
    ```python
    TICK = object()

    def simulate(height, width):
        while True:
            for y in range(height):
                for x in range(width):
                    yield from step_cell(y, x)
            yield TICK
    ```
    `simulate`的好处在于，它和外界环境完全脱离了关联。我们目前还没有定义如何用Python对象来表示网格，也没有定义外部代码应该如何处理`Query`、`Transition`、`TICK`值并设置游戏的初始状态。尽管如此，游戏的逻辑依然是清晰的。每个细胞都可以通过运行`step_cell`来迁移到下一个状态。待所有细胞都迁移好之后，游戏的始终就会向前走一步。只要`simulate`协程还在推进，这个过程就会一直持续下去。
    <br>协程的优势正在于此。它令开发者可以把精力放在当前索要完成的逻辑上面。协程会把程序对环境所下达的指令，与发指令时所用的实现代码互相解耦。这使得程序好像能够平行的运行多个协程，也是的开发者能够在不修改协程的前提下，逐渐改进发布指令时所用的实现代码。
    <br>现在，我们要在真是环境中运行`simulate`。为此，我们需要把网格中每个细胞的状态表示出来。下面定义的这个`Grid`类，代表整张网格：
    ```python
    class Grid(object):
        def __init__(self, height, width):
            self.height = height
            self.width = width
            self.rows = []
            for _ in range(self.height):
                self.rows.append([EMPTY] * self.width)

        def __str__(self):
            output = ''
            for row in self.rows:
                for cell in row:
                    output += cell
                output += '\n'
            return output
    ```
    在查询或设置该网格中的细胞时，调用者可以把任意值当成坐标。如果传入的坐标值越界，那就自动折回，这使得网格看上去好像是一种无限循环的空间。
    ```python
        def query(self, y, x):
        return self.rows[y % self.height][x % self.width]

        def assign(self, y, x, state):
            self.rows[y % self.height][x % self.width] = state
    ```
    最后，定义下面这个函数，它可以对`simulate`及其内部的协程所产生的各种值进行解释。该函数会把协程所产生的指令，转换为与外部环境相关的交互操作。这个函数会把网格内的所有细胞都向前推进一步，待个细胞的状态迁移操作完之后，这些细胞就构成了一张新的网格，而`live_a_generation`函数会把这张新网格返回给调用者。
    ```python
    def live_a_generation(grid, sim):
    progeny = Grid(grid.height, grid.width)
    item = next(sim)
    while item is not TICK:
        if isinstance(item, Query):
            state = grid.query(item.y, item.x)
            item = sim.send(state)
        else:  # Must be a Transition
            progeny.assign(item.y, item.x, item.state)
            item = next(sim)
    return progeny
    ```
    为了验证这个函数的效果，我们需要创建网格并设置其初始状态。下面这段代码，会制作一种名叫glider(滑翔机)的经典形状。
    ```python
    grid = Grid(5, 9)
    grid.assign(0, 3, ALIVE)
    grid.assign(1, 4, ALIVE)
    grid.assign(2, 2, ALIVE)
    grid.assign(2, 3, ALIVE)
    grid.assign(2, 4, ALIVE)
    >>> print(grid)
    ---*-----
    ----*----
    --***----
    ---------
    ---------
    ```
    现在，我们可以逐代推进这张网格，每推进一次，它就变迁到下一代。刚才绘制的那个滑翔机形状，会逐渐朝网格的右下方移动，而这种移动效果，正是由`game_logic`函数里那些简单的规则所确立的。
    ```python
    class ColumnPrinter(object):
        def __init__(self):
            self.columns = []
        
        def append(self, data):
            self.columns.append(data)
        
        def __str__(self):
            row_count = 1
            for data in self.columns:
                row_count = max(row_count, len(data.splitlines()) + 1)
            rows = [''] * row_count
            for j in range(row_count):
                for i, data in enumerate(self.columns):
                    line = data.splitlines()[max(0, j - 1)]
                    if j == 0:
                        padding = ' ' * (len(line) // 2)
                        rows[j] += padding + str(i) + padding
                    else:
                        rows[j] += line
                    if (i + 1) < len(self.columns):
                        rows[j] += ' | '
            return '\n'.join(rows)

    columns = ColumnPrinter()
    sim = simulate(grid.height, grid.width)
    for i in range(5):
        columns.append(str(grid))
        grid = live_a_generation(grid, sim)

    >>> print(columns)
        0     |     1     |     2     |     3     |     4
    ---*----- | --------- | --------- | --------- | ---------
    ----*---- | --*-*---- | ----*---- | ---*----- | ----*----
    --***---- | ---**---- | --*-*---- | ----**--- | -----*---
    --------- | ---*----- | ---**---- | ---**---- | ---***---
    --------- | --------- | --------- | --------- | ---------
    ```
    上面这套实现方式，最大的优势在于：开发者能够在不修改`game_logic`函数的前提下，更新该函数外围的那些代码。我们可以在现有的`Query`、`Transition`和`TICK`机制的基础之上修改相关的规则，或是家影响范围更为广泛的效果。上面这套范例代码，演示了如何用协程来分离程序中的各个关注点，而关注点的分离(the separation of concerns),正是一条重要的设计原则。
2.  **Python2的协程**
    <br>Python3的协程写起来非常优雅，这是因为它提供了方便开发者编写代码的语法糖，而Python2则缺乏这样的机制。下面我们来讲讲协程在Python2中的两项限制。
    *   首先，Python2没有`yield from`表达式。这意味着，如果要在Python2程序里里面把两个生成器协程组合起来，就需要在委派给另一个协程的地方，多写一层循环。
        ```python
        # Python 2
        def delegated():
            yield 1
            yield 2

        def composed():
            yield 'A'
            for value in delegated():  # yield from in Python 3
                yield value
            yield 'B'

        print list(composed())
        >>>
        ['A', 1, 2, 'B']
        ```
    *   第二个限制是：Python2的生成器不支持`return`语句。为了通过`try/except/finally`代码块正确实现出与Python3相同的行为，我们需要定义自己的异常类型，并在需要返回某个值的时候，抛出该异常。
        ```python
        # Python 2
        class MyReturn(Exception):
            def __init__(self, value):
                self.value = value

        def delegated():
            yield 1
            raise MyReturn(2)  # return 2 in Python 3
            yield 'Not reached'

        def composed():
            try:
                for value in delegated():
                    yield value
            except MyReturn as e:
                output = e.value
            yield output * 4

        print list(composed())
        >>>
        [1, 8]
        ```

#### 要点
*  协程提供了一种有效的方式，令程序看上去好像能够同时运行大量函数
*  对于生成器内的yield表达式来说，外部代码通过send方法传给生成器的那个值，就是该表达式所要具备的值
*  协程是一种强大的工具，它可以把程序的核心逻辑与程序同外部环境交互时所用的代码相分离
*  Python2不支持yield from表达式，也不支持从生成器内通过return语句向外界返回某个值

###  第41条： 考虑用concurrent.futures来实现真正的平行计算
Python的全局解释器锁(GIL)使得我们没有办法用线程实现真正的平行计算。
<br>要想Python程序实现真正的平行计算来提高性能，常见的建议是：用C语言把程序中性能要求较高的部分代码改写为扩展模块。由于C语言更贴近硬件，所以运行的比Python快，一旦运行速度达到要求，自然就不用再考虑平行计算了。C语言扩展也可以启动并平行地运行多条原生线程(native thread),从而充分利用CPU的多个内核。Python中的C语言扩展API，有完备的文档可供查阅，这使得它成为解决性能问题的一个好办法。
<br>但是用C语言重写代码，是有很大代价的。短小而易读的Python代码，会变成冗长而费解的C代码。在进行这样的移植时，必须进行大量的测试，以确保移植之后的C程序，在功能上与原来的Python程序等效，而且还要确保移植过程中没有引入bug。有时候，这些努力是值得的。例如，Python开发社区的各种C语言扩展模块，就构成了一套庞大的生态系统，这些模块，能够提升文本解析、图像合成和矩阵运算等操作的执行速度。此外，还有如`CPython`和`Numba`等开源工具，可以帮助开发者把Python代码更加顺畅地迁移到C语言。
<br>然后问题在于：只把程序中的一小部分迁移到C，通常是不够的。一般来说，Python程序之所以执行得比较慢，并不是某个主要因素单独造成的，而是多个因素联合导致的。所以，要想充分利用C语言的硬件和线程优势，就必须把程序中的大量代码移植到C，而这样做大幅增加了测试量和风险。于是我们应该思考一下：有没有一种更好的方式，只需使用较少的Python代码，即可有效提升执行速度，并迅速解决复杂的计算问题。
<br>我们可以试着通过内置的`concurrent.futures`模块，来利用另外一个名叫`multiprocessing`的内置模块，从而实现这种需求。该做法会以子进程的形式，平行地运行多个解释器，而从令Python程序能够利用多核心CPU来提升执行速度。由于子进程与主解释器相分离，所以他们的全局解释器锁也是互相独立的。每个子进程都可以完整地利用一个CPU内核，而且这些子进程，都与主进程之间有着联系，通过这条联系渠道，子进程可以接受主进程发过来的指令，并把计算结果返回给主进程。
<br>例如，我们现在要编写一个运算量很大的Python程序，并且要在该程序中充分利用CPU的多个内核。范例采用查找两数最大公约数的算法来演示这种编程方式。
```python
def gcd(pair):
    a, b = pair
    low = min(a, b)
    for i in range(low, 0, -1):
        if a % i == 0 and b % i == 0:
            return i
```
由于没有做平行计算，所以程序会一次用`gcd`函数来求各组数字的最大公约数，这将导致程序的运行时间随着数据量的增多而变长。
```python
from time import time
numbers = [(1963309, 2265973), (2030677, 3814172),
           (1551645, 2229620), (2039045, 2020802)]
start = time()
results = list(map(gcd, numbers))
end = time()
print('Took %.3f seconds' % (end - start))
>>>
Took 0.433 seconds
```
因为GIL的原因，用多条线程来改进上述程序是没有效果的。
<br>下面这个程序，借助`concurrent.futures`模块来执行与刚才相同的运算，它使用`ThreadPoolExecutor`类及两个工作线程来实现(*max_workers*表示工作线程的数量，此参数与CPU的核心数相同)：
```python
from concurrent.futures import ThreadPoolExecutor

start = time()
pool = ThreadPoolExecutor(max_workers=2)
results = list(pool.map(gcd, numbers))
end = time()
print('Took %.3f seconds' % (end - start))
>>>
Took 0.431 seconds
```
线程启动的时候是有一定开销的，与线程池进行通信也会有开销，所以上面这个程序运行的速度和单线程版本差不多。
<br>然而神奇的是：我们只需改动一行代码，就可以提升整个程序的速度。只要把`ThreadPoolExecutor`换成`concurrent.futures`模块里面的`ProcessPoolExecutor`,程序的速度就上去了。
```python
from concurrent.futures import ProcessPoolExecutor

start = time()
pool = ProcessPoolExecutor(max_workers=2)  # The one change
results = list(pool.map(gcd, numbers))
end = time()
print('Took %.3f seconds' % (end - start))
>>>
Took 0.313 seconds
```
这是什么原因呢？
<br>这是因为`ProcessPoolExecutor`类会利用有`multiprocessing`模块所提供的底层机制来逐步完成下列操作：
1.  把`numbers`列表中的米一项输入数据都传给`map`
2.  用`pickle`模块(第44条)对数据进行序列化，将其变成二进制形式
3.  通过本地套接字(local socket)，将序列化之后的数据从主解释器所在的进程，发送到子解释器所在的进程
4.  接下来，在子进程中，用`pickle`对二进制数据进行反序列化操作，将其还原为Python对象。
5.  引入包含`gcd`函数的那个Python模块
6.  各条子进程平行地针对各自的输入数据，来运行`gcd`函数
7.  对运算结果进行序列化操作，将其转变为字节
8.  将这些字节通过`socket`复制到主进程之中
9.  主进程对这些字节执行反序列化操作，将其还原为Python对象
10. 最后，把每条子进程所求出的计算结果合并到一份列表之中，并返回给调用者。

从编程者的角度看，这些步骤似乎是比较简单的，但实际上，为了实现平行计算，`multiprocessing`模块和`ProcessPoolExecutor`类在幕后做了大量的工作。如果改用其他编程语言来写，那么开发者只需用一把同步锁或一项原子操作，就可以把线程之间的通信过程协调好而在Python语言中，我们却必须使用开销较高的`multiprocesssing`模块。`multiprocessing`模块的开销之所以比较大，原因就在于：主进程和子进程之间，必须进行序列化和反序列化操作，而程序中的大量开销，正是这些操作引起的。
<br>对于某些较为孤立，且数据利用率较高的任务来说，这套方案非常合适。所谓孤立，是指待运行的函数不需要与程序中的其他部分共享状态。所以利用率高，是指只需要在主进程与子进程之间传递一小部分数据，就能完成大量的运算。本例中的最大公约数算法，满足这两个条件，其他一些类似的数学算法，也可以通过这套方案实现平行计算。
<br>如果待执行的运算不符合上述特征，那么`multiprocessing`所产生的开销，可能是我们无法通过平行化(parallelization)来提升程序速度。在那种情况下，可以求助`multiprocessing`所提供的一些高级机制，如共享内存(shared memory)、跨进程锁定(cross-process lock)、队列(queue)和代理(proxy)等。不过，那些特性用起来非常复杂。想通过那些工具令多条Python线程共享同一个进程的内存空间，本身已经相当困难，若还想经由`socket`把他们套用到其他进程，则会使代码变得更加难懂。
<br>建议大家多使用简单的`concurrent.futures`模块，并且尽量避开`multiprocessing`里的那些复杂特性。对于较为孤立且数据利用率较高的函数来说，一开始可以试着用多线程的`ThreadPoolExecutor`来运行。稍后，可以将其迁移到`ProcessPoolExecutor`类，看能不能提升程序的执行速度。如果试遍了各种方案，还是无法达到理想的执行速度，再考虑使用`multiprocessing`模块中的复杂特性。

#### 要点
*  把引发cpu性能瓶颈的那部分代码，用C语言扩展模块来改写，即可在尽量发挥Python特性的前提下，有效提升程序的执行速度。但是，这样做的工作量比较大，而且可能会引入bug
*  multiprocessing模块提供了一些强大的工具。对于某些类型的任务来说，开发者只需编写少量代码，即可实现平行计算
*  若想利用强大的multiprocessing模块，最恰当的做法，就是通过内置的concurrent.futures模块及其ProcessPoolExecutor类来使用它
*  multiprocessing模块所提供的那些高级功能，都特别复杂，所以开发者尽量不要直接使用它们


##   第六章 内置模块
###  第42条： 用functools.wraps定义函数修饰器
Python用特殊的语法来表示修饰其(*decorator*),这些修饰其可以用来修饰函数。对于受到封装的原函数来说，修饰器能够在那个函数执行前后分别运行一些附加代码。这是的开发者可以在修饰器里面访问并修改原函数的参数及返回值，以实现约束语义(enforce semantics)、调试程序、注册函数等目标。
<br>例如，我们要打印某个函数在受到调用时所接受的参数以及该函数的返回值。对于包含一系列函数调用的递归函数来说，这样的调试功能尤其有用。下面就来定义这种修饰其：
```python
def trace(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print('%s(%r, %r) -> %r' %
              (func.__name__, args, kwargs, result))
        return result
    return wrapper
```
可以用`@`符号把刚才那个修饰套用到某个函数上面。
```python
@trace
def fibonacci(n):
    """Return the n-th Fibonacci number"""
    if n in (0, 1):
        return n
    return (fibonacci(n - 2) + fibonacci(n - 1))
```
使用`@`符号来修饰函数，其效果就等于先以该函数为参数，调用修饰器，然后把修饰器所返回的结果，赋给同一作用域中与原函数同名的那个变量。
```python
fibonacci = trace(fibonacci)
```
如果调用这个修饰之后的`fibonacci`函数，那么它就会在执行原函数之前及之后，分别运行`wrapper`中的附加代码，使开发者能够看到原`fibonacci`函数在递归栈的每一个层级上所具备的参数和返回值。
```python
fibonacci(3)
>>>
fibonacci((1,), {}) -> 1
fibonacci((0,), {}) -> 0
fibonacci((1,), {}) -> 1
fibonacci((2,), {}) -> 1
fibonacci((3,), {}) -> 2
```
上面这个修饰器虽然可以正常运作，但却会产生一种我们不希望看到的副作用。也就是说，修饰其所返回的那个值，其名称会和原来的函数不同，它现在不叫`fibonacci`了。
```python
print(fibonacci)
>>> 
<function trace.<locals>.wrapper at 0x7f30e8bad048>
```
这种效果的产生原因比较微妙。`trace`函数所返回的值，是它内部定义的那个`wrapper`。而我们又使用`trace`来修饰原有的`fibonacci`函数，于是，Python就会把修饰器内部的那个`wrapper`函数，赋给当前模块中与原函数同名的`fibonacci`变量。对于调试器(第57条)和对象序列化器(第44条)等需要使用内省(intospection)机制的那些工具来说，这样的行为会干扰他们的正常工作。
<br>例如，修饰后的`fibonacci`函数，会令内置的`help`函数失效。
```python
help(fibonacci)
>>>
Help on function wrapper in module __main__:

wrapper(*args, **kawargs)
```
这个问题，可以用内置的`functools`模块中名为`wraps`的辅助函数来解决。`wraps`本身也是修饰器，它可以帮助开发者编写其他修饰器。将`wraps`修饰其运用到`wrapper`函数之后，它就会把与内部函数相关的重要元数据全部复制到外围函数。
```python
from functools import wraps
def trace(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print('%s(%r, %r) -> %r' %
              (func.__name__, args, kwargs, result))
        return result
    return wrapper

@trace
def fibonacci(n):
    """Return the n-th Fibonacci number"""
    if n in (0, 1):
        return n
    return (fibonacci(n - 2) +
            fibonacci(n - 1))
```
现在，即使函数经过装饰，运行`help`也依然能打印出合理的结果。
```python
help(fibonacci)
>>>
Help on function fibonacci in module __main__:

fibonacci(n)
    Return the n-th Fibonacci number
```
刚才我们看到:如果编写修饰器的时候，没有用`wraps`做相应的处理，那就会令`help`函数失效。除了`help`函数，修饰器还会导致其他一些难于排查的问题。为了维护函数的接口，修饰之后的函数还会导致其他一些难于排查的问题。为了，维护函数的接口，修饰之后的函数，必须保留原函数的某些标准Python属性，例如，`__name__`和`__module__`。因此，我们需要用`wraps`来确保修饰后的函数具备正确的行为。


#### 要点
*   Python为修饰器提供了专门的语法，它是的程序在运行的时候，能够用一个函数来修改另一个函数
*   对于调试器这种依靠内省机制的工具，直接编写修饰其会引发奇怪的行为
*   内置的`functools`模块提供了名为`wraps`的修饰器，开发者在定义自己的修饰器时，应该用`wraps`对其做一些处理，以避免一些问题

###  第43条： 考虑以contenexlib和with语句来改写可复用的try/finally代码
有些代码需要运行在特殊的情境之下，开发和可以用Python语言的`with`语句来表达这些代码的运行机制。例如，如果把互斥锁放在`with`语句之中，那就表示：只有当程序持有该锁的时候，`with`语句块里的那些代码才会运行。
```python
from threading import Lock
lock = Lock()
with lock:
    print('Lock is held')
```
由于`Lock`类对`with`语句提供了适当的支持，所以上面那种写法，可以达到下面`try/finally`结构相仿的效果：
```python
lock.acquire()
try:
    print('Lock is held')
finally:
    lock.release()
```
上面两种写法中，使用`with`语句的版本会更好一些，因为它免去了编写`try/finally`结构所需的重复代码。开发者可以用内置的`contextlib`模块来处理自己所编写的对象和函数，使他们能够支持`with`语句。该模块提供了名为`contextmanager`的修饰器。一个简单的函数，只需经过`contextmanager`修饰，即可用在with语句之中。这样做，要比标准的写法更加便捷。如果按标准方式来做，那就要定义新的类，并提供名为`__enter__`和`__exit__`的特殊方法。
<br>例如，当程序运行到某一部分是，我们希望针对这部分代码，打印出更为详细的调试信息。下面定义的这个函数，可以打印两种严重程度不同的日志：
```python
import logging
logging.getLogger().setLevel(logging.WARNING)
def my_function():
    logging.debug('Some debug data')
    logging.error('Error log here')
    logging.debug('More debug data')
```
如果程序的默认日志级别是`WARNING`，那么运行该函数是，就只会打印出`ERROR`级别的日志。
```python
my_function()
>>>
Error log here
```
我们可以定义一种情境管理器，来临时提升该函数的日志级别。下面这个辅助函数，会在运行`with`块内的代码之前，临时提升日志级别，待`with`块执行完毕，在恢复原有级别。
```python
from contextlib import contextmanager
@contextmanager
def debug_logging(level):
    logger = logging.getLogger()
    old_level = logger.getEffectiveLevel()
    logger.setLevel(level)
    try:
        yield
    finally:
        logger.setLevel(old_level)
```
`yield`表达式所在的地方，就是`with`块中的语句所要展开执行的地方。`with`块所抛出任何异常，都会有`yield`表达式重新抛出，这是的开发者可以在辅助函数里面捕获它(第40条)。
<br>现在重新运行这个`my_function`函数，但是这一次，我们把它放在`debug_logging`情境之下。可以看到：`with`块中的那个`my_function`函数，会把所有*DEBUG*级别的信息打印出来，而`with`块外的那个`my_function`函数，则不会打印*DEBUG*级别的信息。
```python
with debug_logging(logging.DEBUG):
    print('Inside:')
    my_function()
print('After:')
my_function()
>>>
Inside:
Some debug data
Error log here
More debug data
After:
Error log here
```
**使用带有目标的`with`语句**
<br>传给`with`语句的那个情境管理器，本身也可以返回一个对象。而开发者可以通过`with`符合语句中的`as`关键字，来指定一个局部变量，Python会把那个对象，赋给这个局部变量。这使得`with`块中的代码，可以直接与外部情境相交互。
<br>例如，我们要向文件中写入数据，并且要确保该文件总是能正确地关闭。这个功能，可以通过with语句来实现。把`open`传给`with`语句，并通过`as`关键字来制定一个目标变量，用以接受`open`所返回的文件句柄，等到`with`语句块退出时，该句柄会自动关闭。
```python
with open('my_output.txt', 'w') as handle:
    handle.write('This is some data!')
```
与每次手工开启并关闭文件句柄的写法相比，上面这个写法更好一些。它是的开发者能够确信：只要程序离开`with`语句块，文件就一定会关闭。此外，它也促使开发者在打开文件句柄之后，尽量少些一些代码，这通常是一种良好的编程习惯。
<br>我们只需在情境管理器里，通过`yield`语句返回一个值，即可令自己的函数把该值提供给由`as`关键字所指定的目标变量。例如，下面定义的这个情境管理器，能够获取`Logger`实例、设置其级别，并通过`yield`语句将其提供给由`as`关键字所指定的目标：
```python
@contextmanager
def log_level(level, name):
    logger = logging.getLogger(name)
    old_level = logger.getEffectiveLevel()
    logger.setLevel(level)
    try:
        yield logger
    finally:
        logger.setLevel(old_level)
```
由于`with`语句块可以把日志级别调低，所以在`as`目标变量上面调用`debug`等方法时，可以打印出*DEBUG*级别的调试信息。与之相反，若直接在`logging`模块上面调用`debug`,则不会打印出任何*DEBUG*级别的日志，因为Python自带的`logger`,默认会处在*WARNING*级别。
```python
with log_level(logging.DEBUG, 'my-log') as logger:
    logger.debug('This is my message!')
    logging.debug('This will not print')
>>>
This is my message!
```
退出`with`语句块之后，我们在名为'my-log'的`Logger`上面调用`debug`方法，是打印不出日志的，因为该`Logger`的日志级别已经恢复到默认的*WARNING*了。但是，由于*ERROR*级别高于*WARNING*级别，所以在这个`Logger`上面调用`error`法官法，仍然能够打印出日志。
```python
logger = logging.getLogger('my-log')
logger.debug('Debug will not print')
logger.error('Error will print')
>>>
Error will print
```

#### 要点
*   可以用`with`语句来改写`try/finally`块中的逻辑，以便提升复用程度，并使代码更加整洁。
*   内置的`contextlib`模块提供了名叫`contextmanager`的修饰其，开发者只需用它来修饰自己的函数，即可令函数支持`with`语句。
*   情景管理器可以通过`yield`语句向`with`语句返回一个值，此值会赋给由`as`关键字所指定的变量。该机制阐明了这个特殊情境的编写动机，并令`with`块中的语句能够直接访问这个目标变量。

###  第44条： 用copyreg实现可靠的pickle操作
内置的`pickle`模块能够将Python对象序列化成字节流，也能把这个写字节流反序列化为Python对象。但经过`pickle`处理的字节流，不应该在未受信任的程序之间传播。`pickle`的设计目标是提供一种二进制渠道，使开发者能够在自己所控制的各程序之间传递Python对象。
<br>注：由`pickle`模块所产生的序列化数据，采用的是一种不安全的格式。这种序列化数据实际上就是一个程序，它描述了如何来构建原始的Python对象。这意味着：经过`pickle`处理之后的数据如果混入了恶意信息，那么Python程序在对其进行反序列化时，这些恶意信息可能对程序造成损害。
<br>例如，我们要用Python对象表示玩家的游戏进度。下面这个`GameState`类，包含了玩家当前的级别，以及剩余的生命值。
```python
class GameState(object):
    def __init__(self):
        self.level = 0
        self.lives = 4
```
程序会在游戏过程中修改`GameState`对象。
```python
state = GameState()
state.level += 1  # Player beat a level
state.lives -= 1  # Player had to try again
```
玩家退出游戏时，程序可以把游戏状态保存到文件里，以便稍后恢复。使用`pickle`模块来实现这个功能，是非常简单的。下面这段代码，会把`GameState`对象直接写到一份文件里：
```python
state_path = 'game_state.bin'
with open(state_path, 'wb') as f:
    pickle.dump(state, f)
```
以后，可以用`load`函数来加载这个文件，并把`GameState`对象还原回来。还原好的`GameState`对象，与没有经过序列化操作的普通对象一样，看不出太大区别。
```python
with open(state_path, 'rb') as f:
    state_after = pickle.load(f)
print(state_after.__dict__)
>>>
{'lives':3, 'level':1}
```
在游戏功能逐渐扩展的过程中，上面那种写法会暴漏出一些问题。例如，为了鼓励玩家追求高分，我们想给游戏添加记分功能。于是，给`GameState`类添加`points`字段以表示玩家的分数。
```python
class GameState(object):
    def __init__(self):
        self.level = 0
        self.lives = 4
        self.points = 0
```
针对新版的`GameState`类来使用`pickle`模块，其效果与早前相同。下面这个段代码，先用`dumps`函数将对象序列化成字符串，然后又用`loads`函数将字符串还原成对象，这样做，与通过文件来保存并还原对象是相似的：
```python
state = GameState()
serialized = pickle.dumps(state)
state_after = pickle.loads(serialized)
print(state_after.__dict__)
>>>
{'lives':4, 'level':0, 'points':0}
```
但是，如果有一份存档是用旧版的`GameState`格式保存的，而现在玩家有要用这份存档来继续游戏，会出现什么情况呢？下面这段范例代码，根据新版的`GameState`定义来对旧版的游戏存档进行`unpickle`操作：
```python
with open(state_path, 'rb') as f:
    state_after = pickle.load(f)
print(state_after.__dict__)
>>>
{'lives':3, 'level':1}
```
还原出来的对象，竟然没有`points`属性！由`pickle.load`所返回的对象，是个新版的`GameState`实例，可是这个新版的`GameState`实例，怎么会没有`points`属性呢？这非常令人困惑。`assert isinstance(state_after, GameState)`
<br>这种行为，是`pickle`模块的工作机制所表现出的副作用。`pickle`模块的主要意图，是帮助开发者轻松地在对象上面执行序列化操作。如果对`pickle`模块的运用超越了这个范围，那么该模块的功能就会出现奇怪的问题。
<br>要想解决这些问题，也非常简单，只需借助内置的`copyreg`模块即可。开发者可以用`copyreg`模块注册一些函数，Python对象的序列化，将由这些函数来责对。这使得我么可以控制`pickle`操作的行为，令其变得更加可靠。
1.  为缺失的属性提供默认值
    在最简单的情况下，我们可以用带默认值的构造器(第19条)，来确保`GameState`对象在经过`unpickle`操作之后，总是能够够具备所有的属性。下面重新来定义`GameState`类的构造器：
    ```python
    class GameState(object):
        def __init__(self, level=0, lives=4, points=0):
            self.level = level
            self.lives = lives
            self.points = points
    ```
    为了使用这个构造器进行`pickle`操作，定义下面的辅助函数，它接受`GameState`对象，并将其转换为一个包含参数的元组，以便提供给`copyreg`模块。返回的这个元组，含有`unpickle`操作所使用的函数，以及要传给那个`unpickle`函数的参数。
    ```python
    def pickle_game_state(game_state):
        kwargs = game_state.__dict__
        return unpickle_game_state, (kwargs,)
    ```
    现在，定义`unpickle_game_state`这个辅助函数。该函数接受由`pickle_game_state`所传来的序列化数据及参数，并返回响应的`GameState`对象。这其实就是对构造器做了小小的封装。
    ```python
    def unpickle_game_state(kwargs):
        return GameState(**kwargs)
    ```
    下面通过内置的`copyreg`模块来注册`pickle_game_state`函数。
    ```python
    import copyreg
    copyreg.pickle(GameState, pickle_game_state)
    ```
    序列化与反序列化操作，都可以像原来那样，照常进行。
    ```python
    state = GameState()
    state.points += 1000
    serialized = pickle.dumps(state)
    state_after = pickle.loads(serialized)
    print(state_after.__dict__)
    >>>
    {'lives':4, 'level':0, 'points':1000}
    ```
    注册好`pickle_game_state`函数之后，我们可以修改`GameState`的定义，给玩家一定数量的魔法卷轴。这次修改与早前给`GameState`类添加`points`字段时所做的修改时类似的。
    ```python
    class GameState(object):
        def __init__(self, level=0, lives=4, points=0, magic=5):
            self.level = level
            self.lives = lives
            self.points = points
            self.magic = magic
    ```
    但是这一次，对旧版的`GameState`对象进行反序列化操作，就可以得到正确的游戏数据了，而不会再像原来那样丢失某些属性。由于`unpickle_game_state`会直接调用`GameState`构造器，所以反序列化之后的`GameState`对象，其属性是完备的。构造器的关键字参数，都带有默认值，如果某个参数缺失，那么对应的属性就会自动具备相应的默认值。于是，旧版的游戏存档在经过反序列化操作之后，就会拥有新的`magic`字段，而该字段的值，正是构造器的关键字参数所提供的默认值。
    ```python
    state_after = pickle.loads(serialized)
    print(state_after.__dict__)
    >>>
    {'level':0, 'points':1000, 'magic':5, 'lives':4}
    ```
2.  用版本号来管理类
    有的时候，我们要从现有的Pytho类中移除某些字段，而这种操作，会导致新类与旧类无法兼容。刚才那种通过带有默认值的参数来进行反序列化的方案，无法应对这种情况。
    <br>例如，我们现在认为，游戏不应该限制玩家的生命值，于是，我们就像把生命值这一概念从游戏中拿掉。下面定义的这个`GameState`构造器，不在包含`lives`字段：
    ```python
    class GameState(object):
        def __init__(self, level=0, points=0, magic=5):
            self.level = level
            self.points = points
            self.magic = magic
    ```
    修改了构造器之后，程序就无法对旧版的游戏数据进行反序列化操作了。因为旧版游戏数据中的所有字段，都会通过`unpickle_game_state`函数，传给`GameState`构造器，即使某个字段已经从新的`GameState`类里移除，它也依然要传入`GameState`构造器。
    ```python
    pickle.loads(serialized)
    >>>
    TypeError: __init__ got an unexpected keyword argument
    'lives'
    ```
    解决办法是：修改我们向`copyreg`模块注册的那个`pickle_ganme_state`函数，在该函数里添加一个表示版本号的参数。在对新版的`GameState`对象进行`pickle`的时候，`pickle_game_state`函数会在序列化后的新版数据里面，添加值为2的`version`参数。
    ```python
    def pickle_game_state(game_state):
        kwargs = game_state.__dict__
        kwargs['version'] = 2
        return unpickle_game_state, (kwargs,)
    ```
    而旧版的游戏数据里面，并没有这个`version`参数，所以，在把参数传给`GameState`构造器的时候，我们可以根据数据中是否包含`version`参数，来做相应的处理。
    ```python
    def unpickle_game_state(kwargs):
        version = kwargs.pop('version', 1)
        if version == 1:
            kwargs.pop('lives')
        return GameState(**kwargs)
    ```
    现在，我们就可以照常对旧版的游戏存档进行反序列化操作了。
    ```python
    copyreg.pickle(GameState, pickle_game_state)
    state_after = pickle.loads(serialized)
    print(state_after.__dict__)
    >>>
    {'magic':5, 'level':0, 'points':1000}
    ```
    以后如果还要在这个类上面继续做修改，那我们依然可以用这套办法来管理不同的版本。把旧版的类适配到新版的类时，需要编写一些代码，而这些代码，都可以放到`unpickle_game_state`函数里面。
3.  **固定的引入路径**
    使用`pickle`模块时，还会出现一个问题，那就是：当类的名称改变之后，原有的数据无法正常执行反序列化操作。在程序的生命期内，我们通常会重构自己的代码，修改某些类的名称，并把它们移动到其他模块。在做这种重构时，必须多加小心，否则会令程序无法正常使用`pickle`模块。
    <br>下面把`GameState`类的名称改为`BetterGameState`，并把原来的类从程序中彻底删掉：
    ```python
    copyreg.dispatch_table.clear()
    state = GameState()
    serialized = pickle.dumps(state)
    del GameState
    class BetterGameState(object):
        def __init__(self, level=0, points=0, magic=5):
            self.level = level
            self.points = points
            self.magic = magic
    ```
    此时对旧版的`GameState`对象进行反序列化操作，那么程序就会出错，因为它找不到这个类。
    ```python
    pickle.loads(serialized)
    >>>
    AttributeError: Can't get attribute 'GameState' on <module> '__main__' from 'my_code.py'
    ```
    发生这个异常的原因在于：序列化之后的数据，把该对象所属类的引入路径，直接写在了里面。
    ```python
    print(serialized[:25])
    >>>
    b'\x80\x03c__main__\nGameState\nq\x00'
    ```
    这个问题也可以通过`copyreg`模块来解决。我们可以给函数指定一个固定的标识符，令他采用这个标识符来对数据进行`unpickle`操作。这使得我们在反序列化的时候能够把原来的数据迁移到名称不同的其他类上面。我们可以利用这种间接的机制，来处理类名变更问题。
    ```python
    copyreg.pickle(BetterGameState, pickle_game_state)
    ```
    使用了`copyreg`之后，我们可以看到：嵌入序列化数据之中的引入路径，不再指向`BetterGameState`这个类名了，而是会执向`unpickle_game_state`函数。
    ```python
    state = BetterGameState()
    serialized = pickle.dumps(state)
    print(serialized[:35])
    >>>
    b'\x80\x03c__main__\nunpickle_game_state\nq\n\x00'
    ```
    唯一需要注意的地方是，不能修改`unpickle_game_state`函数所在的模块路径。程序把这个函数写尽了序列化之后的数据里面，所以，将来执行反序列化操作的时候，程序也必须能找到这个函数才行。

#### 要点
*   内置的`pickle`模块，只适合用来在彼此信任的程序之间，对相关对象执行序列化和反序列化操作。
*   如果用法比较复杂，那么`pickle`模块的功能也许就会出问题。
*   我们可以把内置的`copyreg`模块同`pikle`结合起来使用，以便为旧数据添加缺失的属性值，进行类的版本管理，并给序列化之后的数据提供固定的引入路径。
*   

###  第45条： 应该用datetime模块来处理本地时间，而不是用time模块
协调世界时(Coordinated Universal Time, UTC)是一种标准的时间表述方式，与时区无关。但大多数时候，我们都是通过本地的时区来描述时间。
<br>Python提供了两种时间转换方式。旧的方式是使用内置的`time`模块，这是一种极易出错的方式。而新的方式，则是采用内置的`datetime`模块。该模块的效果非常好，它得益于Python开发者社区所构建的`pytz`软件包。
<br>为了详细了解`datetime`模块的优点和`time`模块的缺点，我们必须熟悉这两个模块的用法。
1.  `time`模块
    在内置的`time`模块中，有个名叫`localtime`的函数，它可以把*UNIX时间戳(UNIX timestamp, 也就是某个UTC时刻距离UNIX计时原点的秒数)*转换为宿主计算机的时区相符的当地时间。
    ```python
    from time import localtime, strftime

    now = 1407694710
    local_tuple = localtime(now)
    time_format = '%Y-%m-%d %H:%M:%S'
    time_str = strftime(time_format, local_tuple)
    print(time_str)
    >>>
    2014-08-11 02:18:30
    ```
    程序通常还需要做反向处理，也就是说，要把用户输入的本地时间，转换为UTC时间。我们可以用`strptime`函数来解析包含时间信息的字符串，然后调用`mktime`函数，将本地时间转换为UNIX时间戳。
    ```python
    from time import mktime, strptime

    time_tuple = strptime(time_str, time_format)
    utc_now = mktime(time_tuple)
    print(utc_now)
    >>>
    1407694710.0
    ```
    如何把某个时区的当地时间转换为另一个时区的当地时间呢？例如，坐飞机从旧金山到达纽约之后，我想知道现在是旧金山的几点钟。
    <br>想通过直接操作`time`、`localtime`和`strptime`函数的返回值来进行时区转换，不是一个好办法。由于时区会随着当地法规而变化，所以手工管理起来太过复杂，在处理全球各个城市的航班起降问题时，显得尤其困难。
    <br>许多操作系统都提供了时区配置文件，如果失去信息发生变化，他们就会自动更新。我们可以在Python程序中借助`time`模块来使用这些时区信息。例如，下面这段代码会以中国标准时间(CST)为标准，把航班从旧金山的起飞时间解析出来。
    ```python
    parse_format = '%Y-%m-%d %H:%M:%S %Z'
    depart_sfo = '2014-05-01 15:45:16 CST'
    time_tuple = strptime(depart_sfo, parse_format)
    time_str = strftime(time_format, time_tuple)
    print(time_str)
    >>>
    2014-05-01 15:45:16
    ```
    可以看到，`strptime`函数可以正确解析CST时间，那么，它是否也能解析电脑所支持的其他时区呢？实际上是不行的。用纽约所在的美国东部夏令时(Eastern Daylight Time, EDT)来实验一下，就会发现，`strptime`抛出了异常。
    ```python
    arrival_nyc = '2014-05-01 23:33:24 EDT'
    time_tuple = strptime(arrival_nyc, time_format)
    >>>
    ValueError: unconverted data remains:  EDT
    ```
    之所以出现出现这些问题，是因为`time`模块需要依赖操作系统而运作。该模块的实际行为，取决于底层的C函数如何与宿主操作系统橡胶护。这种工作方式，使得`time`模块的功能不够稳定。它无法协调地处理各种本地时区，因此，我们应该尽量少用这个模块。如果一定要使用`time`模块，那就只应该用他在UTC与宿主计算机的当地时区之间进行转换。对于其他类型的转换来说，还是使用`datetime`模块比较好。
2.  **datetime模块**
    在内置的`datetime`模块中，有个名叫`datetime`的类，它也能够像刚才所讲的`time`模块那样，用来在Python程序中描述时间。与`time`模块类似，`datetime`可以把UTC格式的当前时间，转换为本地时间。
    <br>下面这段代码，会把UTC格式的当前时间，转换为计算机所用的本地时间：
    ```python
    from datetime import datetime, timezone

    now = datetime(2014, 8, 10, 18, 18, 30)
    now_utc = now.replace(tzinfo=timezone.utc)
    now_local = now_utc.astimezone()
    print(now_local)
    >>>
    2014-08-11 02:18:30+08:00
    ```
    `datetime`模块还可以把本地时间轻松地转换成UTC格式的UNIX时间错。
    ```python
    time_str = '2014-08-10 11:18:30'
    now = datetime.strptime(time_str, time_format)
    time_tuple = now.timetuple()
    utc_now = mktime(time_tuple)
    print(utc_now)
    >>>
    1407640710.0
    ```
    与`time`模块不同的是，`datetime`模块提供了一套机制，能够把某一种当地时间可靠地转换为另外一种当地时间。然而，在默认情况下，我们只能通过`datetime`中的`tzinfo`类及相关方法，来使用这套时区操作机制，因为他并没有提供UTC之外的时区定义。
    <br>所幸Python开发者社区提供了`pytz`模块填补了这一空缺。该模块可以从[Python Package Index](https://pypi.python.org/pypi/pytz/)下载。`pytz`模块带有完整完整的数据库，其中包含了开发者可能会用到的每一种时区定义信息。
    <br>为了有效地使用`pytz`模块，我们总是应该先把当地时间转换为UTC，然后针对UTC值进行`datetime`操作(例如，执行与时区偏移有关的操作)，最后在把UTC转回当地时间。
    <br>例如，我们可以用下面这段代码，把航班到达纽约的时间，转换为UTC格式的`datetime`对象。某些函数调用语句，看上去似乎显得多余，但实际上，为了正确使用`pytz`模块，我们必须编写这些语句。
    ```python
    import pytz
    arrival_nyc = '2014-05-01 23:33:24'
    nyc_dt_naive = datetime.strptime(arrival_nyc, time_format)
    eastern = pytz.timezone('US/Eastern')
    nyc_dt = eastern.localize(nyc_dt_naive)
    utc_dt = pytz.utc.normalize(nyc_dt.astimezone(pytz.utc))
    print(utc_dt)
    >>>
    2014-05-02 03:33:24+00:00
    ```
    得到UTC格式的`datetime`之后，再把它转换成旧金山当地时间。
    ```python
    pacific = pytz.timezone('US/Pacific')
    sf_dt = pacific.normalize(utc_dt.astimezone(pacific))
    print(sf_dt)
    >>>
    2014-05-01 20:33:24-07:00
    ```
    我们还可以把这个时间，轻松地转换成尼泊尔当地时间。
    ```python
    nepal = pytz.timezone('Asia/Katmandu')
    nepal_dt = nepal.normalize(utc_dt.astimezone(nepal))
    print(nepal_dt)
    >>>
    2014-05-02 09:18:24+05:45
    ```
    无论宿主计算机运行何种操作系统，我们都可以通过`datetime`模块和`pytz`模块，在各种环境下协调一致地完成时区转换操作。

#### 要点
*   不要用`time`模块在不同时区之间进行转换。
*   如果要在不同时区之间，可靠地执行转换操作，那就应该把内置的`datetime`模块与开发者社区提供的`pytz`模块搭配起来使用。
*   开发者总是应该先把时间表示成`UTC`格式，然后对其执行各种转换操作，最后再把它转回本地时间。

###  第46条： 使用内置算法与数据结构
如果Python程序要处理的数量比较可观，那么代码的执行速度会受到复杂算法的拖累。然后这并不能证明Python是一门执行速度很低的语言(第41条)，因为这种情况很可能是算法和数据结构选择不佳而导致的。
<br>幸运的是Python的标准程序库里面，内置了各种算法与数据结构，以供开发者使用。这些常见的算法与数据结构，不仅执行速度比较快，而且还可以简化编程工作。其中某些实用工具，是很难由开发者自己正确实现出来的。所以，我们应该直接使用这些Python自带的功能，而不要重新去实现它，以节省时间和精力。
1.  **双向队列**
    <br>`collections`模块中的`deque`类，是一种双向队列(double-ended queue, 双端队列)。从该队列的头部或尾部插入或移除一个元素，只需消耗常数级别的时间。这一特性，使得它非常适合用来表示先进先出(first-in-first-out,FIFO)的队列。
    ```python
    from collections import deque
    fifo = deque()
    fifo.append(1)      # Producer
    fifo.append(2)
    fifo.append(3)
    x = fifo.popleft()  # Consumer
    print(x)
    >>>
    1
    ```
    内置的`list`类型，也可以向队列那样，按照一定的顺序来存放元素。从`list`尾部插入或移除元素，也仅仅需要常数级别的时间。但是，从`list`头部插入或移除元素，却会耗费线性级别的时间，这与`deque`的常数级时间相比，要慢的多。
2.  **有序字典**
    <br>标准的字典是无序的。也就是说，在拥有相同键值对的两个`dict`上面迭代，可能会出现不同的迭代顺序。标准的字典之所以会出现这种奇怪的现象，是由其快速哈希表(fast hash table)的实现方式而导致的。
    ```python
    a = {}
    a['foo'] = 1
    a['bar'] = 2
    from random import randint

    # Randomly populate 'b' to cause hash conflicts
    while True:
        z = randint(99, 1013)
        b = {}
        for i in range(z):
            b[i] = i
        b['foo'] = 1
        b['bar'] = 2
        for i in range(z):
            del b[i]
        if str(b) != str(a):
            break

    print(a)
    print(b)
    print('Equal?', a == b)
    >>>
    {'foo':1, 'bar':2}
    {'bar':2, 'foo':1}
    Equal?True
    ```
    `collections`模块中的`OrderedDict`类，是一种特殊的字典，它能够按照键的插入顺序，来保留键值对在字典中的次序。在`OrderedDict`上面根据键来迭代，其行为是确定的。这种确定的行为，可以极大地简化测试与调试工作。
    ```python
    from collections import OrderedDict
    a = OrderedDict()
    a['foo'] = 1
    a['bar'] = 2

    b = OrderedDict()
    b['foo'] = 'red'
    b['bar'] = 'blue'

    for value1, value2 in zip(a.values(), b.values()):
        print(value1, value2)

    >>>
    1 red
    2 blue
    ```
3.  **带有默认值的字典**
    <br>典可用来保存并记录一些统计数据。但是，由于字典里面未必有我们要查询的那个键，所以在用字典保存计数器的时候，就必须要用稍微麻烦一些的方式，才能够实现这种简单的功能。
    ```python
    stats = {}
    key = 'my_counter'
    if key not in stats:
        stats[key] = 0

    stats[key] += 1
    print(stats)
    >>>
    {'my_counter': 1}
    ```
    我们可以用`collections`模块中的`defaultdict`类来简化上述代码。如果字典里面没有待访问的键，那么它就会把某个默认值与这个键自动关联起来。于是，我们只需提供返回默认值的函数即可，字典会用该函数为每一个默认的键指定默认值。在本例中，我们使用内置的`int`函数来创建字典，这使得该字典能够以0为默认值(第23条)。现在，计数器的实现代码就变得非常简单了。
    ```python
    from collections import defaultdict
    stats = defaultdict(int)
    stats['my_counter'] += 1
    print(dict(stats))
    >>>
    {'my_counter': 1}
    ```
4.  **堆队列(优先级队列)**
    <br>堆(heap)是一种数据结构，很适合用来实现优先级队列。`heapq`模块提供了`heappush`、`heappop`和`nsmallest`等一些函数，能够在标准的`list`类型之中创建堆结构。
    <br>各种优先级的元素，都可以按任意顺序插入堆中。
    ```python
    from heapq import *
    a = []
    heappush(a, 5)
    heappush(a, 3)
    heappush(a, 7)
    heappush(a, 4)
    ```
    这些元素总是会按照优先级从高到低的顺序，从堆中弹出(数值较小的元素，优先级较高)。
    ```python
    print(heappop(a), heappop(a), heappop(a), heappop(a))
    >>>
    3 4 5 7
    ```
    用`heapq`把这样的`list`制作好之后，我们可以在其他场合使用它。只要访问堆中下标为0的那个元素，就总是能够查出最小值。
    ```python
    a = []
    heappush(a, 5)
    heappush(a, 3)
    heappush(a, 7)
    heappush(a, 4)
    assert a[0] == nsmallest(1, a)[0] == 3
    ```
    在这种`list`上面调用`sort`方法之后，该`list`依然能够保持堆的结构。
    ```python
    print('Before:', a)
    a.sort()
    print('After: ', a)
    >>>
    Before: [3, 4, 7, 5]
    After:  [3, 4, 5, 7]
    ```
    这些`heapq`操作所耗费的时间，与列表长度的对数成正比。如果在普通的Python列表上面执行相关操作，那么将会耗费线性级别的时间。
5.  **二分查找**
    <br>在`list`上面使用`index`方法来搜索某个元素，所耗的时间会与列表的长度呈线性比例。
    ```python
    x = list(range(10**6))
    i = x.index(991234)
    print(i)
    >>>
    991234
    ```
    `bisect`模块中的`bisect_left`等函数，提供了高效的二分折半搜索算法，能够在一系列排好顺序的元素之中搜寻某个值。由`bisect_left`函数所返回的索引，表示待搜寻的值在序列中的插入点。
    ```python
    from bisect import bisect_left
    i = bisect_left(x, 991234)
    print(i)
    >>>
    991234
    ```
    二分搜索算法的复杂度，是对数级别的。这就意味着，用`bisect`来搜索包含一百万个元素的列表，与用`index`来搜索包含14个元素的列表，所耗的时间差不多。由此可见，这种对数级别的算法，要比线性级别的算法块很多。
6.  **与迭代器有关的工具**
    <br>内置的`itertools`模块中，包含大量的函数，可以用来组合并操控迭代器(在交互式的Python界面输入`help(itertools)`查看详细信息)。
    <br>`itertools`函数分为三大类：
    *   能够把迭代器链接起来的函数：
        *   `chain`：将多个迭代器按顺序连成一个迭代器
        *   `cycle`：无限地重复某个迭代器中的各个元素
        *   `tee`：把一个迭代器拆分成多个平行的迭代器
        *   `zip_longest`：与内置的`zip`函数相似，但是它可以应对长度不同的迭代器
    *   能够从迭代器中过滤元素的函数：
        *   `islice`：在不进行复制的前提下，根据索引值来切割迭代器
        *   `takewhile`：在判定函数(predicate function,谓词函数)为`True`的时候，从迭代器中逐个返回元素
        *   `dropwhile`：从判定函数初次为`False`的地方开始，逐个返回迭代器中的元素
        *   `filterfalse`：从迭代器中逐个返回能令判定函数为`False`的所有元素，其效果与内置的`filter`函数相反
    *   能够把迭代器中的元素组合起来的函数：
        *   `product`：根据迭代器中的元素计算笛卡尔积(Cartesian product),并将其返回。可以用`product`来改写深度嵌套的列表推导操作
        *   `permutations`：用迭代器中的元素构建长度为*N*的各种有序排列，并将所有排序形式返回给调用者
        *   `combination`：用迭代器中的元素构建长度为*N*的各种无序组合，并将所有组合形式返回给调用者

    除了上面这些，`itertools`模块里面还有其他一些函数及教程。如果你发现自己要编写一段非常麻烦的迭代程序，那就应该先花些时间来阅读`itertools`的文档，看看里面有没有现成的工具可供使用。

#### 要点
*   我们应该用Python内置的模块来描述各种算法和数据结构。
*   开发者不应该自己去重新实现那些功能，因为我们很难把它写好。

###  第47条： 在重视精确度的场合，应该使用decimal
Python语言很适合用来编写与数值型数据打交道的代码。Python的整数类型，可以表达任意长度的值，其双精度浮点数类型，也遵循IEEE 754标准。此外，Python还提供了标准的复数类型，用来表示虚数值。然而这些数值类型，并不能覆盖每一种情况。
<br>例如，要根据通话时长和费率，来计算用户拨打国际长途电话所应支付的费用。加入用户打了3分42秒，从美国打往南极洲的电话，每分钟1.45美元，那么，这次通话的费用是多少呢？我们可能认为，只要使用浮点数就能算出合理的结果。
```python
rate = 1.45
seconds = 3*60 + 42
cost = rate * seconds / 60
print(cost)
>>>
5.364999999999999
```
但是，把计算结果向分位取整之后，却发现，`round`函数把分位右侧的那些数字完全舍去了。实际上，不足1分钱的部分，是应该按1分钱收取的，我们希望`round`函数把该值上调为5.37,而不是下调为5.36。
```python
print(round(cost, 2))
>>>
5.36
```
假设我们还要对那种通话时长很短，而且费率很低的电话呼叫进行计费。下面这段代码，按照每分钟0.05美元的费率，来计算长度为5秒的通过费用：
```python
rate = 0.05
seconds = 5
cost = rate * seconds / 60
print(cost)
>>>
0.004166666666666667
```
由于计算出来的数值很小，所以`round`函数会把它下调为0，而这当然不是我们想要的结果。
```python
print(round(cost, 2))
>>>
0.0
```
内置的`decimal`模块中，有个`Decimal`类，可以解决上面那些问题。该类默认提供28个小数位，以进行定点(fixed point)数学运算。如果有需要，还可以把精确度调得更高一些。`Decimal`类解决了IEEE 754浮点数所产生的精度问题，而且开发者还可以更加精准地控制该类的舍入行为。
<br>例如，我们可以把刚才计算美国与南极洲长途电话费的那个程序，用`Decimal`类改写。改写之后，程序会算出精确的结果，而不会像原来那样，只求出近似值。
```python
from decimal import Decimal
from decimal import ROUND_UP
rate = Decimal('1.45')
seconds = Decimal('222')  # 3*60 + 42
cost = rate * seconds / Decimal('60')
print(cost)
>>>
5.365
```
`Decimal`类提供了一个内置的函数，它可以按照开发者所要求的精度及舍入方式，来准确地调整整数值。
```python
rounded = cost.quantize(Decimal('0.01'), rounding=ROUND_UP)
print(rounded)
>>>
5.37
```
这个`quantize`方法，也能对那种时长很短、费用很低的电话，正确地进行计费。我们用`Decimal`类来改写之前的那段代码。改写之后，计算出来的电话费用还是不足1分钱：
```python
rate = Decimal('0.05')
seconds = Decimal('5')
cost = rate * seconds / Decimal('60')
print(cost)
>>>
0.004166666666666666666666666667
```
但是，我们可以在调用`quantize`方法时，指定合理的舍入方式，从而确保该方法能够把不足1分钱的部分，上调为1分钱。
```python
rounded = cost.quantize(Decimal('0.01'), rounding=ROUND_UP)
print(rounded)
>>>
0.01
```
虽然`Decimal`类很适合执行定点数的运算，但他在精确度方面仍有局限，例如，1/3这个数，就只能用近似值来表示。如果要用精度不受限制的方式来表达有理数，那么可以考虑使用`Fraction`类，该类包含在内置的`fractions`模块里。

#### 要点
*   对于编程中可能用到的每一种数值，我们都可以拿对应的Python内置类型，或内置模块中的类表示。
*   `Decimal`类非常适合用在那种对精度要求很高，且对舍入行为要求很严的场合，例如：设计货币计算的场合。

###  第48条： 学会安装由python开发者社区所构建的模块

#### 要点
*   pip


##   第七章 习作开发
###  第49条： 为每个函数，类和模块编写文档字符串
###  第50条： 用包来安装模块，并提供稳固的API
###  第51条： 为自编的模块定义根异常，以便将调用者与API隔离
###  第52条： 用适当的方式打破循环依赖关系
###  第53条： 用虚拟环境隔离项目，并重建其依赖

##   第八章 部署
###  第54条： 考虑用模块级别的代码来配置不用的部署环境
###  第55条： 通过repr字符串来输出调试信息
###  第56条： 用unittest来测试全部代码
###  第57条： 考虑用pdb实现交互调试
###  第58条： 先分析性能，然后再优化
###  第59条： 用tracemalloc来掌握内存的使用及泄漏情况