# Numpy 使用手册



# 1. 基本概念
NumPy的主要对象是一个均衡的多维数组。也就是一张元素的表格，这些元素通常是数字，所有的元素都是相同类型的，通过下标构成的元组进行索引。在NumPy中的多维数组的维度被成为轴（axes）。轴的数量被称为秩（rank）

例如，在3D空间中的一个点的坐标[1, 2, 1]可以表示为一个秩为1的数组。因为它只有一个轴。这个轴的长度是3.在下面的例子中，这个数组的秩为2（这是一个二维数组）。第一维（轴）的长度为2，第二维的长度为3。

```Python
[
    [1, 0, 0],
    [0, 1, 2]
]
```

NumPy 的数组类被称为 `ndarray`，其别名为 `array`。注意，`numpy.array` 与标准 Python 类库中的 `array.array` 是不同的，标准类库中的数组只能处理一维数组，并且只提供了极少数的功能。下面列举了一些`ndarray` 对象的重要属性。

## 1.1. 维度（ndarray.ndim）

表示数组中轴（维度）的数量。在Python中，维度的数量也被称为秩（rank）

**一维数组**
```Python
import numpy as np

myArray = np.array([1, 0])
print(myArray.ndim) # print 1
```

**二维数组**
```Python
from numpy import *

myArray = array([[1, 0, 3], [2, 3, 5]])
print(myArray.ndim) # print 2
```

**三维数组**
```Python
from numpy import *

myArray = array(
    [
        [
            [1, 2], 
            [3, 1]
        ], [
            [1, 2], 
            [3, 1]
        ]
    ]
)
print(myArray.ndim)    # print 3
```

## 1.2. 维度的大小（ndarray.shape）
是一个表示数组维度大小的元组。元组中的整型数字表示数组中相应维度的大小。例如一个有 n 行 m 列的举证，shape 就是(n, m)。这个 shape 元组的长度也就是秩（rank）或者说是维度的数量。

**一维数组**
```Python
from numpy import *

myArray = array([1, 0])
print(myArray.shape)
```

这个打印的结果是 (2) 。这是个一维数组，一共有两个元素，因此shape是(2)。

**二维数组**
```Python
from numpy import *

myArray = array([[1, 0, 3], [2, 3, 5]])
print(myArray.shape)
```

这个打印的结果是(2, 3)。这是一个二维数组，因此这个shape元组中有两个元素。

第一个元素表示第一个维度中元素的数量，这个数组中第一个维度中有两个元素，分别是[1, 0, 3], [2, 3, 5]。

第二个元素表示第二个维度中元素的数量，也就是[1, 0, 3]和[2, 3, 5]中都有三个元素，因此是3。

**三维数组**
```Python
from numpy import *

myArray = array(
    [
        [
            [1, 2, 1], 
            [3, 1, 1]
        ], [
            [1, 2, 1],
            [3, 1, 1]
        ]
    ]
)
print(myArray.shape)
```

这个打印出来的结果是(2, 2, 3)，这个数组是一个三维数组，因此这个shape元组中有三个元素。

第一个维度中有两个元素，第二个维度中也是有两个元素，第三个维度则是最内部的有三个元素的数组。

## 1.3. 元素数量（ndarray.size）

数组中所有元素的数量。这个数字等同于 shape 的元组中所有元素的乘积。

**一维数组**
```Python
from numpy import *

myArray = array([1, 0])
print(myArray.size)
```

这个打印出来的结果是2。表示这个数组中一个有两个元素。

**二维数组**
```Python
from numpy import *

myArray = array([[1, 0, 3], [2, 3, 0]])
print(myArray.size)
```

这个数组中有六个元素，因此打印出来的结果是6。

## 1.4. 元素类型（ndarray.dtype）

一个用于描述数组中元素类型的对象。使用标准Python类型可以创建或指定dtype，除此之外，NumPy还提供了其自定义的类型：numpy.int32，numpy.int16和numpy.float64等等

**浮点型**
```Python
from numpy import *

myArray = array([3.2, 5.1])
print(myArray.dtype)
```

这个打印出来的内容是 float64。表示这里的元素是浮点数。

**整型**
```Python
from numpy import *

myArray = array([3, 5])
print(myArray.dtype)
```
这里打印出来的内容是 int32。表示是32位整型。

**布尔型**
```Python
from numpy import *

myArray = array([False, True])
print(myArray.dtype)
```

这里打印出来的内容是 bool。表示是布尔值。

**字符型**
```Python
from numpy import *

myArray = array(['a', 'cc'])
print(myArray.dtype)
```

