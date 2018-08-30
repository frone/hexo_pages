---
title: PulP线性优化（三）python编码 
date: 2018-08-03 18:23:51
tags:
    - PULP
    - 线性优化
categories: 算法
---
*本文根据PuLP文档翻译而来，原文请参考*
*https://pythonhosted.org/PuLP/main/basic_python_coding.html*

### 基本的Python编码
___
在本课程中，您将学习Python中的基本编程，但也可以在Internet上免费获得优秀的Python语言参考资料。您可以下载[Dive Into Python](http://www.diveintopython.org/)这本书， 或者 在Python网站上有许多Python [初学者指南](http://wiki.python.org/moin/BeginnersGuide)。点击以下链接：

- [BeginnersGuide /非程序员](http://wiki.python.org/moin/BeginnersGuide/NonProgrammers)
- [BeginnersGuide /程序员](http://wiki.python.org/moin/BeginnersGuide/Programmers)

取决于您当前的编程知识水平。下面的代码部分假定了基本编程原理的知识，并主要关注特定于Python编程的语法。

注意：>>>表示Python命令行提示符。

### Python中的循环
___
#### for循环
一般格式是：
```
为 变量 在 序列：
    #some命令
#other for循环之后的命令
```
请注意，格式（缩进和新行）控制for循环的结束，而循环的开头是冒号：。

观察下面的循环，这类似于您将在课程中使用的循环。变量i通过字符串列表依次变为每个字符串。顶部是.py文件中的代码，底部显示输出
```
＃以下代码演示了一个列表，其中包含字符串
ingredientslist  =  [ “Rice” ，“Water” ，“Jelly” ] 
for  i  in  ingredientslist ：
    print  i 
print  “不再在循环中”
```
输出
```
Rice
Water
Jelly
No longer in the loop
```
#### while循环
这些类似于for循环，除了它们继续循环，直到指定的条件不再为真。没有告诉while循环通过任何特定的序列。
```
i = 3
while i <= 15:
    # some commands
    i = i + 1 # a command that will eventually end the loop is naturally
    required
# other commands after while loop
```
对于这个特定的简单while循环，最好做一个for循环，但它演示了语法。如果循环之前的迭代次数需要结束，while循环是有用的，是未知的。

#### if语句
这与上面的循环非常相似。键标识符是冒号：启动语句和缩进结束以结束它。

```
if j in testlist:
    # some commands
elif j == 5:
    # some commands
else:
    # some commands
```
这里显示“elif”（else if）和“else”也可以在if语句之后使用。事实上，“else”可以在两个循环之后以相同的方式使用。

### python中的数组类型
___
#### 列表
列表只是一组变量组合在一起。范围函数通常用于创建整数列表，具有范围的一般格式（开始，停止，步骤）。start的默认值为0，步骤的默认值为1。

```
>>> range(3,8)
[3,4,5,6,7]
```
这是一个列表/序列。除了整数之外，列表中还可以包含字符串或整数，浮点数和字符串。它们可以通过循环（如下一节所示）或通过显式创建（如下所示）创建。请注意，print语句将向用户显示字符串/变量/ list / ....
```
>>> a = [5,8,"pt"]
>>> print a
[5,8,'pt']
>>> print a[0]
5
```
#### 元组
元组与列表基本相同，但重要的区别在于它们一旦创建就无法修改。它们由以下人员分配：

>>> X  =  （4 ，1 ，8 ，“字符串” ，[ 1 ，0 ]，（“J” ，4 ，“○” ），14 ）
元组可以在其中包含任何类型的数字，字符串，列表，其他元组，函数和对象。另请注意，元组中的第一个元素编号为元素“零”。访问此数据的方法是：

>>> x [ 0 ] 
4 
>>> x [ 3 ] 
“string”
#### 字典
字典是每个具有关联数据的引用键列表，其中该顺序根本不影响字典的操作。对于字典，键不是连续的整数（与列表不同），而是可以是整数，浮点数或字符串。这将变得清晰：

>>> x  =  {}  ＃创建一个新的空字典 - 注意表示创建字典的大括号
>>> x [ 4 ]  =  “编程”  ＃字符串“programming”被添加到字典x中，“ 4“因为它是参考
>>> x [ ”游戏“ ]  =  12 
>>> 打印 x [ ”游戏“ ] 
12
在字典中，引用键和存储的值可以是任何类型的输入。新词典元素在创建时添加（使用列表，您无法访问或写入列表中超出最初定义的列表维度的位置）。

成本 =  { “鸡肉” ： 1.3 ， “牛肉” ： 0.8 ， “羊肉” ： 12 } 
打印 “肉类的成本” 
为 我 在 成本：
    打印 我
    打印 成本[ 我] 
成本[ “LAMB” ]  =  5 
打印 “更新肉类成本“ 
对于 我 的 成本：
    打印 我的
    打印 成本[ i ]
给

成本 的 肉类
鸡肉
1.3 
羊肉
12和
牛肉
0.8 
更新 成本 的 肉类
LAMB 
5 
鸡
1.3 
羊肉
12 
牛肉
0.8
在上面的示例中，使用大括号和冒号创建字典以表示将数据分配给字典键。变量i依次分配给每个键（与列表中的变量相同）

>>> 用于 我 在 范围（1 ，10 ）
）。然后使用此键调用字典，并返回存储在该键名下的数据。使用词典的这些类型的for循环与使用PuLP在本课程中对LP进行建模高度相关。

### List / Tuple / Dictionary语法
注意创建一个：

列表用方括号[];
元组用圆括号和逗号（，）完成;
字典是用括号{}完成的。
但是，在创建之后，当访问list / tuple / dictionary中的元素时，操作总是用方括号执行（即a [3]？）。如果a是列表或元组，则返回第四个元素。如果a是字典，它将返回使用引用键3存储的数据。

### 列表生成式
Python支持List Comprehensions，这是一种快速而简洁的方法，可以在不使用多行的情况下创建列表。在简单的情况下，它们很容易理解，并且您将在本课程的代码中使用它们。

>>> a  =  [ i  for  i  in  range （5 ）] 
>>> a 
[0,1,2,3,4]
上面的语句将创建列表[0,1,2,3,4]并将其分配给变量“a”。

>>> 赔率 =  [ i  for  i  in  range （25 ） if  i ％2 == 1 ] 
>>> 赔率
[1,3,5,7,9,11,13,15,17,19,21,23 ]
上面的语句使用if语句和模数运算符（％），因此列表中只包含奇数：[1,3,5，...，19,21,23]。（注意：模数运算符从整数除法计算余数。）

>>> fifths = [i for i in range(25) if i%5==0]
>>> fifths
[0, 5, 10, 15, 20]
This will create a list with every fifth value in it [0,5,10,15,20]. Existing lists can also be used in the creation of new lists below:

>>> a = [i for i in range(25) if (i in odds and i not in fifths)]
Note that this could also have been done in one step from scratch:

>>> a = [i for i in range(25) if (i%2==1 and i%5==0)]
For a challenge you can try creating

a list of prime numbers up to 100, and
a list of all “perfect” numbers.
More List Comprehensions Examples

Wikipedia: Perfect Numbers.

### Other important language features
---
#### Commenting in Python
使用“”“开始并结束评论部分，在文件顶部进行评论。整个代码中的注释是使用行开头的hash＃符号完成的。

#### 导入语句
在您打算使用PuLP进行建模的所有Python编码的顶部，您将需要import语句。此语句使您当前正在编写的模块中的另一个模块（程序代码文件）的内容可用，即您将需要调用的pulp.py中定义的函数和值可用。在本课程中，您将使用：

>>> 来自 纸浆 进口 *
星号表示您正在从纸浆模块中导入所有名称。现在可以调用在pulp.py中定义的函数，就好像它们是在您自己的模块中定义的那样。

#### functions
Python中的函数定义如下：（def是define的缩写）

高清 名（inputparameter1 ， inputparameter2 ， 。 。 。）：
    #function体
对于一个真实的例子，请注意，如果在函数定义中为输入分配了一个值，这是默认值，并且仅在没有传入其他值时才使用。输入参数的顺序（在定义中）不无论如何，只要调用该函数，就会以相应的顺序输入位置参数。如果使用关键字，参数的顺序根本不重要：

def  string_appender （head = 'begin' ， tail = 'end' ， end_message = 'EOL' ）：
    result  =  head  +  tail  +  end_message 
    返回 结果
>>> string_appender （'newbegin' ， end_message  =  'StringOver' ）
newbeginendStringOver
在上面的示例中，将打印函数调用的输出。head的默认值是'begin'，但是使用了'newbegin'的输入。使用了'end'尾部的默认值。并使用endmessage的输入值。请注意，必须将end_message指定为关键字参数，因为没有给出tail的值

#### 类
要演示类在Python中的工作方式，请查看以下类结构。

类名是Pattern，它包含几个与Pattern类的任何实例（即Pattern）相关的类变量。功能是

__在里面__
函数，它创建Pattern类的实例，并使用self分配name和lengthsdict的属性。
__str__
函数定义了打印类实例时要返回的内容。
修剪
函数就像任何普通函数一样，除了所有类函数之外，self必须在输入括号中。
类 模式：
    “”， “ 
    上在SpongeRoll问题的特定图案的信息
    ”“” 
    成本 =  1 
    trimValue  =  0.04 
    totalRollLength  =  20 
    lenOpts  =  [ 5 ， 7 ， 9 ]

def  __init__ （self ，name ，lengths  =  None ）：
    self 。name  =  name 
    self 。lengthsdict  =  字典（拉链（自我。lenOpts ，长度））

def  __str __ （self ）：
    返回 自我。名称

def  trim （self ）：
返回 模式。totalRollLength  -  总和（[ INT （我）* 自我。lengthsdict [ 我]  为 我 在 自我。lengthsdict ]）
这个类可以用如下：
```
>>> Pattern.cost # The class attributes can be accessed without making an instance of the class
1
>>> a = Pattern("PatternA",[1,0,1])
>>> a.cost # a is now an instance of the Pattern class and is associated with Pattern class variables
1
>>> print a # This calls the Pattern.__str__() function
"PatternA"
>>> a.trim() # This calls the Pattern.trim() function. Note that no input is required.
```

函数定义中的self是隐含的输入