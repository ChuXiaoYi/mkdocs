该模块实现了专门的容器数据类型，为Python的通用内置容器`dict`，`list`，`set`和`tuple`提供了替代方案。

## **namedtuple()**
包含命名字段的元组工厂方法
命名元组为元组中的每个位置赋予含义，并允许更可读，自文档代码。 
它们可以在使用常规元组的任何地方使用，并且它们添加了按名称而不是位置索引访问字段的功能。

实现：
```python
collections.namedtuple(typename, field_names, *, rename=False, defaults=None, module=None)
```

- 返回一个名为`typename`的新元组子类。 新子类用于创建类似元组的对象，这些对象具有可通过属性查找访问的字段以及可索引和可迭代的字段。 子类的实例还有一个有用的文档字符串（带有`typename`和`field_names`）和一个有用的`__repr __()`方法，它以`name = value`格式列出元组内容。

- `field_names`是一系列字符串，例如`['x'，'y']`。 或者，`field_names`可以是单个字符串，每个字段名由`空格`和`/`或`逗号`分隔，例如`'x y'`或`'x，y'`。

- 除了以下划线开头的名称外，任何有效的Python标识符都可用于字段名。有效标识符由字母，数字和下划线组成，但不以数字或下划线开头，也不能是类，for，return，global，pass或raise等关键字。

- 如果`rename`为`true`，则无效的字段名称将自动替换为位置名称。 例如，`['abc'，'def'，'ghi'，'abc']`被转换为`['abc'，'_1'，'ghi'，'_3']`，消除了关键字`def`和重复的字段名`abc`。

- `defaults`可以是None或可迭代的默认值。 由于具有默认值的字段必须位于没有默认值的任何字段之后，因此默认值将应用于最右侧的参数。 例如，如果字段名是['x'，'y'，'z']并且默认值是（1,2），则x将是必需参数，y将默认为1，z将默认为2。

- 如果定义了`module`，则将命名元组的`__module__`属性设置为该值。

- 命名的元组实例没有每个实例的字典，因此它们是轻量级的，并且不需要比常规元组更多的内存。

举个例子：
```python
>>> # Basic example
>>> Point = namedtuple('Point', ['x', 'y'])
>>> p = Point(11, y=22)     # instantiate with positional or keyword arguments
>>> p[0] + p[1]             # indexable like the plain tuple (11, 22)
33
>>> x, y = p                # unpack like a regular tuple
>>> x, y
(11, 22)
>>> p.x + p.y               # fields also accessible by name
33
>>> p                       # readable __repr__ with a name=value style
Point(x=11, y=22)
```

命名元组对于将字段名称分配给csv或sqlite3模块返回的结果元组特别有用：
```python
EmployeeRecord = namedtuple('EmployeeRecord', 'name, age, title, department, paygrade')

import csv
for emp in map(EmployeeRecord._make, csv.reader(open("employees.csv", "rb"))):
    print(emp.name, emp.title)

import sqlite3
conn = sqlite3.connect('/companydata')
cursor = conn.cursor()
cursor.execute('SELECT name, age, title, department, paygrade FROM employees')
for emp in map(EmployeeRecord._make, cursor.fetchall()):
    print(emp.name, emp.title)
```

除了从元组继承的方法之外，命名元组还支持三个额外的方法和两个属性。 为防止与字段名称冲突，方法和属性名称以下划线开头。

- **classmethod somenamedtuple._make(iterable)**

	从现有序列或可迭代对象生成新的实例

		>>> t = [11, 22]
		>>> Point._make(t)
		Point(x=11, y=22)

- **somenamedtuple._asdict()**

	返回一个新的OrderedDict，它将字段名称映射到它们对应的值：

		>>> p = Point(x=11, y=22)
		>>> p._asdict()
		OrderedDict([('x', 11), ('y', 22)])

- **somenamedtuple._replace(**kwargs)**
	
	返回namedtuple的新实例，用新值替换特定字段：

		>>> p = Point(x=11, y=22)
		>>> p._replace(x=33)
		Point(x=33, y=22)
		>>>id(p._replace(x=66))
		4572244008
		>>>id(p)
		4569821544

