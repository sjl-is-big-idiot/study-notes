

声明，初始化，赋值的区别



```java
// 声明
int a; // 类型 对象名
// 声明并初始化
int a = 0; //
// 赋值
a = 10;
```



## 1.1 python中 try...except…finally的执行顺序？

 [python中 try、except、finally 的执行顺序](https://www.cnblogs.com/ybwang/p/4738621.html)

```py
# test1.py
def test1():
  	try:
  		print('to do stuff')
      	raise Exception('hehe')
      	print('to return in try')
      	return 'try'
	except Exception:
      	print('process except')
      	print('to return in except')
      	return 'except'
	finally:
      	print('to return in finally')
      	return 'finally'

test1Return = test1()
print('test1Return : ' + test1Return)
```



```py
# test2.py
def test2():
   	try:
      	print('to do stuff')
      	print('to return in try')
      	return 'try'
 	except Exception:
      	print('process except')
      	print('to return in except')
      	return 'except'
 	finally:
      	print('to return in finally')
      	return 'finally'

test2Return = test2()
print('test1Return : ' + test2Return)
```



从test1.py 和 test2.py 的输出结果得出如下结论：

- 无论是在try还是在except中，遇到return时，只要设定了finally语句，就会中断当前的return语句，跳转到finally中执行，如果finally中遇到return语句，就直接返回，不再跳转回try/excpet中被中断的return语句

```py
# test3.py
def test3():
   	i = 0
 	try:
      	i += 1
 		print('i in try : %s'%i)
      	raise Exception('hehe')
 	except Exception:
      	i += 1
 		print('i in except : %s'%i)
      	return i
    finally:
      	i += 1
 		print ('i in finally : %s'%i )

print('test3Return : %s'% test3())
```



```py
# test4.py
def test4():
   	i = 0
 	try:
      	i += 1
 		return i
   	finally:
      	i += 1
 		print ('i in finally : %s'%i )
print('test4Return : %s' % test4())
```



从 test3.py 和 test4.py 的输出结果得出如下结论：

- 在except和try中遇到return时，会锁定return的值，然后跳转到finally中，如果finally中没有return语句，则finally执行完毕之后仍返回原return点，将之前锁定的值返回(即finally中的动作不影响返回值)，如果finally中有return语句，则执行finally中的return语句。

```py
# test5.py 
def test5():
   	for i in range(5):
      	try:
         	print('do stuff %s'%i)
         	raise Exception(i)
      	except Exception:
         	print('exception %s'%i)
         	continue
 		finally:
         	print('do finally %s'%i)
test5()
```

从 test5.py 的输出结果得出如下结论：

- 在一个循环中，最终要跳出循环之前，会先转到finally执行，执行完毕之后才开始下一轮循环

## 1.2 python2 和 python3的继承顺序

[python2与python3的经典类与新式类的继承](https://www.jianshu.com/p/3b3d12063011)

 python2与python3都有经典类和新式类这两种写法，在python2中经典类和新式类的继承方式不一样，在python3中都一样。
 python2的经典类采用 **深度优先搜索** 继承方式，新式类采用 **广度优先搜索** 的继承方式
 python3中经典类和新式类都采用 **广度优先搜索** 的继承方式

![img](Python学习笔记.assets/9098035-60edf601a9ee402c.png)

![img](Python学习笔记.assets/9098035-24eae5396003d63f.png)



![img](Python学习笔记.assets/9098035-80ce0243ffae0e35.png)

## 1.3 python2 和 python3的round函数是一样的吗？

不一样。

 

python2 中的round(x) 把x四舍五入为远离0的最近倍数，如：

```py
round(0.5) # == 1
round(-0.5) # == -1
round(1.5) # == 2
```


python3 中的round(x) 把x 四舍五入为最近的偶数倍数，如：

```py
round(0.5) # == 0
round(-0.5) # == 0
round(1.5) # == 2
round(2.5) # == 2
```


python3中round() 对浮点数的取舍遵循的是四舍六入五平分，“五平分”就是根据取舍位数前的小数的奇偶性来判断，进行平分。奇数舍弃，偶数加一。

```py
round(2.135, 2) # == 2.13
round(2.145, 2) # == 2.15
```

## 1.4 python作用域和命名空间

==命名空间(Namespace)是从名称到对象的映射，大部分的命名空间都是通过 Python 字典来实现的。==


命名空间提供了在项目中避免名字冲突的一种方法。各个命名空间是独立的，没有任何关系的，所以一个命名空间中不能有重名，但不同的命名空间是可以重名而没有任何影响。

一般有3种命名空间，查找顺序是` local` -> `global` -> `built-in`

![python_namespace](Python学习笔记.assets/python_namespace.png)


命名空间的生命周期：

命名空间的生命周期取决于对象的作用域，如果对象执行完成，则该命名空间的生命周期就结束。

因此，我们无法从外部命名空间访问内部命名空间的对象。

 

作用域就是一个 Python 程序可以直接访问命名空间的文本区域。

> 这句话怎么理解呢？就是作用域就是你所写的代码文本区域。例如，一个函数的局部作用域就是，从def开始到函数结束这这段代码文本区域！因为，在这段文本区域中可以直接访问该函数的局部命名空间！！DO YOU KNOW?

python 中有4种作用域。

` local` -> `nonlocal（enclosing local）`-> `global` -> `built-in`

 ![python_作用域](Python学习笔记.assets/python_作用域.png)

## 1.5 python 自省和反射

[python自省与反射](https://blog.csdn.net/zjbyough/article/details/96037399)

==自省是获取对象的能力，反射是操纵对象的能力，python中使用`getattr()`和`setattr()`实现反射，而其他的则是自省。==

 通过JAVA来理解自省和反射。

JAVA反射 (`Reflection`) ， JAVA 反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法；这种动态获取的信息以及动态调用对象的方法的功能称为 java 语言的反射机制。

JAVA自省 (`Introspector`) ，是“看透 class ”的能力（内省、内观、反省）。

 

## 1.6 面向对象的几大特征？为什么继承破坏了封装？

[[面向对象的三大基本特征，五大基本原则 ](https://www.cnblogs.com/fzz9/p/8973315.html)

 **封装**：封装就是隐藏对象的属性和实现细节，仅对外公开接口，控制在程序中属性的读和修改的访问级别，将抽象得到的数据和行为（或功能）相结合，形成一个有机的整体，也就是将数据与操作数据的源代码进行有机的结合，形成“类”，其中数据和函数都是类的成员。

**继承**：继承是面向对象的基本特征之一，继承机制允许创建分等级层次的类。继承就是子类继承父类的特征和行为，使得子类对象（实例）具有父类的实例域和方法，或子类从父类继承方法，使得子类具有父类相同的行为。

**多态**：同一个行为具有多个不同表现形式或形态的能力。是指一个类实例（对象）的相同方法在不同情形有不同表现形式。多态机制使具有不同内部结构的对象可以共享相同的外部接口。这意味着，虽然针对不同对象的具体操作不同，但通过一个公共的类，它们（那些操作）可以通过相同的方式予以调用。

## 1.7 一句话解释什么样的语言能用装饰器

==函数可以作为参数传递==的语言，可以使用装饰器



## 1.8 Python的GIL

[python中的GIL详解](https://www.cnblogs.com/SuKiWX/p/8804974.html)

In CPython, the global interpreter lock, or GIL, is a mutex that prevents multiple native threads from executing Python bytecodes at once. This lock is necessary mainly because CPython’s memory management is not thread-safe. (However, since the GIL exists, other features have grown to depend on the guarantees that it enforces.)

全局解释器锁（Global Interpreter Lock），一个防止多线程并发执行机器码的一个Mutex。

每个线程在执行时候都需要先获取GIL，保证同一时刻只有一个线程可以执行代码，即同一时刻只有一个线程使用CPU，也就是说多线程并不是真正意义上的同时执行。



### 什么时候会释放GIL锁?

1. 遇到像 i/o操作这种 会有时间空闲情况 造成cpu闲置的情况会释放GIL
2. 会有一个专门ticks进行计数 一旦ticks数值达到100 这个时候释放GIL锁 线程之间开始竞争GIL锁(说明，ticks这个数值可以进行设置来延长或者缩减获得GIL锁的线程使用cpu的时间)

### 互斥锁和GIL锁的关系

- GIL锁：保证同一时刻只有一个线程能使用到CPU
- 互斥锁：多线程时，保证修改共享数据时有序地修改，且不产生数据修改混乱的情况。



## 1.9 常用的Python标准库

os, sys, re, urllib, requests, threading, multiprocessing, json, csv, time, datetime, logging等

## 1.10 Python内建数据类型

- Number。 不可变类型。int、float、bool、complex(复数)
- str。 不可变类型。
- tuple。 不可变类型。
- list。可变类型。
- set。可变类型。
- dict。可变类型。

## 1.11 with 语句块

```python
with open("./test.txt", 'r') as f:
	result = f.readlines()
	
要用with语句块，必须实现__enter__()和__exit__()魔法方法。
```

上述的例子中`open("./test.txt", 'r')`返回的是一个上下文对象即`f`，一旦获取了上下文对象，就会调用\_\_enter\_\_()方法，当with语句块结束时，会调用\_\_exit\_\_()方法。

## 1.12 Python中的断言

Python assert（断言）用于判断一个表达式，在表达式条件为 false 的时候触发异常。

语法格式如下：

```python
assert expression
# 等价于：
if not expression:
    raise AssertionError
    
# 携带参数版
assert expression [, arguments]
# 等价于：
if not expression:
    raise AssertionError(arguments)
```

## 1.13 Python2 和 Python3 的区别

[Python 2 和 Python 3 有哪些主要区别？](https://www.zhihu.com/question/19698598)

- print 

  ```python
  在Python2中是语句；在Python3中是函数
  ```

- 编码 

  ````python
  在Python2默认编码是ascii（所以需要在第一行`#coding=utf-8`）；在Python3默认编码是Unicode
  ````

- 字符串 

  ```python
  在Python2中字符串有两种类型：str和unicode，文本字符串和字节序列；在Python3中分别是byte和str。
  ```

- True/False 

  ```python
  在Python2中是两个全局变量，可以修改他俩的值；在Python3中是两个关键字，不允许修改。
  ```

- 迭代器 

  在Python2许多内置函数和方法的返回值都是列表；在Python3这些内置函数和方法的返回值都改成迭代器了。迭代器具有惰性加载，使得操作大数据更有效率。
  
  ```python
  # 在Python2中range返回值为列表，xrange返回值为迭代器
  # 在Python3中range返回值为迭代器，没有xrange函数了。
  ```
- nonlocal
  我们都知道在Python2中可以在函数里面可以用关键字 global 声明某个变量为全局变量，但是在嵌 套函数中，想要给一个变量声明为非局部变量是没法实现的，在Pyhon3，新增了关键字 nonlcoal，使得非局部变量成为可能。

- try...except 

   ```python
  except Exception, e 变成 except Exception as e
  ```

- / 和 //

  ```python
  # Python2中/的结果是整型
  # Python3中/的结果是浮点型
  ```

- 类

  ```python
  Python2中既有旧式类，也有新式类。
  在Python 3中，没有旧式类，只有新式类，也就是说不用再像这样 class Foobar(object): pass 显式地子类化object
  
  新式类的MRO(method resolution order 基类搜索顺序)算法采用C3算法广度优先搜索，而旧式类的MRO算法是采用深度优先搜索
  
  Python2 的旧式类是深度优先顺序继承，新式类是广度优先顺序继承
  Python3 的类都是广度优先顺序继承
  ```


- raw_input() 和 input()

  ```python
  Python2.x 中 input() 相等于 eval(raw_input(prompt)) ，用来获取控制台的输入。
  raw_input() 将所有输入作为字符串看待，返回字符串类型。而 input() 在对待纯数字输入时具有自己的特性，它返回所输入的数字的类型（ int, float ）。
  
  Python3中，input()即Python2的raw_input()
  ```

  

## 1.14 用lambda函数实现两数相乘

```python
func = lambda x,y: x*y
print(func(3, 5))
```

## 1.15 字典根据value的大小，从小到大排序

    ```python
    students = {"key1": 10, "key2": 9, "key3": 20, "key4": 15}
    print(sorted(students.items(), key=lambda x:x[1]))
    ```


​    

## 1.16 常见的网络传输协议

![OSI七层模型](Python学习笔记.assets/v2-edafd095d97dd329af845e8e8cecee65_720w.jpg)



## 1.17 正则 re.compile的作用

    re.compile()的定义
    
    > compile(pattern, flags=0)
    >
    > Complie a regular expression pattern, returning a pattern object.
    
    从compile()函数的定义中，可以看出返回的是一个匹配对象，它单独使用就没有任何意义，需要和findall(), search(), match(）搭配使用。 
    
    re.compile()的作用是，将正则表达式封装成一个pattern对象，方便复用。
    
    ```python


​    ```


​    

## 1.18 a = (1, );b = (1); c = ("1") 分别是什么类型

    tuple, int, str


​    

## 1.19 两个列表[1, 5, 7, 9] 和 [2, 2, 6, 8]和并为[1, 2, 2, 5, 6, 7, 8, 9]

    ```python
    a = [1, 5, 7, 9]
    b = [2, 2, 6, 8]
    a.extend(b)
    # 或者， a = a + b
    a.sort()
    ```


​    

## 1.20 用Python删除文件

    ```python
    import os
    os.remove(path)
    os.rmdir(path)
    os.unlink()
    ```


​    

## 1.21 Python垃圾回收机制

[Python垃圾回收机制详解](https://blog.csdn.net/xiongchengluo1129/article/details/80462651)


- 引用计数。Python垃圾回收主要以引用计数为主，分代回收为辅。引用计数法的原理是每个对象维护一个**ob_ref**，用来记录当前对象被引用的次数，也就是来追踪到底有多少引用指向了这个对象。
  
   当发生以下四种情况的时候，该对象的引用计数器**+1**
   - 对象被创建 a = 14
   - 对象被引用 b = a
   - 对象被作为参数，传递到函数中 func(a)
   - 对象作为一个元素，存储在容器中 students_age = [a, 10, 13, 15]
     
   
   当发生如下四种情况时，该对象的引用计数 **-1**
        
   - 该对象的别名被显式销毁时 del a
     
   - 该对象的引用别名被赋予新对象  a = 26
     
   - 一个对象离开它的作用域，例如 func函数执行完毕时，函数里面的局部变量的引用计数器就会减一（但是全局变量不会）
     
   - 该元素从容器中删除时，或容器被销毁时。
     
   
   `当指向该对象的内存的引用计数器为0的时候，该内存将会被Python虚拟机销毁`
   
   优点：高效、对象有确定的生命周期、易于实现
  
  缺点：维护引用计数消耗资源，维护引用计数的次数和引用赋值成正比，、无法解决==循环引用问题==
  
- 标记-清除。
  
>  标记清除（Mark—Sweep）』算法是一种基于追踪回收（tracing GC）技术实现的垃圾回收算法。它分为两个阶段：第一阶段是标记阶段，GC会把所有的『活动对象』打上标记，第二阶段是把那些没有标记的对象『非活动对象』进行回收。那么GC又是如何判断哪些是活动对象哪些是非活动对象的呢？
  >
  >  <br>
  >
  >  对象之间通过引用（指针）连在一起，构成一个有向图，对象构成这个有向图的节点，而引用关系构成这个有向图的边。从根对象（root object）出发，沿着有向边遍历对象，可达的（reachable）对象标记为活动对象，不可达的对象就是要被清除的非活动对象。根对象就是全局变量、调用栈、寄存器。
  >
  >  ![laji](Python学习笔记.assets/mark-sweep.svg)
  >
  >  解决了循环引用的问题。

- 分代回收
  
  > **先给出gc的逻辑：**（重点）
  >
  > ```
  > 分配内存
  > -> 发现超过阈值了
  > -> 触发垃圾回收
  > -> 将所有可收集对象链表放到一起
  > -> 遍历, 计算有效引用计数
  > -> 分成 有效引用计数=0 和 有效引用计数 > 0 两个集合
  > -> 大于0的, 放入到更老一代
  > -> =0的, 执行回收
  > -> 回收遍历容器内的各个元素, 减掉对应元素引用计数(破掉循环引用)
  > -> 执行-1的逻辑, 若发现对象引用计数=0, 触发内存回收
  > -> python底层内存管理机制回收内存
  > ```
  
  Python中, 引入了分代收集, 总共三个”代”. Python 中, 一个代就是一个链表, 所有属于同一”代”的内存块都链接在同一个链表中 , 用来表示“代”的结构体是**gc_generation**， 包括了当前代链表表头、对象数量上限、当前对象数量。新生成的对象会被加入第0代。
  
   分代回收是一种以空间换时间的操作方式，Python将内存根据对象的存活时间划分为不同的集合，每个集合称为一个代，Python将内存分为了3“代”，分别为年轻代（第0代）、中年代（第1代）、老年代（第2代），他们对应的是3个链表，它们的垃圾收集频率与对象的存活时间的增大而减小。新创建的对象都会分配在年轻代，年轻代链表的总数达到上限时，Python垃圾收集机制就会被触发，把那些可以被回收的对象回收掉，而那些不会回收的对象就会被移到中年代去，依此类推，老年代中的对象是存活时间最久的对象，甚至是存活于整个系统的生命周期内。同时，分代回收是建立在标记清除技术基础之上。分代回收同样作为Python的辅助垃圾收集技术处理那些容器对象.

## 1.22 写一段自定义异常代码

    ```python
    # 自定义异常类型
    class LoginException(Exception):
        pass
    
    def login():
        name = input("username:").strip()
        pwd = input("password:").strip()
    
        if name == "blex" and pwd == "123":
            print("登录成功!")
        else:
            raise LoginException("用户名或密码不正确....")
    
    login()
    ```


​    

## 1.23 正则表达式中，(.*) 和 (.\*?) 的区别

    (.*) 表示贪婪匹配
    
    (.\*?) 表示非贪婪匹配


​    

## 1.24 [[1,2], [3, 4], [5, 6]] 一行代码展开该列表，得出[1, 2, 3, 4, 5, 6]

    ```python
    print([j for i in [[1,2], [3, 4], [5, 6]] for j in i])
    ```


​    

## 1.25 Python多行注释

    ```python
    """
    这是多行注释，或者文档字符串
    """
    ```


​    

## 1.26 Python中交换两个数值

    ```python
    a, b = 10, 11
    a, b = b, a
    # 或
    temp = a
    a = b
    b = temp
    # 或
    a = a + b
    b = a - b
    a = a - b
    # 或
    a = a ^ b
    b = a ^ b # b = (a^b)^b = a
    a = a ^ b # a = (a^b)^a = b
    ```


​    

## 1.27 举例说明zip()函数的用法

    > zip()是Python的一个内建函数，它接受一系列可迭代的对象作为参数，将对象中对应的元素打包成一个个tuple（元组），然后返回由这些tuples组成的list（列表）。若传入参数的长度不等，则返回list的长度和参数中长度最短的对象相同。
    
    ```python
    a = [1,2,3] 
    b = [4,5,6] 
    c = [4,5,6,7,8] 
    zipped = zip(a,b) # [(1, 4), (2, 5), (3, 6)] 
    zip(a,c) # [(1, 4), (2, 5), (3, 6)] 
    zip(*zipped) # [(1, 2, 3), (4, 5, 6)] 
    ```


​    

## 1.28 a="张明 98分"，用re.sub, 将98替换成100

    > sub(pattern, repl, string, count=0, flags=0)
    >     Return the string obtained by replacing the leftmost
    >     non-overlapping occurrences of the pattern in string by the
    >     replacement repl.  repl can be either a string or a callable;
    >     if a string, backslash escapes in it are processed.  If it is
    >     a callable, it's passed the match object and must return
    >     a replacement string to be used.
    
    ```python
    import re
    
    a="张明 98分"
    print(re.sub("98", "100", a))
    ```


​    

## 1.29 a="hello"和b="你好"编码成bytes类型

    ```python
    a="hello" # str类型
    b="你好" # str类型
    print(a.encode("utf-8"))
    print(b.encode("utf-8"))
    ```


​    

## 1.30 [1, 2, 3] + [4, 5, 6]的结果是？

    ```python
    a = [1, 2, 3]
    b =[4, 5, 6]
    print(a+b) # [1, 2, 3, 4, 5, 6]
    ```


​    

## 1.31 提高Python运行效率的方法

    - 使用内置数据类型
    - 尽量使用内置函数，去掉属性访问
    - 使用生成器，如列表推导式
    - 尽量使用本地变量
    - ......

## 1.32 正则匹配，匹配日期2018-03-20

    ```python
    import re
    
    pattern = r"([1-9]\d{0,2}|[1-2]\d{3})-(0[1-9]|1[0-2])-(0[1-9]|[1-2][0-9]|3[0-1])"
    a = "2018-03-20"
    b = "9999-03-20"
    re_pattern = re.compile(pattern)
    print(re_pattern.search(a))
    print(re_pattern.search(b))
    ```


​    

## 1.33 简述多线程、多进程

    **进程是资源分配的最小单位，线程是CPU调度的最小单位**
    
    - 线程在进程下行进（单纯的车厢无法运行）
    
    - 一个进程可以包含多个线程（一辆火车可以有多个车厢）
    
    - 不同进程间数据很难共享（一辆火车上的乘客很难换到另外一辆火车，比如站点换乘）
    
    - 同一进程下不同线程间数据很易共享（A车厢换到B车厢很容易）
    
    - 进程要比线程消耗更多的计算机资源（采用多列火车相比多个车厢更耗资源）
    
    - 进程间不会相互影响，一个线程挂掉将导致整个进程挂掉（一列火车不会影响到另外一列火车，但是如果一列火车上中间的一节车厢着火了，将影响到所有车厢）
    
    - 进程可以拓展到多机，进程最多适合多核（不同火车可以开在多个轨道上，同一火车的车厢不能在行进的不同的轨道上）
    
    - 进程使用的内存地址可以上锁，即一个线程使用某些共享内存时，其他线程必须等它结束，才能使用这一块内存。（比如火车上的洗手间）－"互斥锁"
    
    - 进程使用的内存地址可以限定使用量（比如火车上的餐厅，最多只允许多少人进入，如果满了需要在门口等，等有人出来了才能进去）－“信号量”


​    

## 1.34 写一个单例模式

    pass


​    

## 1.35 列出常见的状态码和意义

    1. 信息。服务器收到请求，请继续执行请求 
    
    2. 成功。请求被成功接收并处理 
    3. 重定向。需要进一步操作来完成请求  
    4. 客户端错误。无法完成请求，或请求包含语法错误  
    5. 服务器错误。服务器在处理请求的过程中发成错误


​    

## 1.36 分别从前端、后端、数据库阐述web项目的性能优化

    > **前端优化：**
    >
    > - 减少http请求、例如制作精灵图
    > - 在首页通过嵌入式方式加入CSS，javascript放在页面下面，因为js加载比HTML和CSS加载慢，所有要优先加载html和css，以防止页面显示不全，性能查，也影响用户体验差
    > - 压缩文件
    >
    > **后端优化：**
    >
    > - 缓存存储读写次数高，变化少的数据，比如网站页面的信息、商品的信息等。应用程序读取数据时，一般是先从缓存中读取，如果读取不到或数据已失效，再访问磁盘数据库，并将数据再次写入缓存。
    > - 动静分离。将动态页面弄成html页面，通过ngnix来访问。
    > - 异步方式，如果有耗时操作，可以采用异步，比如celery
    > - 代码优化，避免循环和判断次数太多，如果多个if else判断，有限判断最有可能先发送的情况
    >
    > **数据库优化：**
    >
    > - 选择合适的数据库，如果有条件，数据可以存放于redis，读取速度块。
    > - 建立索引、外键等

## 1.37 Python中读取Excel文件的方法

    ```python
    import pandas as pd
    # 需要pip3 install xlrd
    
    df = pd.read_excel("111.xlsx")
    print(df)
    
    ```


​    

## 1.38 lambda匿名函数的好处

    >  编程中提到的lambda表达式，通常是在需要一个函数，但是又不想费神去命名一个函数的场合下使用。
    
    简洁


​    

## 1.39 简述同源策略

    [同源策略概念 简述](https://blog.csdn.net/syh_486_007/article/details/51043838)
    
    >  **浏览器有一个很重要的概念——同源策略(Same-Origin Policy)。所谓同源是指，域名，协议，端口相同。不同源的客户端脚本(javascript、ActionScript)在没明确授权的情况下，不能读写对方的资源。**
    >
    > 
    >
    > 简单的来说，浏览器允许包含在页面A的脚本访问第二个页面B的数据资源，这一切是建立在A和B页面是同源的基础上。
    >
    > 同源策略是由Netscape提出的一个著名的安全策略，现在所有支持JavaScript 的浏览器都会使用这个策略。
    > 实际上，这种策略只是一个规范，并不是强制要求，各大厂商的浏览器只是针对同源策略的一种实现。它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，则浏览器的正常功能可能都会受到影响。

## 1.40 简述any() 和 all() 方法

    可迭代对象中元素都为真，才为True
    
    可迭代对象中元素只要有为真的，就为True
    
    ```python
    a = [0, 1, 3, False, True, "4"]
    print(any(a)) # True
    print(all(a)) # False
    ```


​    

## 1.41 Python中copy() 和 deepcopy()

    - 浅拷贝。只复制一层。如果是可变对象，拷贝的两个对象会相互影响。
    - 深拷贝。每一层都复制。不管是可变还是不可变对象，拷贝的两个对象都是完全独立的。

## 1.42 列出几种魔法方法并简要介绍用途

    __init__: 对象初始化方法
    __new__: 创建对象时候执行的方法，单列模式会用到
    
    __init__是当实例对象创建完成后被调用的，然后设置对象属性的一些初始值。
    __new__是在实例创建之前被调用的，因为它的任务就是创建实例然后返回该实例，是个静态方法。
    即，__new__在__init__之前被调用，__new__的返回值（实例）将传递给__init__方法的第一个参数，然后__init__给这个实例设置一些参数。
    
    __str__: 当使用print输出对象的时候，只要自己定义了__str__(self)方法，那么就会打印从在这个方法中return的数据
    
    __del__: 删除对象执行的方法
    
    __call__: 当调用实例对象时执行的方法
    
    __enter__: 进入with语句块时，执行的方法
    __exit__: 退出with语句块时，执行的方法

## 1.45 生成0-100的随机数

    ```python
    import random
    print(random.randint(0, 100))
    ```


​    

## 1.46 请将[i for i in range(3)]改成生成器

    ```python
    (i for i in range(3))
    ```


​    

## 1.47 a = " heheh "，去掉尾部空格

    ```python
    a = " heheh "
    b = a.rstrip() # str.lstrip(), str.rstrip(), str.strip()
    print(b) # " heheh"
    ```


​    

## 1.48 举例sort和sorted对列表排序，list1 = [0, -1, 3, -10, 5, 9]

    sort是方法是原地排序，sorted的内置函数是返回一个新的对象
    
    ```python
    list1 = [0, -1, 3, -10, 5, 9]
    list1.sort()
    list2 = sorted(list1)
    assert list1 is list2 # AssertionError
    ```


​    

## 1.49 Python中传参数是传值还是传引用

    不可变对象作为参数，传递给函数时，是值传递
    
    可变对象作为参数，传递给函数时，是引用传递。


​    

## 1.50 求两个list的交集、差集和并集

    ```python
    a = [1,2,3]
    b = [2,3]
    # 交集
    print(set(a)&set(b))
    # 差集
    print(set(a)-set(b))
    # 并集
    print(set(a)|set(b))
    ```


​    

## 1.51 列表嵌套字典的排序，分别根据年龄和姓名排序

    ```python
    students = [{"name": "tom", "age": 20}, {"name": "jeery", "age": 29}, {"name": "zara", "age": 23}]
    print(sorted(students, key=lambda x:x["name"]))
    print(sorted(students, key=lambda x:x["age"]))
    ```

​    需要注意的是

```python
>>> a = [1, 2, 3]
>>> b = dict.fromkeys(a, [])
>>> print(b)# 是什么？
>>> {1: [], 2:[], 3:[]}
>>> b[0].append("x")
>>> print(b) # 是什么？
>>> {1: [1], 2:[1], 3:[1]}
```



## 1.50 列表嵌套元组，分别按字母和数字排序

    ```python
    students = [("tom", 20), ("jeery", 29), ("zara", 23)]
    print(sorted(students, key=lambda x:x[0]))
    print(sorted(students, key=lambda x:x[1]))
    ```


​    

## 1.51 列表嵌套列表，年龄数字相同怎么办

    按其他元素的大小
    
    ```python
    students = [["tom", 20], ["jeery", 29], ["zara", 20]]
    print(sorted(students, key=lambda x:x[0]))
    print(sorted(students, key=lambda x:x[1]))
    ```


​    

## 1.52 列表推导式、字典推导式、生成器

    ```python
    # 列表推导式
    [i for i in range(10)]
    # 字典推导式
    {i: 2*i for i in range(10)}
    # 生成器
    (i for i in range(10))
    ```


​    

## 1.53 举例说明SQL注入和解决办法

    利用SQL语句的漏洞
    
    > ```
    > sql="select * from stu where stu_name = $name and stu_email = $password";
    > # SQL注入之后
    > select * from t_admin where stu_name ='xxx' and stu_email ='xxx'' or '1；
    > # 甚至
    > delete rom t_admin where stu_name ='xxx' and stu_email='xxx' or 1 == 1; #本来是只删除某一个admin，现在删了全部的了，心态崩了
    > ```
    >
    > ##### 使用预备义语句和参数化查询，想想Java中的**PreparedStatement**

## 1.54 正则匹配 http://163.com结尾的邮箱

## 1.55 递归求和

    ```python
    a = range(100)
    
    def my_sum(iterable):
        if not iterable:
            return 0
        m_sum = iterable[0] + my_sum(iterable[1:])
        return m_sum
    ```


​    

## 1.56 Python字典和json字符串相互转换的方法

    ```python
    import json
    students = {"name": "tom", "age": 20, "gender": "male", "heigth": 50, "speed": 20}
    serialize_students = json.dumps(students)
    print(serialize_students)
    print(json.loads(serialize_students))
    ```


​    

## 1.57 统计字符串中某字符出现的次数

    ```python
    a = "adf;qekrpm;agdfafdm;akd;ga"
    print(a.count("a")) # 5
    ```


​    

## 58. 字符串转化大小写

    ```python
    a = "adf;qekrpm;agdfafdm;akd;ga"
    print(a.upper()) # "ADF;QEKRPM;AGDFAFDM;AKD;GA"
    ```


​    

## 1.59 用两种方法去空格

    ```python
    # 方法1， str.replace()方法
    a = " adalsj lwel a; "
    print(a.replace(" ", "")) # adalsjlwela;
    
    # 方法2，re.sub()方法
    import re
    print(re.sub(r"\s+", "", a)) # adalsjlwela;
    
    # 方法3，str.strip(), str.lstrip(), str.rstrip()
    ```


​    

## 1.60 正则匹配不是以4和7结尾的手机号

    ```python
    import re
    
    tels = ["13100001234", "18912344321", "10086", "18800007777"]
    pattern = re.compile(r"1\d{9}[^4,7]")
    for t in tels:
        print(t)
        ret = pattern.match(t)
        if ret:
            print(ret.group())
        else:
            print("手机号不符合要求")
    ```


​    

## 1.61 简述Python引用计数机制

    上面讲过了

## 1.62 int("1.4")，int(1.4)输出结果

    ```python
    >>> int("1.4")
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    ValueError: invalid literal for int() with base 10: '1.4'
    >>> int(1.4)
    1
    >>>
    ```


​    

## 1.63 列举3条以上PEP8编码规范

​    

## 1.64 正则表达式匹配第一个URL

    ```python
    pattern = r"(https?|ftp|file)://[-A-Za-z0-9+&@#/%?=~_|!:,.;]+[-A-Za-z0-9+&@#/%=~_|]
    ```


​    

## 1.65 正则匹配中文

    ```python
    import re
    pattern = re.compile(r"[\u4e00-\u9fa5]")
    ```


​    

## 1.66 r、r+、rb、rb+ 文件打开模式的区别

![img](Python学习笔记.assets/1554496-20190125094219530-2095834085.png)

### 1.67 查看对象有哪些属性

[Python动态获取对象的属性和方法 （转载）](https://www.cnblogs.com/zh1164/p/6031464.html)

```python
dir(obj)
obj.__dict__
```

## 1.68 Python静态方法、类方法和实例方法的区别

[Python中静态方法和类方法的区别？](https://www.zhihu.com/question/31844003)

> Python的类就是个语法糖。一个函数写在类里面和写在类外面没有区别，唯一的区别就是参数，所谓实例方法就是第一个参数是self，所谓类方法就是第一个参数是class，而静态方法不需要额外的参数，所以必须区分。

> 一、先是在语法上面的区别:
>
> 1、静态方法不需要传入self参数，类成员方法需要传入代表本类的cls参数；
>
> 2、静态方法是无妨访问实例变量和类变量的，类成员方法无法访问实例变量但是可以访问类变量
>
> 二、使用的区别：
>
> 由于静态方法无法访问类属性，实例属性，相当于一个相对独立的方法，跟类其实并没有什么关系。这样说来，静态方法就是在类的作用域里的函数而已。

## 1.69 requests和urllib的区别

requests是对urllib的进一步封装，使用起来更加便捷。

urllib需要先构造请求，在发起请求

requests可以直接发起请求，更方便。

```python
import urllib
req = urllib.request.Request(url=url, headers=headers)
webpage = urllib.request.urlopen(req)
print(webpage.read().decode("utf-8"))

import requests
response = requests.get(url, headers=headers, params=params)
response1 = requests.post(url, headers=headers, data=data, json=json)
print(response)
print(response1)
```



## 1.70 多个装饰器的执行顺序？

> 多个装饰器的装饰顺序是从下到上。
>
> 多个装饰装饰后的函数的执行顺序是从上到下的。

```python
# test_decorator_order.py

def decorator_1(func):
    print("execute decorator_1")
    def wrapper(*args, **kwargs):
        print("execute decorator_1.wrapper")
        res = func(*args, **kwargs)
        
    return wrapper

def decorator_2(func):
    print("execute decorator_2")
    def wrapper(*args, **kwargs):
        print("execute decorator_2.wrapper")
        res = func(*args, **kwargs)
        
    return wrapper


@decorator_1
@decorator_2
def test():
    print("execute test function")


test()
```

输出结果如下：

```powershell
(myscrapy) PS C:\Users\sjl\Desktop> python .\Untitled-1.py
execute decorator_2
execute decorator_1
execute decorator_1.wrapper
execute decorator_2.wrapper
execute test function
(myscrapy) PS C:\Users\sjl\Desktop>
```



## 1.71 迭代器和生成器

[Python官方文档 迭代器](https://docs.python.org/zh-cn/3/tutorial/classes.html#iterators)

![relations](Python学习笔记.assets/relationships.png)

> iterable: 定义了`__iter__()`方法，返回一个定义了`__next__()`方法的对象。通过`collections`包中的`Iterable`可以判断是否是可迭代对象。
>
> iterator: 同时定义了`__iter__()`方法和`__next__()`方法的对象。
>
> for i in iterable: 语句实际做了两件事：
>
> - 调用`iterable.__iter__()`方法获的一个定义了`__next__()`方法的对象x。
> - 调用`x.__next__()`方法获取每次迭代的值。

看过迭代器协议的幕后机制，给你的类添加迭代器行为就很容易了。 定义一个 [`__iter__()`](https://docs.python.org/zh-cn/3/reference/datamodel.html#object.__iter__) 方法来返回一个带有 [`__next__()`](https://docs.python.org/zh-cn/3/library/stdtypes.html#iterator.__next__) 方法的对象。 如果类已定义了 `__next__()`，则 [`__iter__()`](https://docs.python.org/zh-cn/3/reference/datamodel.html#object.__iter__) 可以简单地返回 `self`



```python
from collections import Iterable

class A:
    def __iter__(self):
        return [1, 2, 3]
    
isinstance(A(), Iterable) # True
```

**结论：**

**1）可迭代对象包含迭代器。
2）如果一个对象拥有__iter__方法，其是可迭代对象；如果一个对象拥有next方法，其是迭代器。
3）定义可迭代对象，必须实现__iter__方法；定义迭代器，必须实现__iter__和next方法。** 

## 1.72 Python中的私有变量

> 链接：https://www.nowcoder.com/questionTerminal/1c4b55a530ee499c986efe82c173b645?toCommentId=9287324
> 来源：牛客网
>
> 
>
> python中主要存在四种命名方式：   
>
>   1、object #公用方法  
>
>   2、_object #半保护  
>
> ​           \#被看作是“protect”，意思是只有类对象和子类对象自己能访问到这些变量， 
>
> ​           在模块或类外不可以使用，不能用’from module import *’导入。  
>
> ​          \#__object 是为了避免与子类的方法名称冲突， 对于该标识符描述的方法，父 
>
> ​           类的方法不能轻易地被子类的方法覆盖，他们的名字实际上是 
>
> ​           _classname__methodname。  
>
>   3、_ _ object #全私有，全保护  
>
> ​              \#私有成员“private”，意思是只有类对象自己能访问，连子类对象也不能访 
>
> ​               问到这个数据，不能用’from module import *’导入。  
>
>   4、_ _ object_ _   #**内建方法，用户不要这样定义**

其实，在[Python官方文档](https://docs.python.org/zh-cn/3/tutorial/)中都有的，多看看，多实践。

[Python类-私有变量](https://docs.python.org/zh-cn/3/tutorial/classes.html#private-variables)

有一种说法是：

> _variable 相当于java中的protected
>
> __variable相当于java中的private
>
> 引用官方文档中话：
>
> 带有一个下划线的名称 (例如 `_spam`) 应该被当作是 API 的非公有部分 (无论它是函数、方法或是数据成员)。 这应当被视为一个实现细节，可能不经通知即加以改变。
>
> 由于存在对于类私有成员的有效使用场景（例如避免名称与子类所定义的名称相冲突），因此存在对此种机制的有限支持，称为` 名称改写`。 任何形式为 `__spam` 的标识符（至少带有两个前缀下划线，至多一个后缀下划线）的文本将被替换为 `_classname__spam`，其中 `classname` 为去除了前缀下划线的当前类名称。 这种改写不考虑标识符的句法位置，只要它出现在类定义内部就会进行。

实践一下：

```python
class A(object):
    __name = "a"
    def __lalala(self):
        pass

    def __lalala_(self):
        pass

    def __lalala__(self):
        pass


class B(A):
    __name = "b"
    def __lalala(self):
        pass

    def __lalala_(self):
        pass

    def __lalala__(self):
        pass

print(A._A__name)
print(B._B__name)
```

执行结果：

```shell
(learnpython) PS D:\work\python-project> & C:/Users/sjl/Anaconda3/envs/learnpython/python.exe d:/work/python-project/test-python/test-private-variable.py
Traceback (most recent call last):
  File "d:/work/python-project/test-python/test-private-variable.py", line 25, in <module>
    print(A.__name)
AttributeError: type object 'A' has no attribute '__name'
(learnpython) PS D:\work\python-project> & C:/Users/sjl/Anaconda3/envs/learnpython/python.exe d:/work/python-project/test-python/test-private-variable.py
['_A__name', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__']
Traceback (most recent call last):
  File "d:/work/python-project/test-python/test-private-variable.py", line 26, in <module>
    print(A.__name)
AttributeError: type object 'A' has no attribute '__name'
```

发现类A并没有`__name`这个属性，通过`dir()`函数，发现对应的属性名应该是`_A__name`。

将上面代码修改后，执行：

```python
class A(object):
    __name = "a"
    def __lalala(self):
        pass

    def __lalala_(self):
        pass

    def __lalala__(self):
        pass


class B(A):
    __name = "b"
    def __lalala(self):
        pass

    def __lalala_(self):
        pass

    def __lalala__(self):
        pass

print(dir(A))
print(A._A__name)
print(dir(B))
print(B._B__name)
```

输出结果：

```shell
(learnpython) PS D:\work\python-project> & C:/Users/sjl/Anaconda3/envs/learnpython/python.exe d:/work/python-project/test-python/test-private-variable.py
['_A__lalala', '_A__lalala_', '_A__name', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__lalala__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__']
a
['_A__lalala', '_A__lalala_', '_A__name', '_B__lalala', '_B__lalala_', '_B__name', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__lalala__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__']
b
(learnpython) PS D:\work\python-project>
```

类B继承自类A，所以B中有`_A__name`和`_B__name`属性。

**你学会飞了吗**



## 1.73 windows cmd中pip3无法使用，怎么办

双击Python安装包  > repair > restart 电脑 > OK。

## 1.74 Celery

[Celery官方网站](https://docs.celeryproject.org/en/stable/)

[Celery 中文手册](https://www.celerycn.io/)

[任务调度利器：Celery](https://www.liaoxuefeng.com/article/903701468278784)

[Python 并行分布式框架 Celery 详解](https://blog.csdn.net/cuomer/article/details/81214438)

[celery 简要概述](https://blog.csdn.net/u010339879/article/details/97691231)

[python celery 任务队列](https://www.pyfdtic.com/2018/03/16/python-celery-%E4%BB%BB%E5%8A%A1%E9%98%9F%E5%88%97/)

Celery能做什么？



### 1. Celery是什么?

> Celery 是一款非常简单、灵活、可靠的分布式系统，可用于处理大量消息，并且提供了一整套操作此系统的一系列工具。
>
> Celery 是一款**消息队列工具**，可用于处理实时数据以及任务调度。

![Celery_framework](Python学习笔记.assets/584bbf78e1783.png)

#### **1.1 特点：**

1. **简单 Simple**， 不需要配置文件，简单易用。
2. **高可用 Highly Available**，多个broker，多个worker，主/主，主/从方式保证高可用。
3. **快速 Fast**，一个Celery进程能完成 100w个任务/minute
4. **灵活 Flexible**，Celery 的每个部分几乎都可以自定义扩展和单独使用，例如自定义连接池、序列化方式、压缩方式、日志记录方式、任务调度、生产者、消费者、中间人（Broker）等。

![image-20210324230410855](Python学习笔记.assets/image-20210324230410855.png)

#### **1.2 功能：**

![image-20210324230512168](Python学习笔记.assets/image-20210324230512168.png)



### 2. 什么是任务队列?

> **任务队列**一般用于线程或计算机之间分配工作的一种机制。
>
> 任务队列的输入是一个称为任务的工作单元，有专门的职程（**Worker**）进行不断的监视任务队列，进行执行新的任务工作。
>
> Celery 通过消息机制进行通信，通常使用中间人（Broker）作为客户端和职程（Worker）调节。启动一个任务，客户端向消息队列发送一条消息，然后中间人（Broker）将消息传递给一个职程（Worker），最后由职程（Worker）进行执行中间人（Broker）分配的任务。
>
> Celery 需要消息中间件来进行发送和接收消息。 **RabbitMQ** 和 **Redis** 中间人的功能比较齐全，但也支持其它的实验性的解决方案，其中包括 SQLite 进行本地开发。
>
> **Celery 是一个资金最少的项目，因此我们不支持 Microsoft Windows。请不要提出与该平台相关的任何问题。**



### 3. 安装Celery 

**使用Celery的前提条件?**:

> Celery 4.0 运行：
>
> - Python ❨2.7,3.4,3.5❩
> - PyPy ❨5.4,5.5❩
>
> 这是支持 Python2.7 的最后一个版本，从下一个版本Celery5.x开始，需要Python3.5或更高的版本。
>
> 如果您的 Python 运行环境比较老，则需要使用旧版本的Celery：
>
> - Python 2.6：Celery 3.1 或更早版本。
> - Python 2.5：Celery 3.0 或更早版本。
> - Python 2.4：Celery 2.2 或更早版本。



#### 3.1 通过`pip`安装Celery

```shell
pip install -U celery
```

#### 3.2 通过`pip`安装Celery和指定的绑定包。

```shell
pip install "celery[librabbitmq]"

pip install "celery[librabbitmq,redis,auth,msgpack]"
```

#### 3.3 源码安装

```shell
https://PyPI.org/project/celery/
tar zxvf celery-0.0.0.tar.gz
cd celery-0.0.0
python setup.py build
python setup.py install
```

#### 3.4 安装开发版Celery

```shell
$ pip install https://github.com/celery/celery/zipball/master#egg=celery
$ pip install https://github.com/celery/billiard/zipball/master#egg=billiard
$ pip install https://github.com/celery/py-amqp/zipball/master#egg=amqp
$ pip install https://github.com/celery/kombu/zipball/master#egg=kombu
$ pip install https://github.com/celery/vine/zipball/master#egg=vine
```



### 4. Brokers

Celery所支持的不同消息中间件的对比情况如下表：

| **Name**     | **Status**   | **Monitoring** | **Remote Control** |
| ------------ | ------------ | -------------- | ------------------ |
| *RabbitMQ*   | Stable       | Yes            | Yes                |
| *Redis*      | Stable       | Yes            | Yes                |
| *Amazon SQS* | Stable       | No             | No                 |
| *Zookeeper*  | Experimental | No             | No                 |

#### 4.1 使用RabbitMQ

**[安装 RabbitMQ](https://www.rabbitmq.com/)**

**配置**

要使用 Celery，需要创一个RabbitMQ账户

```shell
$ sudo rabbitmqctl add_user myuser mypassword
$ sudo rabbitmqctl add_vhost myvhost
$ sudo rabbitmqctl set_user_tags myuser mytag
$ sudo rabbitmqctl set_permissions -p myvhost myuser ".*" ".*" ".*"
```



**使用**

```shell
# 前台运行reabbitmq
$ sudo rabbitmq-server
# 后台运行rabbitmq
$ sudo rabbitmq-server -detached
# 千万不用用kill来停止rabbitmq，而要使用rabbitmqctl
$ sudo rabbitmqctl stop
```

#### 4.2 使用Redis

安装

```shell
$ pip install -U "celery[redis]"
```

配置

```shell
# 配置redis数据库的位置，URL格式如下
redis://:password@hostname:port/db_number
# 或使用套接字
redis+socket:///path/to/redis.sock
# 使用virtual_host参数指明使用的几号数据库，redis有16个数据库，0-15
redis+socket:///path/to/redis.sock?virtual_host=db_number

app.conf.broker_url = 'redis://localhost:6379/0'

# 或使用redis sentinel
app.conf.broker_url = 'sentinel://localhost:26379;sentinel://localhost:26380;sentinel://localhost:26381'
app.conf.broker_transport_options = { 'master_name': "cluster1" }

# 设置可见性超时，见性超时为将消息重新下发给另外一个程序之前等待确认的任务秒数。
app.conf.broker_transport_options = {'visibility_timeout': 3600}  # 1 hour.
```

使用

#### 4.3 使用Amazon SQS

安装

```shell
$ pip install celery[sqs]
```

配置

```shell
broker_url = 'sqs://ABCDEFGHIJKLMNOPQRST:ZYXK7NiynGlTogH8Nj+P9nlE73sq3@'
# URL格式
sqs://aws_access_key_id:aws_secret_access_key@

# 设置区域
broker_transport_options = {'region': 'eu-west-1'}

# 设置可见性超时
broker_transport_options = {'visibility_timeout': 3600}  # 1 hour.

# 设置轮询间隔时间
broker_transport_options = {'polling_interval': 0.3}
# 设置队列前缀
broker_transport_options = {'queue_name_prefix': 'celery-'}
```

使用

### 5. 初步使用Celery

流程如下：

选择一种broker（消息中间件）-> 安装celery -> 编写应用程序 -> 运行celery worker -> 调用task -> 保存结果 -> 配置

#### 5.1 选择一种broker

Celery最好是选择Rabbitmq，其次是Redis。因为Celery对这二者的支持最好。

```shell
# ubuntu或debian中安装rabbitmq
$ sudo apt-get install rabbitmq-server

# docker中运行rabbitmq
$ docker run -d -p 5672:5672 rabbitmq

# docker中运行redis
$ docker run -d -p 6379:6379 redis
```

#### 5.2 安装celery

```shell
$ pip install celery
```

#### 5.3 创建celery应用程序

要使用Celery所要做的第一件事就是创建一个Celery应用程序，它是做任何事的入口。**应用程序是线程安全的，可以配置多个不同配置、组件、任务的 Celery 程序在同一个进程中运行。**

```python
# tasks.py

from celery import Celery

# 第一个参数'tasks'是模块名，
# 第一个关键字参数broker指明了，消息中间件的URL
# 第二个关键字参数backend指明了，任务结果存储的位置
app = Celery('tasks', broker='pyamqp://guest@localhost//')

# app = Celery('tasks', broker='pyamqp://test:test123456@localhost/myrabbitmq', backend="rpc://")

@app.task
def add(x, y):
    return x + y
```

使用装饰器的方式定义了一个task，`add`，返回两个数字的和。

#### 5.4 运行 Celery worker 服务器

```shell
celery -A tasks worker --loglevel=INFO
# rabbitmq的配置如下：
$ sudo rabbitmqctl add_user test test123456
$ sudo rabbitmqctl add_vhost myrabbitmq
$ sudo rabbitmqctl set_user_tags test test-rabbitmq
$ sudo rabbitmqctl set_permissions -p myvhost myuser ".*" ".*" ".*"
```

在生产环境中，要将**Celery worker**作为守护进程在后台运行，可以使用类似`supervisord`这样的工具。

查看其它`celery`命令行参数：

```shell
celery worker --help

celery help
```

#### 5.5 调用task

使用`delay()`方法来调用我们的task，`delay()`方法是`apply_async()`方法的快捷方式。

```shell
>>> from tasks import add
>>> add.delay(4, 4)
```

此时，这个task就会被之前所启动的Celery worker服务器所处理。但是，由于上面的配置中，没有配置worker执行的结果存储在哪里，所以我们无法获得task的执行状态和结果。

**修改tasks.py内容**

```python
# tasks.py

from celery import Celery

# 第一个参数'tasks'是模块名，
# 第一个关键字参数broker指明了，消息中间件的URL
# 第二个关键字参数backend指明了，任务结果存储的位置
app = Celery('tasks', broker='pyamqp://test:test123456@localhost/myrabbitmq', backend="rpc://")

@app.task
def add(x, y):
    return x + y
```

```shell
>>> from tasks import add
>>> add.delay(4, 4)
>>> add.ready()
>>> add.get()
>>> add.forget()
>>> add.traceback #查看异常的详细情况
```

> **如果后端使用资源进行存储结果，必须要针对调用任务后返回每一个 AsyncResult 实例调用 get() 或 forget() ，进行资源释放。**

### 5.6 配置celery

在大型项目中，通常将所有配置集中放到某一个文件中，如**celeryconfig.py**

```python
# celeryconfig.py

broker_url = "pyamqp://"
result_backend = "rpc://"

task_serializer = "json"
result_serializer = "json"
accept_content = ["json"]
timezone = "Asia/Shanghai"
enable_utc = True        
```

通过`x.config_from_object`加载配置

```python
app = Celery('tasks', broker='pyamqp://test:test123456@localhost/myrabbitmq', backend="rpc://")
app.config_from_object('celeryconfig')
```



通过`x.conf.update()`更新配置

```python
app = Celery('tasks', broker='pyamqp://test:test123456@localhost/myrabbitmq', backend="rpc://")
app.conf.update(
    task_serializer='json',
    accept_content=['json'],  # Ignore other content
    result_serializer='json',
    timezone='Europe/Oslo',
    enable_utc=True,
)
```

**记录日志**

```python
from celery.utils.log import get_task_logger
logger = get_task_logger(__name__)

# TODO
# 然后像使用logging模块中的logger实例一样用就可以了。
```



**任务绑定**

通过任务绑定(bind=True)，在我们编写的任务函数中可以访问到任务的属性和方法。示例如下：

```python
@app.task(bind=True)
def dump_context(self, x, y):
    print('Executing task id {0.id}, args: {0.args!r} kwargs: {0.kwargs!r}'.format(
            self.request))
```



### 使用Celery

**项目结构**

```shell
learn_celery/
├── celeryconfig.py
└── tasks.py
	
```



```python
# learn_celery/celery.py

from celery import Celery
 
 
app = Celery('mycelery',
        broker='amqp://',
        backend='rpc://',
        include=['learn_celery.tasks'])


app.conf.update(
    result_expires=3600,
)
 

if __name__ == '__main__':
    pp.start()                                                        
```



```python
# learn_celery/tasks.py

from .celery import app

@app.task
def add(x, y):
    return x + y

@app.task
def mul(x, y):
    return x * y

@app.task
def xsum(numbers):
    return sum(numbers)
```

**运行celery worker**

```shell
# 进入learn_celery目录的上一层，运行如下命令
# -A 指定celery应用程序所在模块名
(celery-env) [root@prometheus myProjects]# celery -A learn_celery worker -l INFO
/root/myProjects/virtualenvs/celery-env/lib/python3.7/site-packages/celery/platforms.py:797: RuntimeWarning: You're running the worker with superuser privileges: this is
absolutely not recommended!

Please specify a different user using the --uid option.

User information: uid=0 euid=0 gid=0 egid=0

  uid=uid, euid=euid, gid=gid, egid=egid,
 
 -------------- celery@prometheus.sjl.com v5.0.5 (singularity)
--- ***** ----- 
-- ******* ---- Linux-3.10.0-1127.el7.x86_64-x86_64-with-centos-7.8.2003-Core 2021-03-31 14:48:54
- *** --- * --- 
- ** ---------- [config]
- ** ---------- .> app:         mycelery:0x7f7692c95450
- ** ---------- .> transport:   amqp://guest:**@localhost:5672//
- ** ---------- .> results:     rpc://
- *** --- * --- .> concurrency: 4 (prefork)
-- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
--- ***** ----- 
 -------------- [queues]
                .> celery           exchange=celery(direct) key=celery
                

[tasks]
  . learn_celery.tasks.add
  . learn_celery.tasks.mul
  . learn_celery.tasks.xsum

[2021-03-31 14:48:55,134: INFO/MainProcess] Connected to amqp://guest:**@127.0.0.1:5672//
[2021-03-31 14:48:55,190: INFO/MainProcess] mingle: searching for neighbors
[2021-03-31 14:48:56,283: INFO/MainProcess] mingle: all alone
[2021-03-31 14:48:56,310: INFO/MainProcess] celery@prometheus.sjl.com ready.


$ celery worker --help
```

**后台运行 celery worker**

```shell
# 启动多个worker
(celery-env) [root@prometheus myProjects]# celery multi start w1 -A learn_celery -l info
celery multi v5.0.5 (singularity)
> Starting nodes...
	> w1@prometheus.sjl.com: OK

# 重启worker
(celery-env) [root@prometheus myProjects]# celery  multi restart w1 -A celery -l info
celery multi v5.0.5 (singularity)
> Stopping nodes...
	> w1@prometheus.sjl.com: TERM -> 102925
> Waiting for 1 node -> 102925.....
	> w1@prometheus.sjl.com: OK
> Restarting node w1@prometheus.sjl.com: OK
> Waiting for 1 node -> None...

# 停止worker
(celery-env) [root@prometheus myProjects]# celery multi stop w1 -A celery -l info
celery multi v5.0.5 (singularity)
> Stopping nodes...
	> w1@prometheus.sjl.com: TERM -> 102982

```

`stop` 命令是异步的，所以不会等待职程（Worker）关闭。可以通过 `stopwait` 命令进行停止运行，可以保证在退出之前完成当前正在执行的任务：

```shell
(celery-env) [root@prometheus myProjects]# celery multi stopwait w1 -A celery -l info
celery multi v5.0.5 (singularity)
> w1@prometheus.sjl.com: DOWN

```

> `celery multi` 不存储有关职程（Worker）的信息，所以在重新启动时需要使用相同的命令参数，停止运行时只能通过`pidfile`和`logfile`参数来进行停止运行

默认情况下会在当前目录中创建pid文件和日志文件，为防止多个职程（Worker）干扰，建议将这些文件存放在专门的目录中：



```
$ mkdir -p /var/run/celery
$ mkdir -p /var/log/celery
$ celery multi start w1 -A learn_celery -l info --pidfile=/var/run/celery/%n.pid \
                                        --logfile=/var/log/celery/%n%I.log
```

也可以使用 `multi` 命令启动多个职程（Worker），有一个强大的语法为不同职程（Worker）设置不同的参数：



```
$ celery multi start 10 -A learn_celery -l info -Q:1-3 images,video -Q:4,5 data \
    -Q default -L:4,5 debug
```

更多实例，可也参阅 `multi` API 模块。

**程序调用**

```python
>>> add.delay(2, 2)
>>> # 上面是下面的快捷方式
>>> add.apply_async((2, 2))
>>> # apply_async() 可以指定调用时执行的参数，例如运行的时间，使用的任务队列等：
>>> add.apply_async((2, 2), queue='lopri', countdown=10)

# 如果配置了结果后端，则能够获取任务结果
>>> res = add.delay(2, 2)
>>> res.get(timeout=1)

# 获取任务id
>>> res.id
d6b3aea2-fb9b-4ebc-8da4-848818db9114

# 不希望res.get()抛出异常，使用propagate进行禁用
>>> res.get(propagate=False)
TypeError('add() takes exactly 2 arguments (1 given)',)

# 检查任务执行状态
>>> res.failed()
True
>>> res.successful()
False
>>> res.state
'FAILURE'

# 任务在执行中会存在多种状态，如
PENDING -> STARTED -> SUCCESS
PENDING -> STARTED -> RETRY -> STARTED -> RETRY -> STARTED -> SUCCESS

# 使用celery签名
>>> add.signature((2, 2), countdown=10)
tasks.add(2, 2)
>>> add.s(2, 2)
tasks.add(2, 2)
>>> s1 = add.s(2, 2)
>>> res = s1.delay()
>>> res.get()
4
# incomplete partial: add(?, 2)
>>> s2 = add.s(2)
# resolves the partial: add(8, 2)
>>> res = s2.delay(8)
>>> res.get()
10
>>> s3 = add.s(2, 2, debug=True)
>>> s3.delay(debug=False)   # debug is now False.

#一个 group 并行调用任务列表，返回一个特殊的结果实例，可以将结果作为一个列表进行查看，并且通过索引进去获取返回值。
>>> from celery import group
>>> from learn_celery.tasks import add
>>> group(add.s(i, i) for i in xrange(10))().get()
[0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
# 偏组，patial group
>>> g = group(add.s(i) for i in xrange(10))
>>> g(10).get()
[10, 11, 12, 13, 14, 15, 16, 17, 18, 19]

# 可以将任务链接在一起，在一个任务返回后进行调用另外一个任务
>>> from celery import chain
>>> from learn_celery.tasks import add, mul
# (4 + 4) * 8
>>> chain(add.s(4, 4) | mul.s(8))().get()
64
#偏链，partial chains
>>> # (? + 4) * 8
>>> g = chain(add.s(4) | mul.s(8))
>>> g(4).get()
64

>>> (add.s(4, 4) | mul.s(8))().get()
64

# 和弦：Chords, 和弦是一个带有回调的组
>>> from celery import chord
>>> from learn_celery.tasks import add, xsum
>>> chord((add.s(i, i) for i in xrange(10)), xsum.s())().get()
90 
# 效果是前一个组的结果，再作为输入给另一个函数
# 或者
>>> (group(add.s(i, i) for i in xrange(10)) | xsum.s())().get()
90
```

**路由**

```python
# Celery 支持 AMQP 中提供的所有路由，可以将消息发送到指定的任务队列路由
app.conf.update(
    task_routes = {
        'proj.tasks.add': {'queue': 'hipri'},
    },
)
# 使用queue参数指定任务队列
>>> from proj.tasks import add
>>> add.apply_async((2, 2), queue='hipri')

# 在运行celery worker时，指定该worker所消费的任务队列
$ celery -A proj worker -Q hipri
# 该worker消费多个任务队列
$ celery -A proj worker -Q hipri,celery
```



**远程控制**

```shell
# 使用 RabbitMQ（AMQP）、Redis 或 Qpid 作为中间人（Broker），可以在运行时控制和检查Worker
# 这是通过广播消息实现的，集群中所有职程（Worker）都会所有收到远程控制发出的指令。
$ celery -A proj inspect active
$ celery -A proj inspect active --destination=celery@example.com
(celery-env) [root@prometheus myProjects]# celery -A learn-celery  inspect active
->  w1@prometheus.sjl.com: OK
    - empty -

1 node online.

# 可以强制Worker启用事件消息（用于监控任务以及Worker）：
$ celery -A proj control --help
$ celery -A proj control enable_events

# 远程控制并且显示集群中（Worker的列表
(celery-env) [root@prometheus myProjects]# celery -A learn-celery  status
->  w1@prometheus.sjl.com: OK

1 node online.
```



## 1.75 Python虚拟环境管理

Python中常见的几个虚拟环境包为：`virtualenv`，`virtualenvwrapper`，`venv`，`conda`，`pipenv`。

**virtualenv**

```shell
# 1. 安装virtualenv
pip install virtualenv

# 2. 创建一个虚拟环境
virtualenv myvenv
# 指定python解释器版本
virtualenv -p /usr/bin/python2.7 myvenv
# 如果不想使用系统的包,加上–no-site-packeages参数
virtualenv  --no-site-packages 创建路径名
# 3. 激活虚拟环境
# windows
myvenv\Scripts\activate.bat
# linux
source myvenv/bin/activate
# 4. 退出虚拟环境
# windows
myvenv\Scripts\deactivate.bat
# linux
deactivate
# 5. 删除虚拟环境
rm -rf myvenv
# 6. 其他命令
virtualenv --help

```

**virtualenvwrapper**

Virtaulenvwrapper是virtualenv的扩展包，用于更方便管理虚拟环境

```shell
# 1. 安装
# on Windows
pip install virtualenvwrapper-win
# on macOS / Linux
pip install --user virtualenvwrapper
# then make Bash load virtualenvwrapper automatically
echo "source virtualenvwrapper.sh" >> ~/.bashrc
source ~/.bashrc

# 2. 创建虚拟环境
# on macOS/Linux:
mkvirtualenv --python=python3.6 myvenv
# on Windows
mkvirtualenv --python=python3 myvenv

# 3. 激活虚拟环境
workon #列出虚拟环境列表
workon myvenv #切换环境
# 4. 退出虚拟环境
deactivate
# 5. 删除虚拟环境
rmvirtualenv myvenv

# 6.其他指令
pip freeze #查看当前安装库版本
#创建 requirements.txt 文件，其中包含了当前环境中所有包及 各自的版本的简单列表
#保持部署相同，一键安装所有包
pip install -r requirements.txt
pip freeze > requirements.txt 
lsvirtualenv    #列举所有的环境
cdvirtualenv    #导航到当前激活的虚拟环境的目录中，相当于pushd 目录
cdsitepackages   # 和上面的类似，直接进入到 site-packages 目录
lssitepackages     #显示 site-packages 目录中的内容
```

**venv**

venv是Python3.5之后自带的虚拟环境管理包。

```shell
# 1.创建一个虚拟环境
python3 -m venv /path/to/new/virtual/environment

# 2.激活虚拟环境
# windows
C:\> <venv>\Scripts\activate.bat
PS C:\> <venv>\Scripts\Activate.ps1
# Unix或macOS
 source <venv>/bin/activate
# 3.退出虚拟环境
# windows
C:\> <venv>\Scripts\deactivate.bat
PS C:\> <venv>\Scripts\deactivate.ps1
# Unix或macOS
deactivate
# 4. 删除虚拟环境
rm -rf /path/to/new/virtual/environment
# 5. 其他命令
python3 -m venv --help
```

**conda**

conda是一个包、依赖和环境的管理器，可适用于*Python, R, Ruby, Lua, Scala, Java, JavaScript, C/ C++, FORTRAN, 等语言*

```shell
# 1. 安装conda
# windows中安装，使用miniconda或者anaconda安装包
# linux中安装，使用miniconda或者anaconda安装包
bash Miniconda3-latest-Linux-x86_64.sh
bash Anaconda-latest-Linux-x86_64.sh
# macOS
bash Miniconda3-latest-MacOSX-x86_64.sh

# 2. 创建虚拟环境
# Python 2.7  
$ conda create -n venv python=2.7  
# Python 3.4  
$ conda create -n venv python=3.4  
# Python 3.5  
$ conda create -n venv python=3.5

# 3. 激活虚拟环境
# windows
activate vnev
# linux
source activate vnev

# 4. 退出虚拟环境
# windows
deactivate
# linux
source deactivate

# 5. 删除虚拟环境
conda remove --name venv --all

# 6. 其他命令
# 列出系统中所存在的虚拟环境
conda info -e
conda env list

# 查看当前环境下已安装的包
conda list

# 查看某个指定环境的已安装包
conda list -n venv

# 查找某package信息
conda search numpy

# 安装某package，-n指定安装到哪个虚拟环境中，不指定，则为当前虚拟环境
conda install -n venv numpy

# 更新某package
conda update -n venev numpy

# 删除某package
conda remove -n venv numpy
```

**pipenv**

```shell
# 1. 安装pipenv
pip install pipenv
# 2. 创建一个虚拟环境
$ cd myproject
$ pipenv install # 创建环境
$ pipenv install requests # 或者直接安装库
$ pipenv install requests==2.13.0 # 或者直接安装库
# 卸载包
$ pipenv uninstall requests

# 创建指定Python版本的虚拟环境
pipenv --python 3
pipenv --python 3.6
pipenv --python 2.7.14

# 3. 激活虚拟环境
$ pipenv shell
$ python --version
```



## 1.76 Python发邮件

[SMTP发送邮件](https://www.liaoxuefeng.com/wiki/1016959663602400/1017790702398272)

**POP3**

**POP3**是Post Office Protocol 3的简称，即邮局协议的第3个版本,它规定怎样将个人计算机连接到Internet的邮件服务器和下载电子邮件的电子协议。它是因特网电子邮件的第一个离线协议标准,POP3允许用户从服务器上把邮件存储到本地主机（即自己的计算机）上,同时删除保存在邮件服务器上的邮件，而POP3服务器则是遵循POP3协议的接收邮件服务器，用来接收电子邮件的。([与IMAP有什么区别？](http://help.163.com/10/0203/13/5UJONJ4I00753VB8.html?servCode=6010237))

**SMTP**

**SMTP** 的全称是“Simple Mail Transfer Protocol”，即简单邮件传输协议。它是一组用于从源地址到目的地址传输邮件的规范，通过它来控制邮件的中转方式。SMTP 协议属于 TCP/IP 协议簇，它帮助每台计算机在发送或中转信件时找到下一个目的地。SMTP 服务器就是遵循 SMTP 协议的发送邮件服务器。 
　　SMTP 认证，简单地说就是要求必须在提供了账户名和密码之后才可以登录 SMTP 服务器，这就使得那些垃圾邮件的散播者无可乘之机。 
　　增加 SMTP 认证的目的是为了使用户避免受到垃圾邮件的侵扰。

**IMAP**

**IMAP**全称是Internet Mail Access Protocol，即交互式邮件存取协议，它是跟POP3类似邮件访问标准协议之一。不同的是，开启了IMAP后，您在电子邮件客户端收取的邮件仍然保留在服务器上，同时在客户端上的操作都会反馈到服务器上，如：删除邮件，标记已读等，服务器上的邮件也会做相应的动作。所以无论从浏览器登录邮箱或者客户端软件登录邮箱，看到的邮件以及状态都是一致的。（[与POP3有什么区别？](http://help.163.com/10/0203/13/5UJONJ4I00753VB8.html?servCode=6010237)）

**MIME**

**MIME**(Multipurpose Internet Mail Extensions)多用途互联网邮件扩展类型。是设定某种[扩展名](https://baike.baidu.com/item/扩展名/103577)的[文件](https://baike.baidu.com/item/文件/6270998)用一种[应用程序](https://baike.baidu.com/item/应用程序/5985445)来打开的方式类型，当该扩展名文件被访问的时候，[浏览器](https://baike.baidu.com/item/浏览器/213911)会自动使用指定应用程序来打开。多用于指定一些客户端自定义的文件名，以及一些媒体文件打开方式。

**网易163免费邮箱相关服务器信息：**

![163免费邮客户端设置的POP3、SMTP、IMAP地址](Python学习笔记.assets/download.png) 



**通常SMTP用于发送邮件，POP3用于接收邮件。**

需要开启SMTP服务。

TODO



yagmail



smtplib和email



Python发送邮件的逻辑很简单：

> 1. 构造邮件对象，包含邮件标题，正文，附件等等。
> 2. 登录SMTP服务器
> 3. 发送邮件给SMTP服务器，然后SMTP服务器会转发给你的收件人。

```python
# smtplib 用于邮件的发信动作
import smtplib
from email.mime.text import MIMEText
# email 用于构建邮件内容
from email.header import Header
# 用于构建邮件头
 
# 发信方的信息：发信邮箱，QQ 邮箱授权码
from_addr = 'xxx@qq.com'
password = '你的授权码数字'
 
# 收信方邮箱
to_addr = 'xxx@qq.com'
 
# 发信服务器
smtp_server = 'smtp.qq.com'
 
# 邮箱正文内容，第一个参数为内容，第二个参数为格式(plain 为纯文本)，第三个参数为编码
msg = MIMEText('send by python','plain','utf-8')
 
# 邮件头信息
msg['From'] = Header(from_addr)
msg['To'] = Header(to_addr)
msg['Subject'] = Header('python test')
 
# 开启发信服务，这里使用的是加密传输
server = smtplib.SMTP_SSL()
server.connect(smtp_server,465)
# 登录发信邮箱
server.login(from_addr, password)
# 发送邮件
server.sendmail(from_addr, to_addr, msg.as_string())
# 关闭服务器
server.quit()

```

执行出现问题，163的邮箱服务器提示**认证失败**。

```shell
(learnpython) PS D:\work\python-project> & C:/Users/sjl/Anaconda3/envs/learnpython/python.exe d:/work/python-project/test-python/send_mail.py
error (535, b'Error: authentication failed')
```

认证失败的原因是，现在都是用授权码授权登录了，而不是通过**用户名/密码**的方式了。可以看到授权码管理部分，增加新的授权码使用设备。

尝试再发一次邮件看看。

```shell
(learnpython) PS D:\work\python-project> & C:/Users/sjl/Anaconda3/envs/learnpython/python.exe d:/work/python-project/test-python/send_mail.py
success
(learnpython) PS D:\work\python-project>
```

![image-20210401200823242](Python学习笔记.assets/image-20210401200823242.png)

## 1.77 Python构建和分发工具

[花了两天，终于把 Python 的 setup.py 给整明白了](https://zhuanlan.zhihu.com/p/276461821)

术语

- [Python Packaging Index](https://PyPI.org/) （PyPI）是一个开源许可的软件包公共存储库，可供所有 Python 用户使用。
- [Python Packaging Authority](https://www.pypa.io/) 是负责标准打包工具以及相关元数据和文件格式标准维护与改进的开发人员和文档作者团队。 他们基于 [GitHub](https://github.com/pypa) 和 [Bitbucket](https://bitbucket.org/pypa/) 这两个平台维护着各种工具、文档和问题追踪系统。
- [`distutils`](https://docs.python.org/zh-cn/3/library/distutils.html#module-distutils) 是1998年首次添加到Python标准库的原始构建和分发系统。虽然直接使用 [`distutils`](https://docs.python.org/zh-cn/3/library/distutils.html#module-distutils) 正在逐步淘汰，但它仍然为当前的打包和分发基础架构奠定了基础它不仅仍然是标准库的一部分，而且它的名称还以其他方式存在（例如用于协调Python包装标准开发的邮件列表的名称）。
- [setuptools](https://setuptools.readthedocs.io/en/latest/) （在很大程序上）是作为 [`distutils`](https://docs.python.org/zh-cn/3/library/distutils.html#module-distutils) 的取代者，于 2004 年首次发布。它对未经修改的 [`distutils`](https://docs.python.org/zh-cn/3/library/distutils.html#module-distutils) 工具最重要的补充是能够声明对其他包的依赖。目前它被推荐用来替代 [`distutils`](https://docs.python.org/zh-cn/3/library/distutils.html#module-distutils)，其更新更为频繁，在更为多样的 Python 版本之上为最新的包标准提供持续支持。
- [wheel](https://wheel.readthedocs.io/) （在此上下文中）是一个将 `bdist_wheel` 命令添加到 [`distutils`](https://docs.python.org/zh-cn/3/library/distutils.html#module-distutils)/[setuptools](https://setuptools.readthedocs.io/en/latest/) 的项目。这产生了一个跨平台的二进制打包格式（称为“轮子”或“轮子文件”，并在 [**PEP 427**](https://www.python.org/dev/peps/pep-0427)中定义），它允许在系统上安装Python库，甚至包括二进制扩展的库，而不需在本地进行构建。

```shell
python -m pip install setuptools wheel twine

pip install setuptools wheel twine
```



### 打包Python项目

如何打包Python项目

在本地创建一个项目，项目结构如下：

```shell
packaging_tutorial/
└── src/
    └── example_pkg/
        └── __init__.py
```

现在`cd packaging_tutorial`，进入项目根目录。

在项目根目录中创建如下这些文件，来打包项目和为分发项目做准备。

```shell
packaging_tutorial/
├── LICENSE
├── pyproject.toml # 告诉构建工具，如`pip`和`build`，你用的是什么构建系统，
				   # 构建所需的依赖是哪些
├── README.md
├── setup.cfg
├── setup.py  # optional, needed to make editable pip installs work
├── src/
│   └── example_pkg/
│       └── __init__.py
└── tests/ # 单元测试相关文件
```

```python
# pyproject.toml
[build-system]
requires = [
    "setuptools>=42",
    "wheel"
]
build-backend = "setuptools.build_meta"
```

`build-system.requires`指明了构建我的项目所需的包是哪些。

如何构建Python包

如何将Python包上传到PyPI。

### 配置元数据

有两种类型的元数据：**静态元数据**和**动态元数据**。咱选一种就可以了。

**静态元数据(setup.cfg)**: 每次构建都是一样的，简单易用。

**动态元数据(setup.py)**: 可能不确定。 任何在安装时动态或确定的项目，以及扩展模块或`setuptools`的扩展，都需要进入`setup.py`。

`setup.cfg`是`setuptools`的配置文件，它告诉`setuptools`你的包信息和包含的代码文件是哪些。

```cfg
[metadata]
name = example-pkg-YOUR-USERNAME-HERE # 包名
version = 0.0.1 
author = Example Author 
author_email = author@example.com
description = A small example package
long_description = file: README.md
long_description_content_type = text/markdown # 指明long_description的文件类型
url = https://github.com/pypa/sampleproject # 项目主页
project_urls =
    Bug Tracker = https://github.com/pypa/sampleproject/issues # PyPI中显示的额外链接
classifiers =
    Programming Language :: Python :: 3
    License :: OSI Approved :: MIT License
    Operating System :: OS Independent

[options]
package_dir =
    = src # 指明此包的python源码所在目录
packages = find: # 指明此包中所有导入的python包的列表，`find:` 表示去package_dir中自动发现这些包
python_requires = >=3.6 # 此包所支持的Python版本

[options.packages.find]
where = src
```

`setup.py`是`setuptools`的构建脚本，它告诉`setuptools`你的包信息和包含的代码文件是哪些。

```python
# setup.py

import setuptools

 
with open("README.md", "r", encoding="utf-8") as fh:
    long_description = fh.read()

setuptools.setup(
    name = "my-first-package",
    version="0.0.1",
    author="sjl",
    author_email="497245423@qq.com",
    description="a test package for sjl",
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="https://github.com/sjl-is-big-idiot/my-first-package",
    project_urls={
        "Bug Tracker": "https://github.com/sjl-is-big-idiot/my-first-package/i    ssues",
    },
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    package_dir={"": "src"},
    packages=setuptools.find_packages(where="src"),
    python_requires=">=3.6",
) 
```

```markdown
# README.md

This is a simple example package. You can use
[Github-flavored Markdown](https://guides.github.com/features/mastering-markdown/)
to write your content.
```

从`setup.py`中可知，`setup.py`会读取`README.md`的内容作为`long_description`。

### 创建许可

上传到PyPI的每个包都必须有许可证。它告诉要安装你所发布的软件包的用户关于此包的使用条款。有关选择许可证的帮助，请参阅https://choosealicense.com/。 选择许可证后，打开“许可证”并输入许可证文本。 例如，如果您选择了MIT许可证：

```txt
# LICENSE
Copyright (c) 2018 The Python Packaging Authority

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

### 生成分发包

确保安装了最新版本的`build`包。生成分发包之后，就可以上传到PyPI，且能够使用`pip`安装此包了。

```shell
# Unix/macOS
python3 -m pip install --upgrade build
# Windows
py -m pip install --upgrade build
```

进入到`pyproject.toml`文件所在目录，执行

```shell
python3 -m build
```

执行成功之后，新的目录结构如下所示，会在`dist`目录下生成两个文件：

```shell
.
├── build
│   ├── bdist.linux-x86_64
│   └── lib
│       └── example_pkg
│           └── __init__.py
├── dist
│   ├── my_first_package-0.0.1-py3-none-any.whl # 内置发行版
│   └── my-first-package-0.0.1.tar.gz  # 源码归档文件
├── LICENSE
├── pyproject.toml
├── README.md
├── setup.py
├── src
│   ├── example_pkg
│   │   └── __init__.py
│   └── my_first_package.egg-info
│       ├── dependency_links.txt
│       ├── PKG-INFO
│       ├── SOURCES.txt
│       └── top_level.txt
└── test
```

### 上传分发包

第一步，在[`Test PyPI`](https://test.PyPI.org/account/register/)注册一个账号。

第二步，创建PyPI [API token](https://test.PyPI.org/help/#apitoken)，用于安全上传你所创建的包。

第三步，使用[twine](https://packaging.python.org/key_projects/#twine) 上传分发包。

```shell
# Unix/macOS，安装twine
python3 -m pip install --user --upgrade twine
# Windows
py -m pip install --user --upgrade twine

# 上传`dist`目录下的所有归档文件
# Unix/macOS
python3 -m twine upload --repository testpypi dist/*
# Windows
py -m twine upload --repository testpypi dist/*
```
提示要输入用户名/密码。用户名使用__token__，密码使用token的值。
```
Uploading distributions to https://test.pypi.org/legacy/
Enter your username: [your username]
Enter your password:
Uploading example_pkg_YOUR_USERNAME_HERE-0.0.1-py3-none-any.whl
100%|█████████████████████| 4.65k/4.65k [00:01<00:00, 2.88kB/s]
Uploading example_pkg_YOUR_USERNAME_HERE-0.0.1.tar.gz
100%|█████████████████████| 4.25k/4.25k [00:01<00:00, 3.05kB/s]
```



### 使用`pip`安装你的包

第一步，创建一个虚拟环境，从TestPyPI安装你的包。

```shell
# Unix/macOS
python3 -m pip install --index-url https://test.pypi.org/simple/ --no-deps example-pkg-YOUR-USERNAME-HERE

# Windows
py -m pip install --index-url https://test.pypi.org/simple/ --no-deps example-pkg-YOUR-USERNAME-HERE


Collecting example-pkg-YOUR-USERNAME-HERE
  Downloading https://test-files.pythonhosted.org/packages/.../example-pkg-YOUR-USERNAME-HERE-0.0.1-py3-none-any.whl
Installing collected packages: example-pkg-YOUR-USERNAME-HERE
Successfully installed example-pkg-YOUR-USERNAME-HERE-0.0.1
```

第二步，测试是否安装成功了。

```shell
python3

>>> 
>>> import example_pkg
```



### 离线安装此包

#### 通过wheel二进制文件安装

```shell
pip3 install /path/to/xxx.wheel
```

#### 通过.tar.gz源码包安装

```shell
# 安装本地某指定的归档文件（包）
python3 -m pip install ./downloads/SomeProject-1.0.4.tar.gz

# 从某目录中安装归档文件（包）
python3 -m pip install --no-index --find-links=file:///local/dir/ SomeProject
python3 -m pip install --no-index --find-links=/local/dir/ SomeProject
python3 -m pip install --no-index --find-links=relative/dir/ SomeProject


```

其他命令：

```shell
# 安装requirements.txt文件指定的包
python3 -m pip install -r requirements.txt

# 从其它index安装某包
python3 -m pip install --index-url http://my.package.repo/simple/ SomeProject

python -m pip install --extra-index-url http://localhost:7777 SomeProject
```



## 1.78 Python发微信

TODO

## 1.79 时间字符串、时间戳、日期时间互相转换

### datetime转timestamp

直接使用datetime模块中datetime类的timestamp()实例方法。

```python
import datetime
import time

dt = datetime.datetime.now()
ts = dt.timestamp()
print(dt) # datetime.datetime(2019, 9, 11, 11, 20, 6, 681320)
print(ts) # 1568172006.68132
```

### datetime转时间字符串

直接使用datetime模块中的datetime类的strftime()实例方法即可。

```python
import datetime
import time

dt = datetime.datetime.now()
format = '%Y-%m-%d %H:%M:%S' # 根据此格式来解析datetime.datetime()对象为时间字符串

print(dt) # datetime.datetime(2019, 9, 11, 11, 20, 6, 681320)
print(dt.strftime(format)) # '2019-09-11 11:20:06'
```

### timestamp转datetime

```python
import datetime
import time

ts = 1568172006.68132 # 时间戳
dt = datetime.datetime.fromtimestamp(ts)

print(dt) # datetime.datetime(2019, 9, 11, 11, 20, 6, 681320)
```

### timestamp转时间字符串

转换必须通过`time.struct_time`对象作为桥梁。
先转成`datetime.datetime`对象，再转成时间字符串。

```python
import datetime
import time

# 方法1
ts = 1568172006.68132 # 时间戳
format = '%Y-%m-%d %H:%M:%S' # 根据此格式来时间戳解析为时间字符串

# 时间戳转time.struct_time
ts_struct = time.localtime(ts)

# time.struct_time转时间字符串
date_string = time.strftime(format, ts_struct)

print(date_string) # '2019-09-11 11:20:06'

# 方法2
dt = datetime.datetime.fromtimestamp(ts)
date_string = dt.strftime(format)
```



### 时间字符串转datetime

只需要使用datetime模块中的datetime类的strptime(date_string, format)类方法即可。
这个方法的作用就是：根据指定的format格式将时间字符串date_string，转换成`datetime.datetime()`对象。

```python
import datetime
import time

date_string = '2019-09-11 11:20:06'
format = '%Y-%m-%d %H:%M:%S' # 根据此格式来解析时间字符串为datetime.datetime()对象

dt = datetime.strptime(date_string, format)

print(dt) # datetime.datetime(2019, 9, 11, 11, 20, 6)
```

### 时间字符串转timestamp

转换必须通过`time.struct_time`对象作为桥梁。
先转成`datetime.datetime`对象，再转成`timestamp`。

```python
import datetime
import time

# 方法1
date_string = '2019-09-11 11:20:06'
format = '%Y-%m-%d %H:%M:%S' # 根据此格式来解析时间字符串为time()对象

# 时间字符串转time.struct_time
ts_struct = time.strptime(date_string, format)

# time.struct_time转时间戳
ts = time.mktime(ts_struct)

print(ts) # 1568172006.0

# 方法2
dt = datetime.datetime.strptime(date_string, format)
ts = dt.timestamp()
```