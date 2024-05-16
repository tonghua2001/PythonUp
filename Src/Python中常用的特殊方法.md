# Python中常用的特殊方法
> 注意事项:
> 1. 特殊方法是交给Python解释器使用的,而不是自己
> 2. 特殊方法是隐式调用的
> 3. 编程中大部分时间是用来实现特殊方法,而不是调用特殊方法,除了__init__外的特殊方法基本上都是通过内置函数来隐式调用的
> 4. Python中特殊方法的标记是在方法前后用双下划线,例如__len__
****

## \_\_len__特殊方法的作用:len()函数的调用
假设我们有这样一个需求:  
> 存在一个班级对象,里面存放着一个有关学生的列表  
> 班级里面可以添加新的学生和去除转校学生  
> 需要在任意时间段获取班级里面的学生人数

先定义一个班级对象
```python
class Classes:
    def __init__(self, class_name):
        self.class_name = class_name  # 班级编号
        self.students = []  # 初始班级是空的
    def add(self, name):
        self.students.append(name)  # 班级里面加入新生
    def remove(self, name):
        self.students.remove(name)  # 班级里面某个学生转出去
S_Classes = Classes("S班")
# 此时无法成功获取到班级里面的人数
print(len(S_Classes))  # TypeError: object of type 'Classes' has no len()
```
报错信息显示,Classes这个类里面没有len方法,所以使用时会报错,如果我们在类里面添加一个len方法
```python
class Classes:
    def __init__(self, class_name):
        self.class_name = class_name  # 班级编号
        self.students = []  # 初始班级是空的
    def add(self, name):
        self.students.append(name)  # 班级里面加入新生
    def remove(self, name):
        self.students.remove(name)  # 班级里面某个学生转出去
    def len(self):
        return len(self.students)
S_Classes = Classes("S班")
# 此时无法成功获取到班级里面的人数
# print(len(S_Classes))  # TypeError: object of type 'Classes' has no len()
print(S_Classes.len())  # 0
```
事实证明,特殊方法并不是必须的,即便是普通的类方法也一样可以实现我们需要的功能,但是不够美观,比如无法使用len(对象)这种优雅的方法获取对象的大小。
```python
class Classes:
    def __init__(self, class_name):
        self.class_name = class_name  # 班级编号
        self.students = []  # 初始班级是空的
    def add(self, name):
        self.students.append(name)  # 班级里面加入新生
    def remove(self, name):
        self.students.remove(name)  # 班级里面某个学生转出去
    def __len__(self):
        return len(self.students)
S_Classes = Classes("S班")
print(len(S_Classes))  # 0
S_Classes.add("童话")
S_Classes.add("掘金")
print(len(S_Classes))  # 2
```
****
## \_\_getitem__的作用:让对象可以被索引和遍历
> 新增了一个需求,想要遍历班级里面的学生

