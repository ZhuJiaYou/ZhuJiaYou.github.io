---
title: Python编译Python函数/方法定义与类定义的区别
date: 2019-05-26 10:36:35
tags:
  - Python
categories: Python
---
## 1. 引言
最近在写Python程序时遇到了一个奇怪的问题。
本文中使用Python3(3.5.3)测试。
（1）定义两个类C1、C2，在C1的定义中使用后定义的C2的对象，会报错，因为C1定义时还没有定义C2：
<!--more-->
```python
class C1:
	def __init__(self, ac=C2()):
		print("Hello, I'm an object of C1")


class C2:
	def __init__(self):
		print("Hello, I'm an object of C2")

"""
运行后报错：
Traceback (most recent call last):
  File "test.py", line 1, in <module>
    class C1:
  File "test.py", line 2, in C1
    def __init__(self, ac=C2()):
NameError: name 'C2' is not defined
"""
```
这比较容易理解，因为C2还没有定义，所以在C1的定义中使用时，会报未定义错误。
（2）然而，定义两个函数f1、f2，在f1的定义中使用后定义的f2，则顺利通过：
```python
def f1():
	f2()


def f2():
	print("Hi, I'm f2!")

"""
运行后程序顺利通过，未报告错误。
"""
```
这是为什么呢？同样是引用未定义的对象(Python中函数是一等对象)，类的定义中会报错，函数的定义中就不会。这里的原因主要在于Python在**编译**函数和编译类时的不同处理方式。
## 2. 我们都说Python是解释型语言，那么Python有编译过程吗？
答案是”当然有“。
（1）Python其实和Java/C#相似，也有一个虚拟机(解释器)。我们在运行.py文件时，Python虚拟机先把.py文件编译为PyCodeObject。而我们经常看到的.pyc文件就是内存中的PyCodeObject持久化到硬盘中的形式。
（2）我们平时写程序时发现有时候有.pyc文件有时候没有。因为Python解释器默认只是把我们可能重用到的模块持久化成pyc文件。比如说，我们写了一个模块，在别的Python文件引入(import)它的时候，就会将PyCodeObject持久化成.pyc文件，方便下次调用。
（3）加载模块时，如果同时存在.py和.pyc，Python会尝试使用.pyc，如果.pyc的编译时间早于.py的修改时间，则重新编译.py并更新.pyc。
（4）Python中有一个内置函数compile()，可以将源文件编译成codeobject，首先看这个函数的说明：

　　*compile(...) compile(source, filename, mode[, flags[, dont_inherit]]) -> code object
　　
  参数1：源文件的内容字符串
  参数2：源文件名称
  参数3：exec-编译module，single-编译一个声明，eval-编译一个表达式 一般使用前三个参数就够了*
　　
