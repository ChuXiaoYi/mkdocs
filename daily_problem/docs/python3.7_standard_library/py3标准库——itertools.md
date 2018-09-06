Python的内建模块itertools提供了非常有用的用于操作迭代对象的函数。

### **itertools.count(start=0, step=1)**
创建一个迭代器，生成从n开始的连续整数，如果忽略n，则从0开始计算（注意：此迭代器不支持长整数)

如果超出了sys.maxint，计数器将溢出并继续从-sys.maxint-1开始计算。

当使用浮点数进行计数时，有时可以通过替换乘法代码来实现更高的准确率，例如：`(start + step * i for i in count())`

该方法等价于：
```python
def count(start=0, step=1):
    # count(10) --> 10 11 12 13 14 ...
    # count(2.5, 0.5) -> 2.5 3.0 3.5 ...
    n = start
    while True:
        yield n
        n += step
```

### **itertools.cycle(iterable)**
创造一个迭代器，复制从当前迭代器返回的每一个元素并将其保存到创建的迭代器中。当当前迭代器耗尽时，从创造的迭代器循环返回元素。

简单理解就是，传入一个序列，无限循环下去

大致相当于:
```python
def cycle(iterable):
    # cycle('ABCD') --> A B C D A B C D A B C D ...
    saved = []
    for element in iterable:
        yield element
        saved.append(element)
    while saved:
        for element in saved:
              yield element
```

### **itertools.repeat(object[, times])**

让迭代器一次又一次地返回对象。无限运行，除非指定了`times`参数控制重复次数

大致相当于：
```python
def repeat(object, times=None):
    # repeat(10, 3) --> 10 10 10
    if times is None:
        while True:
            yield object
    else:
        for i in range(times):
            yield object
```

repeat的一个常见用法是提供一个用于map或zip的常数流:
```
>>> list(map(pow, range(10), repeat(2)))
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

### **itertools.accumulate(iterable[, func])**

创建一个迭代器，它返回计算的累积和，或其他二进制函数的计算结果（这个二进制函数可以通过func参数指定）。如果指定了func参数，必须保证这个参数对应的函数可以接收两个参数。

大致相当于：
```python
def accumulate(iterable, func=operator.add):
    'Return running totals'
    # accumulate([1,2,3,4,5]) --> 1 3 6 10 15
    # accumulate([1,2,3,4,5], operator.mul) --> 1 2 6 24 120
    it = iter(iterable)
    try:
        total = next(it)
    except StopIteration:
        return
    yield total
    for element in it:
        total = func(total, element)
        yield total
```

如果很难理解，可以这样理解：

    对于默认func来说，结果就是[p0, p0+p1, p0+p1+p2, …]

### **itertools.chain(\*iterables)**

创建一个迭代器，该迭代器从第一个迭代返回元素，直到它被耗尽，然后继续到下一个迭代，直到所有的迭代都被耗尽。用于将连续序列视为单个序列。
大致相当于:
```python
def chain(*iterables):
    # chain('ABC', 'DEF') --> A B C D E F
    for it in iterables:
        for element in it:
            yield element
```

### **classmethod chain.from_iterable(iterable)**

改变`chain()`的结构，从一个懒加载的可迭代对象中获取输入值。大致相当于：
```python
def from_iterable(iterables):
    # chain.from_iterable(['ABC', 'DEF']) --> A B C D E F
    for it in iterables:
        for element in it:
            yield element

```
### **itertools.compress(data, selectors)**

过滤迭代器中的元素，只返回在selectors中计算为`True`的对应元素。当迭代器或选择器结束后，就停止。
大致相当于：
```python
def compress(data, selectors):
    # compress('ABCDEF', [1,0,1,0,1,1]) --> A C E F
    return (d for d, s in zip(data, selectors) if s)
```

### **itertools.dropwhile(predicate, iterable)**

去除predicate为true的元素，当第一次遇到predicate为false的情况时，直接将其与后面的全都一起返回
大致相当于：
```python

def dropwhile(predicate, iterable):
    # dropwhile(lambda x: x<5, [1,4,6,4,1]) --> 6 4 1
    iterable = iter(iterable)
    for x in iterable:
        if not predicate(x):
            yield x
            break
    for x in iterable:
        yield x
```

### **itertools.filterfalse(predicate, iterable)**

从迭代器中过滤出所有使predicate为`False`的元素，并返回
大致相当于：
```python
def filterfalse(predicate, iterable):
    # filterfalse(lambda x: x%2, range(10)) --> 0 2 4 6 8
    if predicate is None:
        predicate = bool
    for x in iterable:
        if not predicate(x):
            yield x
