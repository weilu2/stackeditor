Pandas tutorial

# 摘要

Pandas 是 Python 语言下的一个用于数据分析的工具类库。使用 Pandas 可以方便的对数据进行处理和分析。

# 1. Data Structures
Pandas 处理数据靠的是两个核心数据结构，Series 和 DataFrame，将会贯穿于整个数据分析过程。

Series 用来处理一维的序列数据，而 DataFrame 用来处理更复杂的多维数据。

## 1.1. Series

Series 是 Pandas 中用来处理一维数据的结构，有点类似于数组，但是增加了许多额外的特性。其数据结构如下所示：

|index|value|
|:-:|:-:|
|0|12|
|1|-4|
|2|7|
|3|9|

Series 包含两个数组，一个是存储的实际的值，另一个存储的是值的索引。Series 中存储的值可以是所有的 NumPy 中的数据结构。

### 1.1.1. 创建 Series

#### 1.1.1.1. 从 list 中创建

```Python
import numpy as np
import pandas as pd

s = pd.Series([12, -4, 7, 9])
print(s)
```

打印结果：

```Python
0    12
1    -4
2     7
3     9
dtype: int64
```

#### 1.1.1.2. 创建指定索引的 Series

```Python
import numpy as np
import pandas as pd

s = pd.Series([12, -4, 7, 9], index = ['a', 'b', 'c', 'd'])
print(s)
```

打印结果：

```Python
a    12
b    -4
c     7
d     9
dtype: int64
```

#### 1.1.1.3. 从 Numpy 数组创建

```Python
arr = np.array([1, 2, 3, 4])
ser = pd.Series(arr)
print(ser)
```

打印结果：

```Python
0    1
1    2
2    3
3    4
dtype: int32
```

**需要注意的是，从 Numpy 中创建的 Series，只是引用，对 Series 中值操作的影响会直接反应到原始的 Numpy 中**

```Python
print(arr)
ser[0] = 0
print(arr)
```

打印结果：

```Python
[1 2 3 4]
[0 2 3 4]
```

#### 1.1.1.4. 从 dict 创建

```Python
dic = {'red': 2000, 'blue': 1000, 'yellow': 500, 'orange': 1000}
ser = pd.Series(dic)
print(ser)
```

打印结果：

```Python
red       2000
blue      1000
yellow     500
orange    1000
dtype: int64
```


### 1.1.2. 查看 Series

#### 1.1.2.1. 访问元素&查看Series

使用指定标签创建的 Series 既可以用下标访问元素，也可以用标签访问元素：

```Python
print("s[1]: " + str(s[1]))
print("s['b']: " + str(s['b']))
```

打印结果：

```Python
s[1]: -4
s['b']: -4
```

另外还可以直接查看 Series 的索引和值：

```Python
print("s.values: " + str(s.values))
print("s.index: " + str(s.index))
```

打印结果：

```Python
s.values: [12 -4  7  9]
s.index: Index(['a', 'b', 'c', 'd'], dtype='object')
```

### 1.1.2.2. 选取值

从 Series 中选取值与 NumPy 中类似，可以直接使用切片的方式选取。

```Python
print(s[0:2])
```

打印结果：

```Python
a    12
b    -4
dtype: int64
```

另外 Series 还支持使用标签的形式来选取对应的值：

```Python
print(s[['b', 'd']])
```
注意，这里的标签是一个数组，打印结果：

```Python
b   -4
d    9
dtype: int64
```

使用表达式选择值：

```Python
print(s[s > 8])
```
打印结果：

```Python
a    12
d     9
dtype: int64
```

### 1.1.2.3. 赋值

可以直接使用标签或索引，类似于数组进行赋值。

```Python
s['b'] = 1
print(s)
```

打印结果：

```Python
a    12
b     1
c     7
d     9
dtype: int64
```

### 1.1.3. 数学运算

类似于 Numpy 中对数学运算的支持，可以使用 Series 直接与数值进行加减乘除。

### 1.1.4. 常用操作

#### 1.1.4.1. 去重

```Python
ser = pd.Series([1, 0, 2, 1, 2, 3])
print(ser.unique())
```

