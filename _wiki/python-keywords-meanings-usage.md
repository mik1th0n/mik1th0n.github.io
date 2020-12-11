---
layout: wiki
title: Python 关键字、含义及其用法
categories: Python
description: Python的关键字介绍，含义解释及其用法
keywords: Python,Keywords
---

### 0x00  Python关键字及含义

| 关键字     | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| `False`    | 布尔类型的值，表示假，与`True`相对                           |
| `None`     | 表示什么也没有，自己的数据类型-`NoneType`                    |
| `True`     | 布尔类型的值，表示真，与`False`相反                          |
| `and`      | 用于表达式运算，逻辑与操作                                   |
| `as`       | 用于类型转换                                                 |
| `assert`   | 断言，用于判断变量或者条件表达式的值是否为真                 |
| `break`    | 中断循环语句的执行                                           |
| `class`    | 用于定义类                                                   |
| `continue` | 跳出本次循环，继续执行下一次循环                             |
| `def`      | 用于定义函数或方法                                           |
| `del`      | 删除变量或序列的值                                           |
| `elif`     | 条件语句，与`if`、`else`结合使用                             |
| `else`     | 条件语句，与`if`、`else`结合使用 ，也可用于异常和循环语句    |
| `except`   | 包含捕获异常后的操作代码块，与`try`、`finally`结合使用       |
| `finally`  | 用于异常语句，出现异常后，始终要执行`finally`包含的代码块，与`try`、`except`结合使用 |
| `for`      | `for`循环语句                                                |
| `from`     | 用于导入模块，与`import`结合使用                             |
| `global`   | 定义全局变量                                                 |
| `if`       | 条件语句，与`elif`、`else`结合使用                           |
| `import`   | 用于导入模块，与`from`结合使用                               |
| `in`       | 判断变量是否在序列中                                         |
| `is`       | 判断变量是否为某个类的实例                                   |
| `lambda`   | 定义匿名函数                                                 |
| `nonlocal` | 用于标识外部作用域的函数                                     |
| `not`      | 用于表达式运算，逻辑非操作                                   |
| `or`       | 用于表达式运算，逻辑或操作                                   |
| `pass`     | 空的类、方法或函数的占位符                                   |
| `raise`    | 异常抛出操作                                                 |
| `return`   | 用于从函数返回计算结果                                       |
| `try`      | `try`包含可能会出现异常的语句，与`except`、`finally`结合使用 |
| `while`    | while循环语句                                                |
| `with`     | 简化Python的语句                                             |
| `yield`    | 用于从函数依次返回值                                         |

### 0x01  Python关键字用法

#### 0x01.0 `and`  `or`

- and , or 为逻辑关系用语，Python具有短路逻辑，False and 返回 False 
- 不执行后面的语句， True or 直接返回True，不执行后面的语句 

#### 0x01.1 `del` 
- 删除变量

```python
if __name__=='__main__':
    a=1       # 对象 1 被 变量a引用，对象1的引用计数器为1
    b=a       # 对象1 被变量b引用，对象1的引用计数器加1
    c=a       #1对象1 被变量c引用，对象1的引用计数器加1
    del a     #删除变量a，解除a对1的引用
    del b     #删除变量b，解除b对1的引用
    #print a   #运行此句出错，name 'a' is not defined，说明 del 删除变量a
    print(c)  #最终变量c仍然引用1
    print c
```

- 而列表本身包含的是变量，例：

```python
list = [1,2,3]
# 包含list[0],list[1],list[2]
# 并不包含数字1,2,3
```

- 所以

```python
if __name__=='__main__':
    li=[1,2,3,4,5]  #列表本身不包含数据1,2,3,4,5，而是包含变量：li[0] li[1] li[2] li[3] li[4] 
    first=li[0]     #拷贝列表，也不会有数据对象的复制，而是创建新的变量引用
    del li[0]  # 列表本身包含的是变量，del 删除的是变量。

    print li     #输出[2, 3, 4, 5]
    print(first)   #输出 1
```

#### 0x01.2 `from`

- from引用模块时会用到，例：

```python
from sys import argv
# 从sys中导入argv
from sys import *
# 将sys中所有东西都导入
import sys 
# 导入sys，当需要sys中内容时，需sys.argv而from sys import *
#不用每次都重复输入'sys.'
```

#### 0x01.3 `golbal`

- golbal为全局变量，但当单个函数中出现同一变量名时，在单个函数中为局部变量

```python
golbal q
q = 66
print "q=", q #q = 66
def function():
    q = 3
    print 'q =',q
function() # q = 3
print 'q =',q # q = 66
```