这个时候,就需要用到__getitem__这个方法了,当然,不用这个特殊方法问题也可以得到解决,代码如下:
```python
class Classes:
    def __init__(self, class_name):
        self.class_name = class_name  # 班级编号
        self.students = []  # 初始班级是空的
    def add(self, name):
        self.students.append(name)  # 班级里面加入新生
    def list_of_students(self):
        for i in self.students:
            yield i

S_Classes = Classes("S班")
S_Classes.add("童话")
S_Classes.add("掘金")
for i in S_Classes.list_of_students():
    print(i)
```
通过上面两个实例可以看到,特殊方法不一定要写,用普通函数也可以一样实现这个功能,但是用特殊方法可以更简洁,例如下面的代码:
```python
class Classes:
    def __init__(self, class_name):
        self.class_name = class_name  # 班级编号
        self.students = []  # 初始班级是空的
    def add(self, name):
        self.students.append(name)  # 班级里面加入新生
    def remove(self, name):
        self.students.remove(name)  # 班级里面某个学生转出去
    # 有了这个特殊方法才能使用len()方法
    def __len__(self):
        return len(self.students)
    # __getitem__获取某个元素
    def __getitem__(self, index):
        return self.students[index]

S_Classes = Classes("S班")
print(len(S_Classes))  # 0
S_Classes.add("童话")
S_Classes.add("掘金")
print(len(S_Classes))  # 2
for i in S_Classes:
    print(i)
```
实现了__getitem__方法能做的不仅仅是遍历,还可以根据索引获取内容和对序列进行切片
```python
class Classes:
    def __init__(self, class_name):
        self.class_name = class_name  # 班级编号
        self.students = []  # 初始班级是空的
    def add(self, name):
        self.students.append(name)  # 班级里面加入新生
    def remove(self, name):
        self.students.remove(name)  # 班级里面某个学生转出去
    # 有了这个特殊方法才能使用len()方法
    def __len__(self):
        return len(self.students)
    # __getitem__获取某个元素
    def __getitem__(self, index):
        return self.students[index]

S_Classes = Classes("S班")
S_Classes.add("童话")
S_Classes.add("掘金")
S_Classes.add("123")
S_Classes.add("456")
S_Classes.add("789")
S_Classes.add("JQK")
print(S_Classes[1])
print(S_Classes[0:5:2])
```
****
## \_\_repr__:打印对象更美观
如果我们这个时候去打印班级对象,得到的是对象的内存属性,这些内容并不符合人类的感官
```python
class Classes:
    def __init__(self, class_name):
        self.class_name = class_name  # 班级编号
        self.students = []  # 初始班级是空的
    def add(self, name):
        self.students.append(name)  # 班级里面加入新生

S_Classes = Classes("S班")
print(S_Classes)  # <__main__.Classes object at 0x000001DF1E12C0A0>
```
为了能够打印出更加美观的对象信息,方便后面使用这个对象的人可以通过打印这个对象从而获取到对象的关键信息,可以使用__repr__这个特殊方法,代码如下:
```python
class Classes:
    def __init__(self, class_name):
        self.class_name = class_name  # 班级编号
        self.students = []  # 初始班级是空的
    def add(self, name):
        self.students.append(name)  # 班级里面加入新生
    def remove(self, name):
        self.students.remove(name)  # 班级里面某个学生转出去
    # 有了这个特殊方法才能使用len()方法
    def __len__(self):
        return len(self.students)
    # __getitem__获取某个元素
    def __getitem__(self, index):
        return self.students[index]
    def __repr__(self):
        return f"{self.class_name}({len(self.students)})"

S_Classes = Classes("S班")
S_Classes.add("童话")
S_Classes.add("掘金")
S_Classes.add("123")
S_Classes.add("456")
S_Classes.add("789")
S_Classes.add("JQK")
print(S_Classes) # S班(6)
```
****
## \_\_add__: 两个相同的类实例相加
```python
class Classes:
    def __init__(self, class_name, students=None):
        self.class_name = class_name  # 班级编号
        self.students = [] if not students else students  # 初始班级是空的
    def add(self, name):
        self.students.append(name)  # 班级里面加入新生
    def remove(self, name):
        self.students.remove(name)  # 班级里面某个学生转出去
    # 有了这个特殊方法才能使用len()方法
    def __len__(self):
        return len(self.students)
    # __getitem__获取某个元素
    def __getitem__(self, index):
        return self.students[index]
    # 控制打印
    def __repr__(self):
        return f"{self.class_name}({len(self.students)})"
    def __add__(self, other):
        students = self.students + other.students
        return Classes("新班级", students)  # 注意这里返回的是一个新对象


S_Classes = Classes("S班")
S_Classes.add("童话")
S_Classes.add("掘金")
S_Classes.add("123")
S_Classes.add("456")
S_Classes.add("789")
S_Classes.add("JQK")
F_Classes = Classes("一班")
F_Classes.add("857")
F_Classes.add("798")
F_Classes.add("1314")
# T_Classes = S_Classes + F_Classes
# print(S_Classes, id(S_Classes))
# print(F_Classes, id(F_Classes))
# print(T_Classes, id(T_Classes))
# print(T_Classes)
print(S_Classes, id(S_Classes))
S_Classes += F_Classes  # S_Classes绑定的是一个新的对象
print(S_Classes, id(S_Classes))
print(S_Classes)
```
****
## 总结
> Python中的特殊方法并不是自己调用,或者说一般情况下不会自己调用,比如__len__方法交给len()函数,__getitem__交给for循环或者对象索引,也从侧面证明了特殊方法是交给Python解释器的  
> 显示调用,例如 对象.--len--,隐式调用,例如len(对象),从上面的代码中也可以验证这个说法  
> 除了上述提到的几个特殊方法,Python中还存在许多的特殊方法, <a href='https://blog.csdn.net/qq_37312720/article/details/84788074#:~:text=python%E4%B8%AD%E4%B8%80%E5%85%B1%E6%9C%89%2083,%E4%B8%AA%E7%89%B9%E6%AE%8A%E6%96%B9%E6%B3%95%EF%BC%8C%E5%85%B6%E4%B8%AD%2047%20%E4%B8%AA%E7%94%A8%E4%BA%8E%E7%AE%97%E6%9C%AF%E8%BF%90%E7%AE%97%E3%80%81%E4%BD%8D%E8%BF%90%E7%AE%97%E5%92%8C%E6%AF%94%E8%BE%83%E6%93%8D%E4%BD%9C%E3%80%82'>详情可见</a>