这里打印出来的内容是U2。U表示的是数据类型，表示Unicode，后面的2表示数据的长度是2个字符。

# 2. 创建数组

## 2.1. 将Python数据结构转为NumPy数组

### 2.1.1. 将元组创建为数组
array()函数可以接受Python的元组为参数，并将这个元组创建为数组。
```Python
from numpy import *

myArray = array((3, 2, 4, 5))
print(myArray)
```

打印内容为：[3 2 4 5]。

### 2.1.2. 将列表创建为数组
array()函数可以接受Python中的列表作为参数，将其转为数组。
```Python
from numpy import *

myArray = array([[1, 0, 3], [2, 3, 0]])
print(myArray)
```

## 2.2. 使用占位符创建指定shape的数组

所谓使用占位符创建数组的意思就是创建一个数组，然后使用占位符来填充其中的每个元素。比如创建一个 shape 为 (2,) 的数组，则表示创建一个如下形式的数组：
```Python
[占位符, 占位符]
```
如果要创建一个shape为(2, 3)的数组，则结果为如下形式：
```Python
[
    [占位符, 占位符, 占位符],
    [占位符, 占位符, 占位符]
]
```

### 2.2.1. 使用0作为占位符创建数组

使用 `zeros()` 函数接受一个 shape 元组可以用来创建占位符为 0 的指定大小的数组。
```Python
from numpy import *

myArray = zeros((3, 4))
print(myArray)
```

打印结果为：

```Python
[[ 0.  0.  0.  0.]
 [ 0.  0.  0.  0.]
 [ 0.  0.  0.  0.]]
```

### 2.2.2. 使用1作为占位符创建数组
使用 `ones()` 函数接受一个 shape 元组可以用来创建占位符为1的指定大小的数组。
```Python
from numpy import *

myArray = ones((3, 4))
print(myArray)
```
打印结果：
```Python
[[ 1.  1.  1.  1.]
 [ 1.  1.  1.  1.]
 [ 1.  1.  1.  1.]]
```

### 2.2.3. 使用随机浮点数作为占位符创建数组
使用 `empty()` 函数接受一个shape元组可以用来创建占位符为随机数的指定大小的数组。
```Python
from numpy import *

myArray = empty((3, 4))
print(myArray)
```
打印结果：
```Python
[[  8.82769181e+025   7.36662981e+228   7.54894003e+252   2.95479883e+137]
 [  1.42800637e+248   2.64686750e+180   1.09936856e+248   6.99481925e+228]
 [  7.54894003e+252   7.67109635e+170   2.64686750e+180   3.50784751e-191]]
```

## 2.3. 创建数字序列数组
使用 `arange()` 创建数字序列数组，这个函数接受三个参数，前两个参数分别表示序列的首尾，第三个参数表示步长。
```Python
from numpy import *

myArray = arange(1, 8, 2)
print(myArray)
```

该段代码创建的数组为：
```Python
[1 3 5 7]
```

# 3. 基本运算

数组上的算术运算是作用于数组内的元素上的。

## 3.1. 减法运算

```Python
from numpy import *

array1 = array([10, 20, 30, 40])
array2 = array([1, 2, 3, 4])
array3 = array1 - array2
print(array3)
```

打印结果：[9, 18, 27, 36]。观察这个运算后的结果可以发现上面两个数组相减实际上是对应位置上的元素进行了减法运算。

## 3.2. 平方运算

同样的道理，对数组进行平方运算也同样是作用在数组的元素上。

```Python
from numpy import *

array1 = array([1, 2, 3, 4])
array2 = array1 ** 2
print(array2)

```

结果为：[1, 4, 9, 16]

## 3.3. 矩阵乘法运算

首先复习一下矩阵的乘法运算，有如下两个A、B矩阵相乘：

```Python
a b    e f
c d    g h

ae+bg af+bh
ce+dg cf+dh
```

仔细观察其运算规则实际上新的矩阵的第一行的第一个元素是原矩阵A的第一行分别与矩阵B的第一列元素相乘之后的和。而第一行第二个元素是矩阵A的第一行与矩阵B的第二列乘积的和。

在 NumPy 中矩阵相乘需要使用 `dot()` 方法，而不是直接使用 `“*”`。

```Python
from numpy import *

array1 = ones((2, 2))
array2 = ones((2, 2))
array3 = array1.dot(array2)
print(array3)
```

结果为：
```Python
[[ 2.  2.]
 [ 2.  2.]]
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjk5NjM1OTcwXX0=
-->