使用示例：
```python
# src.py
def f(a=0):
    b = 1
    print("Hi, I'm f!")

c = 2
d = 3
f()

# run.py
with open("src.py") as f:
	str_src = f.read()
co = compile(str_src,'src','exec')
print(type(co))
print(co.co_names)  # 所有的符号名称
print(co.co_name)  # 模块名、函数名、类名
print(co.co_consts)  # 常量集合、函数f和两个int常量a,b，d
print(co.co_consts[1].co_varnames)  # 可以看到f函数也是一个PyCodeObject,打印f中的局部变量
print(co.co_consts[1].co_firstlineno)  # 代码块在文件中的起始行号
print(co.co_stacksize)  # 代码栈大小
print(co.co_filename)  # 文件名

"""
运行run.py的结果：
<class 'code'>
('f', 'c', 'd')
<module>
(0, <code object f at 0x7fa26309e9c0, file "src", line 1>, 'f', 2, 3, None)
('a', 'b')
1
3
src
"""
```
（5）PyCodeObject的co_code代表了字节码，这个字节码有什么含义？我们可以使用dis模块进行python的反编译：
```python
# 续run.py
import dis

dis.dis(co)

"""
输出结果：
  1           0 LOAD_CONST               0 (0)
              3 LOAD_CONST               1 (<code object f at 0x7fe65d0da8a0, file "src", line 1>)
              6 LOAD_CONST               2 ('f')
              9 MAKE_FUNCTION            1
             12 STORE_NAME               0 (f)

  5          15 LOAD_CONST               3 (2)
             18 STORE_NAME               1 (c)

  6          21 LOAD_CONST               4 (3)
             24 STORE_NAME               2 (d)

  7          27 LOAD_NAME                0 (f)
             30 CALL_FUNCTION            0 (0 positional, 0 keyword pair)
             33 POP_TOP
             34 LOAD_CONST               5 (None)
             37 RETURN_VALUE

"""
```
由反编译结果来看，python字节码其实是模仿的x86的汇编。编译之后，解释器再对PyCodeObject解释执行。
输出编译结果各列含义：
第一列：行号
第二列：指令在代码块中的偏移量
第三列：指令
第四列：操作数
第五列：操作数说明
## 3. Python函数的定义与执行
当Python解释器遇到.py文件中的函数定义时，会将其编译为一个PyCodeObject。
在[Python文档关于函数定义](https://docs.python.org/3/reference/compound_stmts.html#grammar-token-funcdef)有：

*函数定义是一个可执行语句。它的执行将当前本地命名空间中的函数名绑定到函数对象（函数可执行代码的包装器）。此函数对象包含对当前全局命名空间的引用，作为调用函数时要使用的全局命名空间。
**函数定义不执行函数体；只有在调用函数时才会执行。***

也就是说，Python在遇到函数定义时不会执行该函数，它只会将该函数编译成一个对象，该对象稍后可用于执行。python执行函数的唯一时刻是调用函数的时刻。
由于函数定义与执行的区别，在调用函数之前，Python无法验证函数中的变量是否已定义。因此，可以在函数体中使用当前不存在的名称。只要在调用函数时定义了名称，python就不会引发错误。
比如：
```python
>>> def f():
...     return a + b
...
```
我们可以看到，Python解释器没有报错。这是因为它只是简单地编译了f。它没有执行函数，所以不知道a和b没有定义。
我们反汇编f的PyCodeObject：
```python
>>> from dis import dis
>>> dis(f)
  2           0 LOAD_GLOBAL              0 (a)
              3 LOAD_GLOBAL              1 (b)
              6 BINARY_ADD
              7 RETURN_VALUE
>>>
```
python在PyCodeObject中编码了两个LOAD_GLOBAL指令。指令的参数分别是变量名a和b。
这说明，python在编译函数时确实看到了我们试图引用两个变量，并创建了字节码指令来执行此操作。但在调用函数之前，它不会执行指令。
让我们看看当我们试图通过调用f来执行PyCodeObject时会发生什么：
```python
>>> f()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in f
NameError: name 'a' is not defined
>>>
```
可以看到，python引发了一个NameError。这是因为它试图执行两个LOAD_GLOBAL指令，但发现全局范围内未定义的名称。
现在让我们看看如果在**调用**func之前定义a和b会发生什么：
```python
>>> a = 1
>>> b = 5
>>> f()
6
>>>
```
可以看到，这时就不报错了。当python执行f的字节代码(PyCodeObject)时，它能够找到全局变量a和b，并使用这些变量来执行函数。
## 4. Python解释器对类定义的处理
python处理的类定义与函数定义稍有不同。
在[Python文档关于类定义](https://docs.python.org/3/reference/compound_stmts.html#grammar-token-classdef)：

*类的套件，使用新创建的**本地命名空间**和**原始全局命名空间**，在新的执行框架（参见命名和绑定）中执行。（通常，该套件主要包含函数定义。）当类的套件完成执行时，它的执行框架将被丢弃，但它的本地命名空间将被保存。*

当Python遇到一个类定义时，它将为类创建一个与函数相似的PyCodeObject。但是，Python还允许类具有**在类定义期间执行的名称空间(namespace)**。Python立即执行类命名空间，因为类中已定义的任何变量都应属于该类。因此，类名称空间内使用的任何名称必须已定义，以便在类定义期间使用。

比如：
```python
>>> class C1:
...     print("Hello, I'm one of C1")
...
Hello, I'm one of C1
>>>
```
我们可以看到，Python解释器遇到类定义时，会在类定义期间执行的名称空间立即执行其中的可执行语句。
但是请**注意**，这并不适用于类方法。Python对待方法中未定义的名称和函数一样，允许在定义方法时使用未定义名称：
```python
>>> class C2:
...     def m1(self):
...             return v
...
>>> v = 100
>>> c1 = C2()
>>> c1.
>>> c1.m1()
100
>>>
```
## 5. 结语
都说Python简单，其实若要是深究起来还是很有难度的。
## 6. 主要参考
[1] https://blog.csdn.net/june_young_fan/article/details/79755392
[2] https://blog.csdn.net/helloxiaozhe/article/details/78104975
[3] https://blog.csdn.net/sxb0841901116/article/details/21418885
[4] https://stackoverflow.com/questions/45042273/why-can-i-use-a-variable-in-a-function-before-it-is-defined-in-python
[5] https://docs.python.org/3.6/reference/datamodel.html?highlight=data%20model
[6] https://docs.python.org/3/reference/compound_stmts.html

