---
title: "字符串变量替换"
date: 2023-04-11T20:17:11+08:00
draft: false
---
# 需求
在yaml中需要引用自定义变量，我们参照 postman，约定变量符号为 `{{}}`。
# 思路
我们知道在 python 中字符串格式化可以通过 `.format()` 方法来处理。
比如

```python
>>> name = lee
>>> "i am {}".format(name)

i am lee
```
但我们所需字符串中约定的变量符号为 `{{}}` ，因此源码的 format 方法是不行的，为此我们可以重写字符串的 format 方法来达到我们的需求。

继承 UserString 重写其 format 方法即可
```python
import re
from collections import UserString

class String(UserString):
    def __init__(self, seq: object):
        super().__init__(seq)

    def format(self, /, **kwds) -> str:
        pattern_param = r"{{(\w+)}}"
        if params := list(set(re.findall(pattern_param, self.data))):
            for param in params:
                if value := kwds.get(param, ""):
                    rex = f"{{param}}"
                    self.data = re.sub(rex, str(value), self.data)
        return self.data
```
测试

```python
>>> d = {"a": 1, "b": 2}
>>> String("{hello, {{a}}, world {{b}}}").format(**d)

{hello, 1, world 2}
```
# 额外知识
看源码的时候我纳闷 format 的参数定义怎么是这样的。

```python
def format(self, /, *args, **kwds):
    return self.data.format(*args, **kwds)
```


怎么有一个形参是`/`，难道通过`/`传参？心里想不可能吧。网上搜又不好搜。结果今儿无意中刷到了数据派THU的一条微博。我点开来一看，woc这不是我前几天遇到的问题嘛！
真的是out了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/334c4f28a6ca45dda52e59be5c6b6036.png)
最后研究了一下，其实是python3.8的新特性。👉 [Python 3.8 有什么新变化？](https://docs.python.org/zh-cn/3.8/whatsnew/3.8.html)

它加了一个新特性：仅限位置形参 ( positional-only arguments )。
具体解释如下：
### 仅限位置形参
定义函数的形参时可以指定两个分隔符，分别为`/`， `*`。

在下面的例子中，形参 a 和 b 为仅限位置形参，c 或 d 可以是位置形参或关键字形参，而 e 或 f 要求为关键字形参。

```python
def f(a, b, /, c, d, *, e, f):
    print(a, b, c, d, e, f)
```
以下均为合法的调用:

```python
f(10, 20, 30, d=40, e=50, f=60)
```

但是，以下均为不合法的调用:

```python
f(10, b=20, c=30, d=40, e=50, f=60)   # b cannot be a keyword argument
f(10, 20, 30, 40, 50, f=60)           # e must be a keyword argument
```

这种标记形式的一个用例是它允许纯 Python 函数完整模拟现有的用 C 代码编写的函数的行为。 例如，内置的 divmod() 函数不接受关键字参数:

```python
def divmod(a, b, /):
    "Emulate the built in divmod() function"
    return (a // b, a % b)
```

另一个用例是在不需要形参名称时排除关键字参数。 例如，内置的 len() 函数的签名为 len(obj, /)。 这可以排除如下这种笨拙的调用形式:

```python
len(obj='hello')  # The "obj" keyword argument impairs readability
```

另一个益处是将形参标记为仅限位置形参将允许在未来修改形参名而不会破坏客户的代码。 例如，在 statistics 模块中，形参名 dist 在未来可能被修改。 这使得以下函数描述成为可能:

```python
def quantiles(dist, /, *, n=4, method='exclusive')
    ...
```

由于在 / 左侧的形参不会被公开为可用关键字，其他形参名仍可在 **kwargs 中使用:

```python
>>>
>>> def f(a, b, /, **kwargs):
...     print(a, b, kwargs)
...
>>> f(10, 20, a=1, b=2, c=3)         # a and b are used in two ways
10 20 {'a': 1, 'b': 2, 'c': 3}
```

这极大地简化了需要接受任意关键字参数的函数和方法的实现。 例如，以下是一段摘自 collections 模块的代码:

```python
class Counter(dict):

    def __init__(self, iterable=None, /, **kwds):
        # Note "iterable" is a possible keyword argument。
```
还有几个特性：
1. 赋值表达式，也叫海象运算符: `:=`。

```python
# Loop over fixed length blocks
while (block := f.read(256)) != '':
    process(block)
```

2. f-string = 用于自动记录表达式和调试文档，也就是加个=直接输出值。

```python
>>> user = 'eric_idle'
>>> member_since = date(1975, 7, 31)
>>> f'{user=} {member_since=}'
...

"user='eric_idle' member_since=datetime.date(1975, 7, 31)"
```
其他的参考官方文档。
# 参考链接
[Python 3.8 有什么新变化](https://docs.python.org/zh-cn/3.8/whatsnew/3.8.html)