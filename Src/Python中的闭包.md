# Python中的闭包
## 关于闭包
> 嵌套函数  
> 外部函数返回值是一个内部函数  
> 内部函数需要调用外部函数的变量
## 闭包的核心意义
扩展了原有函数的作用域,可以保障变量的长期保存,用一个实例来进行说明
>设计一个存钱罐,每次存钱和取钱后都要返回存钱罐的余额
## 非闭包操作
### 函数+全局变量
Python中实现每一个功能其实都是有多种方案的,就好像数学题一样,是有很多的题解的,我们要做的只是选出其中最简洁、最方便、最稳定的一种。  
比如使用一个单独的函数就可以实现这个功能,代码如下:
```python
money = 0
def operator(num:float):
    global money  # 用来操作全局变量money
    if isinstance(num, (int, float)) and num:
        money += num
        if num > 0:
            print(f"你往账户里面存储了{num}元,账户余额为{money}")
        else:
            print(f"你从账户里面取出了{num}元,账户余额为{money}")
    else:
        print("你的操作违法,账户没有变化")
    return money
t = operator(100.5)
print(t)
t = operator(-100)
print(t)
```
这种可以通过一个函数+全局变量的方式来实现存钱罐的功能,但是这样存在一个问题,比如忘记了这个全局变量,在其他地方**引入并改变了这个全局变量**,就会导致项目出现严重的Bug,如下:
```python
money = 0
def operator(num:float):
    global money  # 用来操作全局变量money
    if isinstance(num, (int, float)) and num:
        money += num
        if num > 0:
            print(f"你往账户里面存储了{num}元,账户余额为{money}")
        else:
            print(f"你从账户里面取出了{num}元,账户余额为{money}")
    else:
        print("你的操作违法,账户没有变化")
    return money
t = operator(100.5)
print(t)
t = operator(-100)
print(t)
money = 500 # 模拟在其他地方误操作了这个全局变量
t = operator(60)  # 正常来说,存钱罐里面应该是60.5
print(t)
```
可以看到,使用全局变量的危害就是如果这个变量在某个地方被非法操作了,将会产生难以预料的Bug,那如果不使用全局变量,而是在函数里面设计一个变量的话,由于局部变量的生命周期将随着函数的返回而终止,将会导致存钱罐每次的金额都是0,代码如下:
```python
def operator(num:float):
    money = 0 # 每次函数运行时创建,结束时销毁
    if isinstance(num, (int, float)) and num:
        money += num
        if num > 0:
            print(f"你往账户里面存储了{num}元,账户余额为{money}")
        else:
            print(f"你从账户里面取出了{num}元,账户余额为{money}")
    else:
        print("你的操作违法,账户没有变化")
    return money
t = operator(100.5)
print(t)
t = operator(-100)
print(t)
t = operator(60) 
print(t)
```
### 通过类来实现
能通过闭包实现的函数,就可以通过类来实现,只是不够美观,为了一个单独的功能而单独写一个类,给人一种杀个鸡用大砍刀一样的感觉,代码如下:
```python
class Operator:
    def __init__(self):
        self.money = 0
    def mov(self, num):
        if isinstance(num, (int, float)) and num:
            self.money += num
            if num > 0:
                print(f"你往账户里面存储了{num}元,账户余额为{self.money}")
            else:
                print(f"你从账户里面取出了{num}元,账户余额为{self.money}")
        else:
            print("你的操作违法,账户没有变化")
        return self.money
operator1 = Operator()
operator1.mov(100)
operator1.mov(-60)
operator1.mov(50)
```
## 通过闭包实现
```python
def Money():
    money = [0]  # 这里选择用列表,而不是数字的方式将在后面进行介绍
    def operator(num):  # 函数是嵌套存在的,在一个函数里面存在另一个函数
        if isinstance(num, (int, float)) and num:
            nowMoney = money[0]  # 要用到外部函数定义的一个变量
            nowMoney += num
            money[0] = nowMoney
            if num > 0:
                print(f"你往账户里面存储了{num}元,账户余额为{nowMoney}")
            else:
                print(f"你从账户里面取出了{num}元,账户余额为{nowMoney}")
        else:
            print("你的操作违法,账户没有变化")
        return nowMoney
    return operator  # 返回值要是内部函数,且不要加()
M = Money()
M(100)
M(-50.8)
M(60.8)
```
这段代码开起来并不完美,因为明明可以将money定义成一个浮点数,但是这里却定义为了一个列表,这是因为作用域问题,如果使用不可变类型的数据(数值、字符串等更改了对应的值,则变量就会绑定到一个新的对象上),将会产生异常。代码如下:
```python
def Money():
    money = 0  # 这里选择用列表,而不是数字的方式将在后面进行介绍
    def operator(num):  # 函数是嵌套存在的,在一个函数里面存在另一个函数
        if isinstance(num, (int, float)) and num:
            money += num
            if num > 0:
                print(f"你往账户里面存储了{num}元,账户余额为{money}")
            else:
                print(f"你从账户里面取出了{num}元,账户余额为{money}")
        else:
            print("你的操作违法,账户没有变化")
        return money
    return operator  # 返回值要是内部函数,且不要加()
M = Money()
M(500) # local variable 'money' referenced before assignment
```
### 函数的作用域
当我们的函数使用某个变量时,如果函数内部没有定义,则Python会自动向上一级去查找这个变量,如下:
```python
num = 0
def count():
    print(num)
    print("hello")
count()
```
此时程序正常运行,没有报错,但是当我们没有global标识这个变量而在函数内部修改找个变量时,则会出现以下问题
```python
num = 0
def count():
    print("hello")
    num += 1
    print(num)
count() # local variable 'num' referenced before assignment
```
之所以出现以上报错,就是Python的作用域问题,或者说Python的机制,当你在函数内部修改某个变量时(非global和nonlocal装饰后的),Python会认为这个变量就是在函数内部定义的,即便内部并没有定义,即便全局存这个变量,也会认为是内部定义的,但是实际上内部并没有,所以就会产生Bug。
### nonlocal
```python
def Money():
    money = 0  # 这里选择用列表,而不是数字的方式将在后面进行介绍
    def operator(num):  # 函数是嵌套存在的,在一个函数里面存在另一个函数
        nonlocal money
        if isinstance(num, (int, float)) and num:
            money += num
            if num > 0:
                print(f"你往账户里面存储了{num}元,账户余额为{money}")
            else:
                print(f"你从账户里面取出了{num}元,账户余额为{money}")
        else:
            print("你的操作违法,账户没有变化")
        return money
    return operator  # 返回值要是内部函数,且不要加()
M = Money()
M(500)
M(-300)
M(200.8)
```
### 关于nonlocal和global
> 引用知乎作者初识CV的介绍
> 
>第一，两者的功能不同。global关键字修饰变量后标识该变量是全局变量，对该变量进行修改就是修改全局变量，而nonlocal关键字修饰变量后标识该变量是上一级函数中的局部变量，如果上一级函数中不存在该局部变量，nonlocal位置会发生错误（最上层的函数使用nonlocal修饰变量必定会报错）。  
>
>第二，两者使用的范围不同。global关键字可以用在任何地方，包括最上层函数中和嵌套函数中，即使之前未定义该变量，global修饰后也可以直接使用，而nonlocal关键字只能用于嵌套函数中，并且外层函数中定义了相应的局部变量，否则会发生错误（见第一）

<a href='https://zhuanlan.zhihu.com/p/341378844'>关于nonlocal和global详细介绍</a>

