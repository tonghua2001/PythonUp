# Python中的装饰器
Python中的装饰器就是将一个函数作用到另一个函数上,作为装饰器的函数返回值应该是一个函数,它的作用就是**可以对函数进行软编码,就是在不更改原函数的情况下,为函数添加更多的功能**,而且一个装饰器可以作用于多个函数
## 一个简单的装饰器
>设计一个功能,能够将函数运行时候传入的参数、执行时间、返回值内容写入到日志文件里面(这里以txt文件做示范)

原始函数是这样的
```python
import requests
def guessUrl(url, **kwargs):
    """
        判断url是否可以访问
    """
    headers = {
        'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36 Edg/124.0.0.0'
    }
    try:
        response = requests.get(url, headers=headers, timeout=10, **kwargs)
        if response.status_code == 200:
            return "该链接可以正常访问"
        else:
            return f"该链接访问状态异常[{response.status_code}]"
    except Exception as e:
        return "该链接无法访问"
urls = [
    "http://httpbin.org/get",
    "http://httpbin.org/post",
    "https://httpbin.org/get",
    "https://httpbin.org/post",
    "http://www.baidu.com",
    "http://www.baidu1234.com",
    "http://www.kanba.com",
    "http://www.hello.com",
]
for url in urls:
    print(guessUrl(url))
```
如果不使用装饰器,硬编码函数会变成这样
```python
import requests
import time


def guessUrl(url, **kwargs):
    """
        判断url是否可以访问
    """
    func_start = time.time()
    headers = {
        'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36 Edg/124.0.0.0'
    }
    try:
        response = requests.get(url, headers=headers, timeout=10, **kwargs)
        if response.status_code == 200:
            result = "该链接可以正常访问"
        else:
            result = f"该链接访问状态异常[{response.status_code}]"
    except Exception as e:
        result = "该链接无法访问"
    finally:
        func_stop = time.time()
        with open('./func.txt', 'a', encoding='utf-8')as f:
            f.write(f"[{(func_stop-func_start):0.8f}]guessUrl('{url}') -> {result}"+"\n")
        return result

urls = [
    "http://httpbin.org/get",
    "http://httpbin.org/post",
    "https://httpbin.org/get",
    "https://httpbin.org/post",
    "http://www.baidu.com",
    "http://www.baidu1234.com",
    "http://www.kanba.com",
    "http://www.hello.com",
]
for url in urls:
    print(guessUrl(url))
```
这样的做法,无疑会使函数变得越发臃肿,而且由于每次功能修改都是在函数内部,可能会导致函数的可读性大幅度下降,而且这些附加功能其实跟函数的实际用途有着极大的差别,如果使用装饰器则可以在很大程度上缓解这个**函数附加功能**的问题,代码如下:
```python
import requests
import time
def log(func):
    def write(*args, **kwargs):
        func_start = time.time()
        result = func(*args, **kwargs)
        func_stop = time.time()
        with open('./func.txt', 'a', encoding='utf-8')as f:
            f.write(f"[{(func_stop-func_start):0.8f}]guessUrl('{url}') -> {result}"+"\n")
        return result
    return write


@log
def guessUrl(url, **kwargs):
    """
        判断url是否可以访问
    """
    headers = {
        'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36 Edg/124.0.0.0'
    }
    try:
        response = requests.get(url, headers=headers, timeout=10, **kwargs)
        if response.status_code == 200:
            return "该链接可以正常访问"
        else:
            return f"该链接访问状态异常[{response.status_code}]"
    except Exception as e:
        return "该链接无法访问"
urls = [
    "http://httpbin.org/get",
    "http://httpbin.org/post",
    "https://httpbin.org/get",
    "https://httpbin.org/post",
    "http://www.baidu.com",
    "http://www.baidu1234.com",
    "http://www.kanba.com",
    "http://www.hello.com",
]
for url in urls:
    print(guessUrl(url))
```
## 装饰器是什么
> 1.装饰器是一个闭包函数  
> 2.装饰器通过@语法糖作用在被装饰函数上  
> 3.执行被装饰函数,实际上是执行log(guessUrl)  
> 4.装饰器优势在于可以通过软编码的方式为某个函数添加一些独立的附加功能
## 装饰器在编译函数的时候运行
```python
import requests
import time
def log(func):
    print("log装饰器开始运行了")
    def write(*args, **kwargs):
        print("装饰器Write开始运行了")
        func_start = time.time()
        result = func(*args, **kwargs)
        func_stop = time.time()
        with open('./func.txt', 'a', encoding='utf-8')as f:
            f.write(f"[{(func_stop-func_start):0.8f}]guessUrl('{url}') -> {result}"+"\n")
        return result
    return write


@log
def guessUrl(url, **kwargs):
    print(f"函数guessUrl('{url}')开始运行了")
    """
        判断url是否可以访问
    """
    headers = {
        'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36 Edg/124.0.0.0'
    }
    try:
        response = requests.get(url, headers=headers, timeout=10, **kwargs)
        if response.status_code == 200:
            return "该链接可以正常访问"
        else:
            return f"该链接访问状态异常[{response.status_code}]"
    except Exception as e:
        return "该链接无法访问"
urls = [
    "http://httpbin.org/get",
    "http://httpbin.org/post",
    "https://httpbin.org/get",
    "https://httpbin.org/post",
    "http://www.baidu.com",
    "http://www.baidu1234.com",
    "http://www.kanba.com",
    "http://www.hello.com",
]
for url in urls:
    print(guessUrl(url))
```
通过上面打印的信息可以看到,装饰器外部函数在函数编译时运行,且只会运行一次,内部函数在运行被装饰函数时执行(由它调用被装饰函数),甚至可以修改内部函数,让被装饰函数不会执行,代码如下:
```python
import requests
import time
def log(func):
    print("log装饰器开始运行了")
    def write(*args, **kwargs):
        print("装饰器Write开始运行了")
        print("我才不要执行被装饰函数呢!")
        return "我是内部函数返回值,我想返回啥就返回啥"
    return write


@log
def guessUrl(url, **kwargs):
    print(f"函数guessUrl('{url}')开始运行了")
    """
        判断url是否可以访问
    """
    headers = {
        'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36 Edg/124.0.0.0'
    }
    try:
        response = requests.get(url, headers=headers, timeout=10, **kwargs)
        if response.status_code == 200:
            return "该链接可以正常访问"
        else:
            return f"该链接访问状态异常[{response.status_code}]"
    except Exception as e:
        return "该链接无法访问"
urls = [
    "http://httpbin.org/get",
    "http://httpbin.org/post",
    "https://httpbin.org/get",
    "https://httpbin.org/post",
    "http://www.baidu.com",
    "http://www.baidu1234.com",
    "http://www.kanba.com",
    "http://www.hello.com",
]
for url in urls:
    print(guessUrl(url))
```
可以看到,如果一个函数被装饰器装饰了,那么它是否被真正使用,装饰器在其中有很大的话语权