#### 0x01.4 `with`

- with被用来处理异常

```python
# 不用with 处理文件异常
file = open("/tmp/foo.txt")
try:
    data = file.read()
finally:
    file.close()
```

```python
# 用with
with open("/tmp/foo.txt")
 as file:
    data = file.read()
```

- 紧跟with后面的语句被求值后，返回对象的**enter**()方法被调用，这个方法的返回值将被赋值给as后面的变量，此处为file 
- 当with后面的代码块全部被执行完后，将调用前面返回对象的**exit()**方法

```python
#with 的工作      
class Sample:
    def __enter__(self):
        print "In __enter__()"
        return "Foo"

    def __exit__(self, type, value, trace):
        print "In __exit__()"


def get_sample():
    return Sample()

with get_sample() as sample:
    print "sample:", sample
#1. __enter__()方法被执行
#2. __enter__()方法返回的值 - 这个例子中是"Foo"，赋值给变量'sample'
#3. 执行代码块，打印变量"sample"的值为 "Foo"
#4. __exit__()方法被调用
```

```python
#with真正强大之处是它可以处理异常。
#可能你已经注意到Sample类的__exit__方法有三个参数- val, type 和 trace。 
#这些参数在异常处理中相当有用。

class Sample:
    def __enter__(self):
        return self

    def __exit__(self, type,
 value, trace):
        print "type:", type
        print "value:",
 value
        print "trace:",
 trace

    def do_something(self):
        bar = 1/0
        return bar + 10

with Sample() as sample:
    sample.do_something()
```

- 实际上，在with后面的代码块抛出任何异常时，**exit**()方法被执行。 
- 正如例子所示，异常抛出时，与之关联的type，value和stack trace传给**exit**()方法， 
- 因此抛出的ZeroDivisionError异常被打印出来了。 
- 开发库时，清理资源，关闭文件等等操作，都可以放在**exit**方法当中。

#### 0x01.5 `while`  `for…in…`

- 均为循环语句，使用while时要注意成立条件，防止陷入死循环 
- for in 遍历

#### 0x01.6 `assert`

- 断言，声明其布尔值必须为真的判定，如果发生异常就说明表达示为假。 
- 可以理解assert断言语句为raise-if-not，用来测试表示式，其返回值为假，就会触发异常。

```python
assert 1==1
assert 1 == 2# 会报错  Asserterror
assert expression , 'arguments'
#assert 表达式 [, 参数]用来解释断言并更好知道哪里错了
```

#### 0x01.7 `pass`

- pass是空语句，为了保证程序结构的完整性， 
- pass不做任何事情，一般用作 占位语句 
- 当你编写程序部分内容还没想好，可用pass语句占位

```python
def no_idea():
   pass

#实例
for letter in 'python':
    if letter == 'h':
        pass
        print u'这是pass块'
    print u'当前字母：', letter
print 'bye,bye' 
```

#### 0x01.8 `yield`

- yield的意思是生产，返回了一个生成器对象，每个生成器只能使用一次

```python
def h():
    print 'To be brave'
    yield 5

h() #看到某个函数包含了yield，这意味着这个函数已经是一个Generator
#调用h()函数后,print 语句并未执行，执行yield用.next()方法
```

```python
def h():
    print 'Wen Chuan'
    yield 5
    print 'Fighting!'

c = h()
# >>>c.next()# 在IDE 中不用print c.next()，直接c.next()。
# next()语句将恢复Generator执行，并直到下一个yield表达式处
# Wen Chuan 
# 5 
# 当再次运行c.next()时由于没有yield了报错
# >>>c.next()
#  Fighting 
#  Traceback (most recent call last):
#  File "/home/evergreen/Codes/yidld.py", line 11, in <module>
#   c.next()
# StopIteration
```

- 一个带有 yield的函数就是一个generation，他和普通函数不同，生成一个generator看起来像函数调用，但不会执行任何函数代码，直到对其调用.next()（在 for 循环中会自动调用 next()）才开始执行 
- 虽然执行流程仍按函数的流程执行，但每执行到一个 yield 语句就会中断，并返回一个迭代值，下次执行时从 yield 的下一个语句继续执行。看起来就好像一个函数在正常执行的过程中被 yield 中断了数次，每次中断都会通过 yield 返回当前的迭代值。

```python
#使用isgeneratorfunction判断一个函数是否是一个特殊的generator 函数
from inspect import isgeneratorfunction 
isgeneratorfunction(h) 
# True
```

#### 0x01.9 `send()`  `next()`