```

### **itertools.groupby(iterable, key=None)**
创建一个从迭代中返回连续键和组的迭代器。 关键是计算每个元素的键值的函数。 如果未指定或为None，则键默认为标识函数并返回元素不变。 通常，迭代需要在相同的键函数上排序。

groupby（）的操作类似于Unix中的uniq过滤器。 每次键函数的值发生变化时，它都会生成一个中断或新组（这就是为什么通常需要使用相同的键函数对数据进行排序）。 这种行为不同于SQL的GROUP BY，它聚合了常见元素而不管它们的输入顺序如何。

返回的组本身是一个迭代器，它与groupby（）共享底层的iterable。 由于源是共享的，因此当groupby（）对象被迭代时，前一个组被迭代的将不再可见。 因此，如果以后需要该数据，则应将其存储为列表：
```python
groups = []
uniquekeys = []
data = sorted(data, key=keyfunc)
for k, g in groupby(data, keyfunc):
    groups.append(list(g))      # Store group iterator as a list
    uniquekeys.append(k)
```
`groupby()`相当于：
```python
class groupby:
    # [k for k, g in groupby('AAAABBBCCDAABBB')] --> A B C D A B
    # [list(g) for k, g in groupby('AAAABBBCCD')] --> AAAA BBB CC D
    def __init__(self, iterable, key=None):
        if key is None:
            key = lambda x: x
        self.keyfunc = key
        self.it = iter(iterable)
        self.tgtkey = self.currkey = self.currvalue = object()
    def __iter__(self):
        return self
    def __next__(self):
        self.id = object()
        while self.currkey == self.tgtkey:
            self.currvalue = next(self.it)    # Exit on StopIteration
            self.currkey = self.keyfunc(self.currvalue)
        self.tgtkey = self.currkey
        return (self.currkey, self._grouper(self.tgtkey, self.id))
    def _grouper(self, tgtkey, id):
        while self.id is id and self.currkey == tgtkey:
            yield self.currvalue
            try:
                self.currvalue = next(self.it)
            except StopIteration:
                return
            self.currkey = self.keyfunc(self.currvalue)
```
举个例子：
```python
from itertools import groupby
qs = [{'date' : 1},{'date' : 2}]
[(name, list(group)) for name, group in itertools.groupby(qs, lambda p:p['date'])]

Out[77]: [(1, [{'date': 1}]), (2, [{'date': 2}])]


>>> from itertools import *
>>> a = ['aa', 'ab', 'abc', 'bcd', 'abcde']
>>> for i, k in groupby(a, len):
...     print i, list(k)
...
2 ['aa', 'ab']
3 ['abc', 'bcd']
5 ['abcde']
```

### **itertools.islice(iterable, start, stop[, step])**和**itertools.islice(iterable, stop)**
创建一个迭代器，该迭代器从iterable返回选中的元素。如果start是非零的，则跳过可迭代的元素，直到到达start为止。之后，元素会连续返回，除非step设置得比step高，这会导致跳过项。如果stop为None，则继续迭代，直到迭代器耗尽为止;否则，它将在指定位置停止。与常规切片不同，islice（）不支持start，stop或step的负值。可以用于从内部结构已被扁平化的数据中提取相关字段(例如，多行报告可能每隔一行列出一个name字段)。
大致相当于:
```python
def islice(iterable, *args):
    # islice('ABCDEFG', 2) --> A B
    # islice('ABCDEFG', 2, 4) --> C D
    # islice('ABCDEFG', 2, None) --> C D E F G
    # islice('ABCDEFG', 0, None, 2) --> A C E G
    s = slice(*args)
    start, stop, step = s.start or 0, s.stop or sys.maxsize, s.step or 1
    it = iter(range(start, stop, step))
    try:
        nexti = next(it)
    except StopIteration:
        # Consume *iterable* up to the *start* position.
        for i, element in zip(range(start), iterable):
            pass
        return
    try:
        for i, element in enumerate(iterable):
            if i == nexti:
                yield element
                nexti = next(it)
    except StopIteration:
        # Consume to *stop*.
        for i, element in zip(range(i + 1, stop), iterable):
            pass
```
如果start为None，那么迭代从0开始。如果step是None，那么step默认为1。

### **itertools.starmap(function, iterable)**