- **somenamedtuple._fields**
	
	列出字段名称的字符串元组。 用于内省和从现有命名元组创建新的命名元组类型。

		>>>p._fields            # view the field names
		('x', 'y')
		>>> Color = namedtuple('Color', 'red green blue')
		>>> Pixel = namedtuple('Pixel', Point._fields + Color._fields)
		>>> Pixel(11, 22, 128, 255, 0)
		Pixel(x=11, y=22, red=128, green=255, blue=0)


- **somenamedtuple._fields_defaults**

	字典将字段名称映射到默认值。

		>>> Account = namedtuple('Account', ['type', 'balance'], defaults=[0])
		>>> Account._fields_defaults
		{'balance': 0}
		>>> Account('premium')
		Account(type='premium', balance=0)

要检索一个对象的字段，使用`getattr()`方法：
```python
>>> getattr(p, 'x')
11
```
要将字典转化为一个命名元组，使用`**x`的方式赋值:
```python
>>> d = {'x': 11, 'y': 22}
>>> Point(**d)
Point(x=11, y=22)
```

由于命名元组是常规Python类，因此很容易使用子类添加或更改功能。 以下是添加计算字段和固定宽度打印格式的方法：
```
>>> class Point(namedtuple('Point', ['x', 'y'])):
...     __slots__ = ()
...     @property
...     def hypot(self):
...         return (self.x ** 2 + self.y ** 2) ** 0.5
...     def __str__(self):
...         return 'Point: x=%6.3f  y=%6.3f  hypot=%6.3f' % (self.x, self.y, self.hypot)

>>> for p in Point(3, 4), Point(14, 5/7):
...     print(p)
Point: x= 3.000  y= 4.000  hypot= 5.000
Point: x=14.000  y= 0.714  hypot=14.018
```
上面显示的子类将__slots__设置为空元组。 这有助于防止创建实例字典，从而降低内存需求。

子类化对于添加新的存储字段没有用。 相反，只需从_fields属性创建一个新的命名元组类型：
```
>>> Point3D = namedtuple('Point3D', Point._fields + ('z',))
```

可以通过直接分配`__doc__`字段来自定义文档字符串：
```
>>> Book = namedtuple('Book', ['id', 'title', 'authors'])
>>> Book.__doc__ += ': Hardcover book in active collection'
>>> Book.id.__doc__ = '13-digit ISBN'
>>> Book.title.__doc__ = 'Title of first printing'
>>> Book.authors.__doc__ = 'List of authors sorted by last name'
```

通过使用`_replace()`对定制的已经有默认值的原型实例进行改造
```
>>> Account = namedtuple('Account', 'owner balance transaction_count')
>>> default_account = Account('<owner name>', 0.0, 0)
>>> johns_account = default_account._replace(owner='John')
>>> janes_account = default_account._replace(owner='Jane')
```

## **deque**
实现：
```python
class collections.deque([iterable[, maxlen]])
```
返回一个新的deque（双端队列）对象，它初始化自`iterable`。 如果未指定iterable，则新的deque为空。

Deques是堆栈和队列的泛化(名称发音为“deck”，是“双端队列”的缩写)。Deques支持从deque的任意一侧线程安全、内存高效的`appends`和`pop`，在任何方向上的性能都大致相同都是O(1)。

尽管`list`对象支持类似的操作，但它们针对快速固定长度操作进行了优化，并导致pop(0)和insert(0，v)操作有O(n)内存移动成本，这些操作改变了底层数据表示的大小和位置。

如果未指定`maxlen`或为None，则deques可能会增长到任意长度。 否则，双端队列限制为指定的最大长度。 一旦有界长度双端队列已满，当添加新项时，则会从对方端丢弃相应数量的项。 有界长度deques提供类似于Unix中的`tail`过滤器的功能。 它们还可用于跟踪仅涉及最近活动的事务和其他数据池。