```python
def h():
    print 'Wen Chuan',
    m = yield 5  # Fighting!
    print m
    d = yield 12
    print 'We are together!'

c = h()
m = c.next()  #m 获取了yield 5 的参数值 5
d = c.send('Fighting!')  #d 获取了yield 12 的参数值12
print 'We will never forget the date', m, '.', d

#send()可以传递yield表达式的值进去，而next()不能传递特定的值，只能传递None进去。
#因此，我们可以看做c.next() 和 c.send(None) 作用是一样的
#注意！！！第一次调用时，请使用next()语句或是send(None)，不能使用send发送一个非None的值，否则会出错的，因为没有yield语句来接收这个值
```

#### 0x01.10 `break`  `contiue`

- Python break语句用来终止循环，用在while和for循环中！！直接跳出 整个 循环 
- 嵌套循环，break语句将停止执行最深层的循环，并开始执行下一行代码

```python
for letter in 'python':# 第一个例子
    if letter == 'h'
        break 
    print u'当期字母：',letter
#输出到'p''y''t'    

var= 10 # 第二个例子
while var > 0:
    print u'当期字母：',var 
    var = var -1
    if var == 5
       break
#输出到6       
print 'bye'
```

- **break是跳出整个循环，continue是跳出当前循环**

```python
#例1
for letter in 'pyhton':
    if letter == 'h':
        continue
    print u'当前字母：', letter
#打印出 pyton

#例2
var = 10
while var > 0:
    var -= 1
    if var == 5:
        continue
    print u'当前字母：', var
#结果 98764321
```

#### 0x01.11 `try` `except` `finally`

```python
try:
<语句>        #运行别的代码
except <名字>：
<语句>        #如果在try部份引发了'name'异常
except <名字>，<数据>:
<语句>        #如果引发了'name'异常，获得附加的数据
else:
<语句>        #如果没有异常发生
```

- 如果当try后的语句执行时发生**异常**，python就跳回到try并执行第一个**匹配**该异常的**except子句**，异常处理完毕，控制流就通过整个try语句（除非在处理异常时又引发新的异常）。 
- 如果在try后的语句里发生了异常，却**没有匹配**的except子句，异常将被递交到上层的try，或者到程序的最上层（这样将结束程序，并打印缺省的出错信息）。 
- 如果在try子句执行时没有发生异常，python将执行else语句后的语句（如果有else的话），然后控制流通过整个try语句。

```python
try:
    try:
        raise NameError
    except TypeError:
        print 'as'
except NameError:
    print 'e'
# e，try后语句raise触发异常，except没有匹配字句，被抛到上层try匹配，print 'e'
```

```python
try:    
     1/0
except Exception , e:    
     print e 
#以上传统的异常处理，加入!!!traceback后会打印出详细的错误信息
import traceback
try:   
      1/0
except Exception: 
     traceback.print_exc()
```

```python
try:
<语句>
finally:
<语句>    #退出try时总会执行
raise

try:
    1 / 0
except Exception as e:
    '''异常的父类，可以捕获所有的异常'''
    print "0不能被除"
else:
    '''保护不抛出异常的代码'''
    print "没有异常"
finally:
    print "最后总是要执行我"
```

#### 0x01.12 `raise`

- 触发异常 
- raise [Exception[,args[,traceback]]] 
- 语句中Exception是异常的类型（例如，NameError）参数是一个异常参数值。 
- 该参数是可选的，如果不提供，异常的参数是”None”。 
- 最后一个参数是可选的（在实践中很少使用），如果存在，是跟踪异常对象。

```python
def mye( level ):
    if level < 1:
        raise Exception("Invalid level!", level)
```

- **raise 触发异常后，后面的代码就不会再执行**

```python
try:
     s = None
     if s is None:
         print "s 是空对象"
         raise NameError     #如果引发NameError异常，后面的代码将不能执行
     print len(s)  #这句不会执行，但是后面的except还是会走到
except TypeError:
     print "空对象没有长度" #由于错误类型并不是TypeError,不执行print

try:
     s = None
     if s is None:
         print u"s 是空对象"
         raise NameError('name is wrong','is')     #如果引发NameError异常，后面的代码将不能执行
     print len(s)  #这句不会执行，但是后面的except还是会走到
except NameError,argvment:
     print u"空对象没有长度",argvment

s = None
if s is None:
    raise NameError

 print 'is here?' #如果不使用try......except这种形式，那么直接抛出异常，不会执行到这里

def mye( level ):
    if level < 1:
        raise Exception("Invalid level!", level)
        # 触发异常后，后面的代码就不会再执行

try:
    mye(0)                # 触发异常
except "Invalid level!":
    print 1
else:
    print 2
```