打印结果：

```Python
[1 0 2 3]
```

#### 1.1.4.2. 统计

```Python
ser = pd.Series([1, 0, 2, 1, 2, 3])
print(ser.value_counts())
```

打印结果：

```Python
2    2
1    2
3    1
0    1
dtype: int64
```

其中第一列表示的是 Series 中的值，第二列表示的是在 Series 中出现的次数。

#### 1.1.4.3. 是否存在

```Python
ser = pd.Series([1, 0, 2, 1, 2, 3])
print(ser.isin([0, 3]))
```

打印结果：

```Python
0    False
1     True
2    False
3    False
4    False
5     True
dtype: bool
```

直接将结果返回回来：

```Python
print(ser[ser.isin([0, 3])])
```

打印结果：

```Python
1    0
5    3
dtype: int64
```

#### 1.1.4.4. 空值

在 Pandas 中使用 Numpy 中的 NaN 表示空值。可以使用 `isnull()` 和 `notnull()` 方法筛选结果。

```Python
ser = pd.Series([5, -3, np.NaN, 15])
print(ser)

print(ser.isnull())
```

打印结果：

```Python
0     5.0
1    -3.0
2     NaN
3    15.0
dtype: float64

0    False
1    False
2     True
3    False
dtype: bool
```

## 1.2. DataFrame

DataFrame 是一种类似于表格的结构，用于处理多维数据。

|index|color|object|price|
|:-:|:-:|:-:|:-:|
|0|blue|ball|1.2|
|1|green|pen|1.0|
|2|yellow|pencil|0.5|
|3|red|paper|0.8|
|4|white|mug|1.5|

不同于 Series，DataFrame 有两列索引，第一个索引是行索引，每个索引关联一行的数据；第二个索引包含的是一系列的标签，关联的是每个特定的列。

我们一般把行索引称为索引（index），把列索引称为标签（label）。

### 1.2.1. 创建 DataFrame

#### 1.2.1.1. 从字典中创建

```Python
import numpy as np
import pandas as pd

myDict = {
	'color': ['blue', 'green', 'yellow', 'red', 'white'],
	'object': ['ball', 'pen', 'pencil', 'paper', 'mug'],
	'price': [1.2, 1.0, 0.5, 0.8, 1.5]
}

df = pd.DataFrame(myDict)
print(df)
```

打印结果：

```Python
    color  object  price
0    blue    ball    1.2
1   green     pen    1.0
2  yellow  pencil    0.5
3     red   paper    0.8
4   white     mug    1.5
```

在创建时还可以指定需要的列，以及指定索引。

```Python
df = pd.DataFrame(myDict, columns=['object','price'],
	index=['one','two','three','four','five'])
print(df)
```

打印结果：

```Python
       object  price
one      ball    1.2
two       pen    1.0
three  pencil    0.5
four    paper    0.8
five      mug    1.5
```

#### 1.2.1.2. 从 Numpy 数组中创建

```Python
arr = np.arange(16).reshape((4, 4))
df = pd.DataFrame(arr, columns=['colA','colB', 'colC', 'colD'])
print(df)
```

打印结果：

```Python
   colA  colB  colC  colD
0     0     1     2     3
1     4     5     6     7
2     8     9    10    11
3    12    13    14    15
```

### 1.2.2. 查看 DataFrame

#### 1.2.2.1. 基本信息

查看索引：

```Python
print(df.index)
```

打印结果：

```Python
RangeIndex(start=0, stop=4, step=1)
```

查看标签：

```Python
print(df.columns)
```

打印结果：

```Python
Index(['colA', 'colB', 'colC', 'colD'], dtype='object')
```

查看值：
```Python
df.values
```

#### 1.2.2.2. 选择列

使用标签选择对应列的值：

```Python
myDict = {
	'color': ['blue', 'green', 'yellow', 'red', 'white'],
	'object': ['ball', 'pen', 'pencil', 'paper', 'mug'],
	'price': [1.2, 1.0, 0.5, 0.8, 1.5]
}

df = pd.DataFrame(myDict)
print(df[["object", "price"]])
```

打印结果：