Deque对象支持以下方法：
```python
append(x)	# 从deque的右边加入x

appendleft(x)	#从deque的左边加入x

clear()		#清除deque中的每一个元素，使其长度为0

copy()		# 创建一个deque的浅拷贝

count(x)	# deque中元素等于x的数量

extend(iterable)	# 从右侧扩展deque

extendleft(iterable)	# 从左侧扩展deque。但是，左边扩展的序列是反转iterable的顺序

index(x[, start[, stop]])	# 返回deque中的x位置（在索引开始时或索引停止之前）。返回第一个匹配的对象，如果没找到，会抛出`ValueError`

insert(i, x)	# 将x插入到deque的位置i。如果插入后导致deque超过`maxlen`，会抛出`IndexError`

pop()	# 从deque的右侧移除并返回一个元素。 如果没有元素，则会抛出IndexError。

popleft()	# 从deque的左侧移除并返回一个元素。 如果没有元素，则会抛出IndexError。

remove(value)	# 删除第一次出现的值。 如果未找到，则会抛出ValueError

reverse()		# 在原位反转deque的元素，然后返回None

rotate(n=1)		# 向右旋转deque n步。 如果n为负数，则向左旋转。
				# 当双端队列不为空时，向右旋转一步相当于d.appendleft(d.pop())，向左旋转一步相当于d.append(d.popleft())。
```

Deque对象还提供一个只读属性：
```python
maxlen	# deque的大小，如果无界，则为None
```

除上述之外，deques支持迭代，pickling, len(d), reverse(d), copy.copy(d), copy.deepcopy(d), 使用`in`运算符进行成员资格测试，以及下标引用，例如d[-1]。 索引访问在两端都是O(1), 但在中间减慢到O(n)。 对于快速随机访问，请改用list。

栗子：
```python
>>> from collections import deque
>>> d = deque('ghi')                 # make a new deque with three items
>>> for elem in d:                   # iterate over the deque's elements
...     print(elem.upper())
G
H
I

>>> d.append('j')                    # add a new entry to the right side
>>> d.appendleft('f')                # add a new entry to the left side
>>> d                                # show the representation of the deque
deque(['f', 'g', 'h', 'i', 'j'])

>>> d.pop()                          # return and remove the rightmost item
'j'
>>> d.popleft()                      # return and remove the leftmost item
'f'
>>> list(d)                          # list the contents of the deque
['g', 'h', 'i']
>>> d[0]                             # peek at leftmost item
'g'
>>> d[-1]                            # peek at rightmost item
'i'

>>> list(reversed(d))                # list the contents of a deque in reverse
['i', 'h', 'g']
>>> 'h' in d                         # search the deque
True
>>> d.extend('jkl')                  # add multiple elements at once
>>> d
deque(['g', 'h', 'i', 'j', 'k', 'l'])
>>> d.rotate(1)                      # right rotation
>>> d
deque(['l', 'g', 'h', 'i', 'j', 'k'])
>>> d.rotate(-1)                     # left rotation
>>> d
deque(['g', 'h', 'i', 'j', 'k', 'l'])

>>> deque(reversed(d))               # make a new deque in reverse order
deque(['l', 'k', 'j', 'i', 'h', 'g'])
>>> d.clear()                        # empty the deque
>>> d.pop()                          # cannot pop from an empty deque
Traceback (most recent call last):
    File "<pyshell#6>", line 1, in -toplevel-
        d.pop()
IndexError: pop from an empty deque

>>> d.extendleft('abc')              # extendleft() reverses the input order
>>> d
deque(['c', 'b', 'a'])
```

接下来，介绍一些deque的使用方法

有界长度deques提供类似于Unix中的`tail`过滤器的功能：
```python
def tail(filename, n=10):
    'Return the last n lines of a file'
    with open(filename) as f:
        return deque(f, n)
```