- **die函数，打印错误信息**

```python
def die(error_massage):
    raise Exception(error_massage)

a = 'wer'
if a == None:
    print 'None'
else:
    die()
```

#### 0x01.13 `exec` `eval` `execfile`

- exec 用来执行储存在字符串或文件中的Python语句 
- **exec**是一条语句将字符串str当成有效的python代码来**执行** 
- eval与execfile是pytho内置函数 
- **eval**(str[globals[locals]])函数将字符串str当成有效的python表达式来**求值**，并提供返回计算值

```python
exec 'print"hello world"'
exec 'a=100'
# 执行后 a = 100
print a #100
eval('3+5')# 8
b = eval('5+6')#eval 返回计算值
print b + 1 #12
```

- execfile(filename)函数可以用来执行文件

```python
execfile(r'F:\learn\ex1.py')
# 若你位于文件所在目录直接执行
execfile(r'ex1.py')
```

- from os.path import exists 
- exists(file)将文件名字符串作为参数，如果文件存在返回True，否则返回False

#### 0x01.14 `return`

- return 是函数返回值

```python
def fun():
    print 'asd'
# fun() 函数没有显示return,默认返回None

def fan(a):
    return a
#有返回值
```

#### 0x01.15 `lambda` `filter` `map` `reduce`

- **lambda** 只是一个表达式，定义了一个匿名函数，起到函数速写的作用 

  - 由于lambda只是一个表达式，它可以直接作为python 列表或python 字典的成员，比如

  ```python
  info = [lambda a:a**3 , lambda b:b**3]
  ```

  ```python
  g = lambda x:x+1
  g(1) #2 等价于 lambda x:x+1(1)
  g(3) #4
  #其中 x 为入口参数，x+1 为函数体
  #用的函数来同样表示
  def g(x):
      return x+1
  
  
  #lambda 也可以用在函数中
  def action(x):
      return lambda y:x+y
  
  a = action(3)# a是action函数的返回值，
  a(22) # 24 ，a(22) ，调用了action返回的lambda表达式
  # 上面函数也可直接写成下式
  b = lambda x:lambda y:x+y
  a = b(3)
  a(2) # 也可直接 (b(3))(2)
  
  
  # lambda 可以一个、多个参数
  g = lambda x:x*2 #one
  print g(3)
  
  m = lambda x,y,z: (x-y)*z # mutiple
  print m(3,1,2)
  
  #lambda 并不会带来程序运行效率的提高，只会使代码更简洁。
  #如果可以使用for...in...if来完成的，坚决不用lambda。
  #如果使用lambda，lambda内不要包含循环，如果有，我宁愿定义函数来完成，
  #使代码获得可重用性和更好的可读性。
  # lambda 是为了减少单行函数的定义而存在的。
  
  
  # --filter(function or None, sequence) -> list, tuple, or string
  # function是一个谓词函数，接受一个参数，返回布尔值True或False。
  # filter函数会对序列参数sequence中的每个元素调用function函数，
  # 最后返回执行结果为True的
  # 返回值的类型和参数sequence（list, tuple, string）的类型相同
  foo = [2, 18, 9, 22, 17, 24, 8, 12, 27]
  
  print filter(lambda x: x % 3 == 0, foo)# filter 是 过滤/筛选 函数
  print[x for x in foo if x % 3==0] #[18, 9, 24, 12, 27] 筛选foo中能被3整除的
  ```

- **map(function, sequence)** 

  - 对sequence中的item 依次执行 function，将执行结果组成list返回 
  - 单个参数

  ```python
  str = ['a', 'b','c', 'd'] 
  
  def fun2(s): 
      return s + ".txt"
  
  ret = map(fun2, str)
  print ret # ['a.txt', 'b.txt', 'c.txt', 'd.txt']
  ```

  - 多个参数，要求函数接受多个参数

  ```python
  def add(x,y):
      return x+y
  
  print map(add,range(5),range(5))
  #[0,2,4,6,8]
  
  foo = [2, 18, 9, 22, 17, 24, 8, 12, 27]
  print map(lambda x: x * 2 + 10, foo) 
  #[14, 46, 28, 54, 44, 58, 26, 34, 64]
  ```

- **reduce(function, sequence, starting_value)** 

  - 对sequence中的item顺序迭代调用function，如果有starting_value， 
  - 还可以作为初始值调用，例如可以用来对List求和

  ```python
  def add1(x,y):
      return x+y
  
  print reduce(add1,range(1,100))
  # 4950 注：1+2+...+99
  print reduce(add1,range(1,100),20)
  # 4970 注：1+2+...+99+20，20为初始值
  ```