```Python
   object  price
0    ball    1.2
1     pen    1.0
2  pencil    0.5
3   paper    0.8
4     mug    1.5
```

#### 1.2.2.3. 选择行

使用索引选择对应行的记录。其中 loc[] 中接受的可以是一个索引值，也可以是一个索引数组。

```Python
myDict = {
	'color': ['blue', 'green', 'yellow', 'red', 'white'],
	'object': ['ball', 'pen', 'pencil', 'paper', 'mug'],
	'price': [1.2, 1.0, 0.5, 0.8, 1.5]
}

df = pd.DataFrame(myDict)
print(df.loc[2])
```

打印结果：

```Python
color     yellow
object    pencil
price        0.5
Name: 2, dtype: object
```

#### 1.2.2.4. 切片

DataFrame 也支持切片，不过只支持行切片。

```Python
myDict = {
	'color': ['blue', 'green', 'yellow', 'red', 'white'],
	'object': ['ball', 'pen', 'pencil', 'paper', 'mug'],
	'price': [1.2, 1.0, 0.5, 0.8, 1.5]
}

df = pd.DataFrame(myDict)
print(df[1:3])
```

打印结果：

```Python
    color  object  price
1   green     pen    1.0
2  yellow  pencil    0.5
```

#### 1.2.2.5. 过滤

```Python
myDict = {
	'color': ['blue', 'green', 'yellow', 'red', 'white'],
	'object': ['ball', 'pen', 'pencil', 'paper', 'mug'],
	'price': [1.2, 1.0, 0.5, 0.8, 1.5]
}

df = pd.DataFrame(myDict)
print(df[df['price'] > 1])
```

打印结果：

```Python
   color object  price
0   blue   ball    1.2
4  white    mug    1.5
```

### 1.2.3. 修改 DataFrame

#### 1.2.3.1. 增加新列

```Python
myDict = {
	'color': ['blue', 'green', 'yellow', 'red', 'white'],
	'object': ['ball', 'pen', 'pencil', 'paper', 'mug'],
	'price': [1.2, 1.0, 0.5, 0.8, 1.5]
}

df = pd.DataFrame(myDict)
df['new'] = 12
print(df)
```

打印结果：

```Python
    color  object  price  new
0    blue    ball    1.2   12
1   green     pen    1.0   12
2  yellow  pencil    0.5   12
3     red   paper    0.8   12
4   white     mug    1.5   12
```

也可以使用一个数组作为新的列：

```Python
df['new'] = [3, 4, 5, 6, 7]
print(df)
```

打印结果：

```Python
    color  object  price  new
0    blue    ball    1.2    3
1   green     pen    1.0    4
2  yellow  pencil    0.5    5
3     red   paper    0.8    6
4   white     mug    1.5    7
```

这里的数组还可以换成 Series。

#### 1.2.3.2. 删除新列

```Python
del df['price']
print(df)
```

打印结果：

```Python
    color  object  new
0    blue    ball    3
1   green     pen    4
2  yellow  pencil    5
3     red   paper    6
4   white     mug    7
```

## Index Object

Index Object 是 Pandas 中表示索引的对象，在前面的 Series 和 DataFrame 中实际上已经接触过这个对象了。

一般不会涉及到这个对象的操作，知道有这么个东西即可。


# 2. 数据操作


## Lambda 表达式

首先创建一个三行四列的 DataFrame：

```Python
arr = np.arange(12).reshape((3, 4)) ** 2
df = pd.DataFrame(arr)
print(df)
```

打印结果：

```Python
    0   1    2    3
0   0   1    4    9
1  16  25   36   49
2  64  81  100  121
```

然后利用 lambda 表达式对其中的每个元素开平方：

```Python
df = df.apply(lambda x: np.sqrt(x))
print(df)
```

打印结果：

```Python
     0    1     2     3
0  0.0  1.0   2.0   3.0
1  4.0  5.0   6.0   7.0
2  8.0  9.0  10.0  11.0
```

![enter image description here](AA)




```Python

```

打印结果：

```Python

```


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwMTM5ODYyODksMTY4MjMyMTAzLDE2OT
IwNjk0M119
-->