使用deques的另一种方法是通过向右追加并弹出到左侧来维护一系列最近添加的元素：
```python
from collections import deque
import itertools

def moving_average(iterable, n=3):
    # moving_average([40, 30, 50, 46, 39, 44]) --> 40.0 42.0 45.0 43.0
    # http://en.wikipedia.org/wiki/Moving_average
    it = iter(iterable)
    d = deque(itertools.islice(it, n-1))
    d.appendleft(0)
    s = sum(d)
    for elem in it:
        s += elem - d.popleft()
        d.append(elem)
        yield s / n

if __name__ == '__main__':
    for i in moving_average([40, 30, 50, 46, 39, 44]):
        print(i)

结果：
40.0
42.0
45.0
43.0

```

可以使用存储在双端队列中的输入迭代器来实现循环调度程序。 值从位置零处的活动迭代器产生。 如果该迭代器耗尽，可以使用popleft()删除它; 否则，它可以使用rotate()方法循环回到最后：
```python
def roundrobin(*iterables):
    "roundrobin('ABC', 'D', 'EF') --> A D E B F C"
    iterators = deque(map(iter, iterables))
    while iterators:
        try:
            while True:
                yield next(iterators[0])
                iterators.rotate(-1)
        except StopIteration:
            # Remove an exhausted iterator.
            iterators.popleft()
```

`rotate()`方法提供了一种实现双端切片和删除的方法。 例如，`del d[n]`的纯Python实现依赖于`rotate()`方法来定位要弹出的元素：
```python
def delete_nth(d, n):
    d.rotate(-n)
    d.popleft()
    d.rotate(n)
```
要实现双端切片，请使用类似的方法应用`rotate()`将目标元素置于双端队列的左侧。 使用`popleft()`删除旧条目，使用`extend()`添加新条目，然后反转旋转。 通过该方法的微小变化，可以轻松实现Forth样式堆栈操作，例如dup，drop，swap，over，pick，rot和roll。


## **ChainMap**

`ChainMap`类提供一个快速链接多个映射（字典）的操作。通常情况下，他会比创建字典然后调用`update()`快。

该类可用于模拟嵌套作用域，在模板中很有用。

实现：
```python
class collections.ChainMap(*maps)
```

`ChainMap`类组合多个字典或其他映射到一个可更新的、单一的对象中。如果没有指定`maps`，就会提供一个空字典，以此来保证每个新链中都会有至少一个字典（映射）

底层映射存储在列表中。 该列表是公共的，可以使用`maps`属性访问或更新。

```python
>>> from collections import ChainMap
>>> m1 = {'color': 'red', 'user': 'guest'}
>>> m2 = {'name': 'drfish', 'age': '18'}
>>> chain_map = ChainMap(m1, m2)
>>> chain_map
ChainMap({'color': 'red', 'user': 'guest'}, {'name': 'drfish', 'age': '18'})
>>> print(chain_map.get('name'))
drfish
```

支持所有常用的字典方法。除此之外，还支持以下属性：

- **maps**

	返回一个用户可以更新的映射列表。他是按照搜索顺序排序的。

		>>> chain_map.maps
		[{'color': 'red', 'user': 'guest'}, {'name': 'drfish', 'age': '18'}]
	

- **new_child(m=None)**
	
	返回一个新的ChainMap，这个新ChainMap包含新添加的map，并且这个map在首位。如果没有指定m，那么就会在最前面添加一个空dict。因此`d.new_child()`相当于`ChainMap({}, *d.maps)`。

	需要注意的是，这将产生一个全新的ChainMap，和之前的互不干扰

	读一下源码会更容易理解：

		def new_child(self, m=None):                # like Django's Context.push()
	        '''New ChainMap with a new map followed by all previous maps.
	        If no map is provided, an empty dict is used.
	        '''
	        if m is None:
	            m = {}
	        return self.__class__(m, *self.maps)


	示例：

		>>> m3 = {'data': '1-6'}
		>>> chain_map.new_child(m=m3)
		ChainMap({'data': '1-6'}, {'color': 'red', 'user': 'guest'}, {'name': 'drfish', 'age': '18'})
		>>> chain_map
		ChainMap({'color': 'red', 'user': 'guest'}, {'name': 'drfish', 'age': '18'})
		>>> id(chain_map.new_child(m=m3))
		4496700080
		>>> id(chain_map)
		4496631176


