# python | 详解read、readline、readlines区别

看了很多博主写的文章，感觉解释的不是很详细，本文结合实际操作详细记录了个人对于read、readline、readlines用法的剖解过程。
# 函数解释及区别
## read
默认读取全部内容（即不指定n），返回字符串。

**函数原型**

```python
    def read(self, n: int = -1) -> AnyStr:
        pass
```

默认读取整个文件，但有以下区别：
* 返回值的类型是字符串。
* 读取的最小单位为字符数
* 最大读取取决于n。
* 最小读取取决于n。
## readline
默认读取一行内容（即不指定limit）返回一个字符串。

**函数原型**

```python
    def readline(self, limit: int = -1) -> AnyStr:
        pass
```

与readlines字面意思一样，也是按**行**读取，但有以下区别：
* 返回值的类型是字符串。
* 最小读取单位为字符数
* 最小读取取决于limit
* 最大读取为一行
## readlines
默认读取所有内容（即不指定hint），按行返回一个列表，一行内容作为列表的一个元素。

**函数原型**

```python
    def readlines(self, hint: int = -1) -> List[AnyStr]:
        pass
```


与readline字面意思一样，也是按**行**读取，但有以下区别：
* 返回值的类型是列表。
* 最小读取单位为行。
* 最大读取取决于hint。
* 最小读取为一行。
# 实际操作
假设有以下文本内容

```bash
hello world1
hello world2
hello world3
hello world4
hello world5
hello world6
hello world7
```
## read

**默认读取**

```python
def load():
	path = r'D:\demo.txt'
	with open(pa, 'r', encoding='utf-8') as f:
		return f.read()
		
if __name__ == '__main__':
	print(repr(load()))
	
# 输出
'hello world1\nhello world2\nhello world3\nhello world4\nhello world5\nhello world6\nhello world7'

```
**一次读取10个字符**

```python
def load():
	path = r'D:\demo.txt'
	with open(pa, 'r', encoding='utf-8') as f:
		return f.read(10)
		
if __name__ == '__main__':
	print(repr(load()))
	
# 输出
'hello worl'

```
## readline

**默认读取**(会将行尾的换行符\n也视为该行一起读取)

```python
def load():
	path = r'D:\demo.txt'
	with open(pa, 'r', encoding='utf-8') as f:
		return f.readline()
		
if __name__ == '__main__':
	print(repr(load()))
	
# 输出
'hello world1\n'

```
**指定一次读取2个字符**
```python
def load():
	path = r'D:\demo.txt'
	with open(pa, 'r', encoding='utf-8') as f:
		return f.readline(2)
		
if __name__ == '__main__':
	print(repr(load()))
	
# 输出
'he'

```
**指定一次读取15个字符**， 可以猜一下输出什么？
`hello world1\nhe`?

```python
def load():
	path = r'D:\demo.txt'
	with open(pa, 'r', encoding='utf-8') as f:
		return f.readline(15)
		
if __name__ == '__main__':
	print(repr(load()))
	
# 输出
'hello world1\n'

```
可以看到并没有将第二行读取进来，也印证了上面我们说：
readline 最大读取为一行，最小读取取决于limit。
## readlines

**默认读取**(会将行尾的换行符\n也视为该行一起读取)

```python
def load():
	path = r'D:\demo.txt'
	with open(pa, 'r', encoding='utf-8') as f:
		return f.readlines()
		
if __name__ == '__main__':
	print(repr(load()))
	
# 输出
['hello world1\n', 'hello world2\n', 'hello world3\n', 'hello world4\n', 'hello world5\n', 'hello world6\n', 'hello world7']

```
**指定一次读取5个字符**，猜一猜输出什么？
`['hello']` ?
```python
def load():
	path = r'D:\demo.txt'
	with open(pa, 'r', encoding='utf-8') as f:
		return f.readlines(5)
		
if __name__ == '__main__':
	print(repr(load()))
	
# 输出
['hello world1\n']

```
并没有。 readlines 最小读取为一行。

**指定一次读取15个字符**，猜一猜输出什么？
`[‘hello world1\nhe’]` ?
```python
def load():
	path = r'D:\demo.txt'
	with open(pa, 'r', encoding='utf-8') as f:
		return f.readlines(15)
		
if __name__ == '__main__':
	print(repr(load()))
	
# 输出
['hello world1\n', 'hello world2\n']

```
并没有，它读取了两行。 当字符数不够一行时，会将剩余字符一起读进来。所以readlines 的最小读取单位为行。