- **parents**

		@property
	    def parents(self):                          # like Django's Context.pop()
	        'New ChainMap from maps[1:].'
	        return self.__class__(*self.maps[1:])

	返回一个新的ChainMap，这个新ChainMap不包括第一个dict。这个对于跳过第一个map搜索很有用。`d.parents`大致相当于`ChainMap(*d.maps[1:])`

		>>> chain_map.parents
		ChainMap({'name': 'drfish', 'age': '18'})
		>>> id(chain_map.parents)
		4492113680
		>>> id(chain_map)
		4496631176


## **Counter**

实现：
```python
class collections.Counter([iterable-or-mapping])
```

源码中，简单介绍了一些用法：

```python
>>> c = Counter('abcdeabcdabcaba')  # count elements from a string

>>> c.most_common(3)                # three most common elements
[('a', 5), ('b', 4), ('c', 3)]
>>> sorted(c)                       # list all unique elements
['a', 'b', 'c', 'd', 'e']
>>> ''.join(sorted(c.elements()))   # list elements with repetitions
'aaaaabbbbcccdde'
>>> sum(c.values())                 # total of all counts
15

>>> c['a']                          # count of letter 'a'
5
>>> for elem in 'shazam':           # update counts from an iterable
...     c[elem] += 1                # by adding 1 to each element's count
>>> c['a']                          # now there are seven 'a'
7
>>> del c['b']                      # remove all 'b'
>>> c['b']                          # now there are zero 'b'
0

>>> d = Counter('simsalabim')       # make another counter
>>> c.update(d)                     # add in the second counter
>>> c['a']                          # now there are nine 'a'
9

>>> c.clear()                       # empty the counter
>>> c
Counter()

Note:  If a count is set to zero or reduced to zero, it will remain
in the counter until the entry is deleted or the counter is cleared:

>>> c = Counter('aaabbc')
>>> c['b'] -= 2                     # reduce the count of 'b' by two
>>> c.most_common()                 # 'b' is still in, but its count is zero
[('a', 3), ('c', 1), ('b', 0)]

```

`Counter`是dict的子类，可以用来计算可哈希对象的数量。它是一个无序的集合，并且元素作为dict的key，数量作为dict的value。数量可以是任意整数值，包括0和负数。

```python
>>> c = Counter()                           # a new, empty counter
>>> Counter("adfadf")						# a new counter from an iterables
Counter({'a': 2, 'd': 2, 'f': 2})			
>>> Counter({'red': 4, 'blue': 2})			# a new counter from a mapping
Counter({'red': 4, 'blue': 2})
>>> Counter(cats=4, dogs=8)					# a new counter from keyword args
Counter({'dogs': 8, 'cats': 4})
```

对于那些不存在的元素，如果想要获取它，Counter会返回0，而不会引发`KeyError`：

```python
>>> c = Counter(['eggs', 'ham'])
>>> c['bacon']                              # count of a missing element is zero
0
```
从源码中可以看出来为什么不引发`KeyError`:

```python
def __missing__(self, key):
    'The count of elements not in the Counter is zero.'
    # Needed so that self[missing_item] does not raise KeyError
    return 0
```

如果count设置为零或减少为零，它将保留在counter中，直到删除该条目或清除计数器：

```python
>>> c['sausage'] = 0                        # counter entry with a zero count
>>> del c['sausage']  
```

由于Counter是dict的子类，因此他具备dict的方法。除此之外，它还具备以下方法：

- **elements()**

	返回每一个元素，元素会根据个数重复count次，并且是以任意顺序返回的。如果元素的个数小于1(包括负值)，那么就会被忽略不返回

		>>> c = Counter(a=4, b=2, c=0, d=-2)
		>>> sorted(c.elements())
		['a', 'a', 'a', 'a', 'b', 'b']

- **most_common([n])**

	返回n个最常见元素及其计数的列表，从最常见到最少。 如果省略n或None，则most_common（）返回计数器中的所有元素。 具有相同计数的元素是任意排序的：

		>>> Counter('abracadabra').most_common(3)  
		[('a', 5), ('r', 2), ('b', 2)]

- **subtract([iterable-or-mapping])**

	根据迭代器或映射中对当前元素进行加减操作。和`dict.update()`类似，但是注意，是操作，而不是替换。输入和输出可以为0或负数

		>>> c = Counter(a=4, b=2, c=0, d=-2)
		>>> d = Counter(a=1, b=2, c=3, d=4)
		>>> c.subtract(d)
		>>> c
		Counter({'a': 3, 'b': 0, 'c': -3, 'd': -6}

通常情况下，Counter对象和字典具有相同的方法。但是以下两个方法会有所不同：

- **fromkeys(iterable)**

	Counter类没有实现这个方法


- **update([iterable-or-mapping])**

	和`dict.update()`相似，但是是进行加减操作，而不是替换。

	从源码看，更容易理解一些：

		    def update(*args, **kwds):
        '''Like dict.update() but add counts instead of replacing them.

        Source can be an iterable, a dictionary, or another Counter instance.

        >>> c = Counter('which')
        >>> c.update('witch')           # add elements from another iterable
        >>> d = Counter('watch')
        >>> c.update(d)                 # add elements from another counter
        >>> c['h']                      # four 'h' in which, witch, and watch
        4

        '''
        # The regular dict.update() operation makes no sense here because the
        # replace behavior results in the some of original untouched counts
        # being mixed-in with all of the other counts for a mismash that
        # doesn't have a straight-forward interpretation in most counting
        # contexts.  Instead, we implement straight-addition.  Both the inputs
        # and outputs are allowed to contain zero and negative counts.

        if not args:
            raise TypeError("descriptor 'update' of 'Counter' object "
                            "needs an argument")
        self, *args = args
        if len(args) > 1:
            raise TypeError('expected at most 1 arguments, got %d' % len(args))
        iterable = args[0] if args else None
        if iterable is not None:
            if isinstance(iterable, _collections_abc.Mapping):
                if self:
                    self_get = self.get
                    for elem, count in iterable.items():
                        self[elem] = count + self_get(elem, 0)
                else:
                    super(Counter, self).update(iterable) # fast path when counter is empty
            else:
                _count_elements(self, iterable)
        if kwds:
            self.update(kwds)

官网给出一些常见操作：
```python
sum(c.values())                 # total of all counts
c.clear()                       # reset all counts
list(c)                         # list unique elements
set(c)                          # convert to a set
dict(c)                         # convert to a regular dictionary
c.items()                       # convert to a list of (elem, cnt) pairs
Counter(dict(list_of_pairs))    # convert from a list of (elem, cnt) pairs
c.most_common()[:-n-1:-1]       # n least common elements
+c                              # remove zero and negative counts
```

提供了几个数学运算来组合Counter对象以生成多个集合（计数大于零的计数器）。 加法和减法通过添加或减去相应元素的计数来组合计数器。 &和 | 返回相应计数的最小值和最大值。 每个操作都可以接受带有符号计数的输入，但输出将排除计数为零或更少的结果。

```python
>>> c = Counter(a=3, b=1)
>>> d = Counter(a=1, b=2)
>>> c + d                       # add two counters together:  c[x] + d[x]
Counter({'a': 4, 'b': 3})
>>> c - d                       # subtract (keeping only positive counts)
Counter({'a': 2})
>>> c & d                       # intersection:  min(c[x], d[x]) 
Counter({'a': 1, 'b': 1})
>>> c | d                       # union:  max(c[x], d[x])
Counter({'a': 3, 'b': 2})
```

一元加法和减法是用于添加空计数器或从空计数器中减去的快捷方式。
```python
>>> c = Counter(a=2, b=-4)
>>> +c
Counter({'a': 2})
>>> -c
Counter({'b': 4})
```

<font color="red">注意</font> : Counter主要用于处理正整数的计数。但是也不要忘记考虑其他类型或负值的情况。Counter类继承自dict，对于key和value是没有限制的。value除了数字也可以存储其他。




















