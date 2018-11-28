# 摘要

# 项目介绍

## 项目描述
在1912年4月15日发生的泰坦尼克沉船事件中，2224名乘客和船员中，有1502人丧生。在这次海难中导致如此多人员丧生的一个重要原因是没有足够的救生船。不过其中一些乘客的存活率要比其他乘客的存活率大，比如妇女/儿童和上层乘客。

在这个挑战中，我们期望你分析一下什么样的座次有利于提高存活率。并且，我们要求你应用机器学习工具来预测哪些用户在此次海难中存活下来。

## 评价

### 目标
预测在海难中存活的乘客。对给定测试集中的乘客ID，预测一个 0 或 1 的值表示其存活情况。

### 度量标准
准确预测乘客存活情况的百分比。我们使用“准确率”来表示。

### 提交文件格式
提交一份包含 418 条数据和一个标题行的 CSV 文件。如果提交的文件中有多于的行或列，回报错。

文件应该正好只有两行：
1、PassengerId：乘客ID，任意顺序
2、Survived：存活情况，1表示存活，0表示丧生

```
PassengerId,Survived
892,0
893,1
894,0
Etc.
```

## 数据

数据被分为两组：
1、训练集（train.csv）
2、测试集（test.csv）

**训练集**
用来构建你的机器学习模型。在这个训练集中，我们为每个乘客ID提供了一个对应的结果，用于表示这位乘客是否生还。你的模型需要基于乘客的特征，如乘客的性别、分类等。你也可以利用特征工程来创建新的特征。

**测试集**
测试集用来检查你的模型对于未知的数据能够有多好的效果。在测试集中，我们并没有提供乘客对应的生还情况。而是由你来预测乘客是否生还。利用你通过训练集训练出来的模型，对测试集中每位乘客的生还情况进行预测。

**数据字典**

|变量|定义|值|
|:-:|:-:|:-:|
|survival|是否生还|0=No,1=Yes|
|pclass|坐席类型|1=一等座, 2=二等座, 3=三等座|
|sex|性别|
|Age|年龄|
|sibsp|在船上的兄弟姐妹/配偶数量|
|parch|在船上的父母/子女数量|
|ticket|船票编号|
|fare|票价|
|cabin|船舱编号|
|embarked|登船港口|C=Cherbourg, Q=Queenstown, S=Southampton

**变量备注**
pclass：表示社会经济地位
- 1：一等座上层
- 2：二等座中层
- 3：三层座底层

age：如果小于一岁，年龄是分数。如果年龄是估计的，其形式为 XX.5

sibsp/parch：数据集中家庭关系的定义

一些儿童仅仅由保姆陪同出行，因此他们的 parch = 0。


# 解决过程

## 整体框架

**1、定义问题**
在真正决定使用什么样的技术、算法之前，要明确我们要解决的问题是什么。而不是一股脑的就套用最新的技术、工具或算法。

**2、收集数据**
John Naisbitt 在他 1984 的书中写到“我们淹没在数据中，却在寻找知识”。所以，现在我们面临的是数据集已经以某种形式存在于某处。可能是开放的或者需要挖掘的，结构化或非结构化的、静态的或流式数据等等。你只需要知道如何去找到它们，然后将这些“脏数据”转化为“干净数据”。

**3、准备数据**
这一阶段经常被称为数据整理（data wrangling），其中一个必须的流程就是将“杂乱的”数据转换为“易于管理的”数据。数据整理包括以下几部分内容：
- 实现易于存储和处理的数据架构（data architecture）
- 开发质量与控制的数据治理（data governance）标准
- 数据抽取（data extraction）
- 以识别异常数据、缺失值以及离群值为目标的数据清理（data cleaning）

**4、探索性分析**
如果有过数据工作经验的人可能会知道垃圾输入（garbage-in）和垃圾输出（garbage-out）。因此应该使用描述性统计分析和图示统计分析在数据集中寻找潜在的问题、模式、分类、相关系数和对照等内容。另外，数据分类对于理解和选择合适的假设检验或数据模型是非常重要的。

**5、模型数据**
类似于描述性和推理性统计，数据模型可以总结并预测特征结果。通过数据集特征值以及预期结果分析，可以决定你的算法是否能够使用。需要注意的是算法并不是魔法棒或者银弹，你必须真正懂得如何去选择合适的工具解决对应的问题。

**6、验证和实现模型**
在基于一部分数据训练完成模型之后，需要检验模型。以确保没有对该部分子集数据过拟合。在这个阶段，我们确定模型是过拟合、泛化还是欠拟合。

**7、优化**
接下来就是不断的优化，让你的模型变得更好、更快、更强。

## STEP 1. 定义问题
在这个项目中，问题的定义非常明确，并且已经在项目介绍中明确的给出了，开发一个算法用于预测泰坦尼克事故中乘客的生还情况。

>在1912年4月15日发生的泰坦尼克沉船事件中，2224名乘客和船员中，有1502人丧生。在这次海难中导致如此多人员丧生的一个重要原因是没有足够的救生船。不过其中一些乘客的存活率要比其他乘客的存活率大，比如妇女/儿童和上层乘客。
>
>在这个挑战中，我们期望你分析一下什么样的座次有利于提高存活率。并且，我们要求你应用机器学习工具来预测哪些用户在此次海难中存活下来。

## STEP 2. 收集数据
在这个项目中，数据已经给定，直接下载即可。

## STEP 3. 准备数据
TODO 一些描述

### STEP 3.1. 准备环境（需要调整，不要一下子导入全部类库）
本项目代码基于 Python 3.x 编写。在正式编写代码之前，首先需要导入一些必要的依赖类库。
```Python

#load packages
import sys #access to system parameters https://docs.python.org/3/library/sys.html
print("Python version: {}". format(sys.version))

import pandas as pd #collection of functions for data processing and analysis modeled after R dataframes with SQL like features
print("pandas version: {}". format(pd.__version__))

import matplotlib #collection of functions for scientific and publication-ready visualization
print("matplotlib version: {}". format(matplotlib.__version__))

import numpy as np #foundational package for scientific computing
print("NumPy version: {}". format(np.__version__))

import scipy as sp #collection of functions for scientific computing and advance mathematics
print("SciPy version: {}". format(sp.__version__)) 

import sklearn #collection of machine learning algorithms
print("scikit-learn version: {}". format(sklearn.__version__))

#misc libraries
import random
import time
```

pip install scikit-learn

### STEP 3.2. 基本分析
接下来要正式开始接触数据了。首先我们需要对数据集有一些基本的了解。比如说数据集看起来是什么样子的，稍微描述一下。其中有哪些特征？ 每个特征大概起到什么样的作用？特征之间的依赖关系？
- Survived，是输出变量。1表示生还，0表示丧生。这个变量的所有值都需要我们进行预测。而除了这个变量以外的所有变量都可能是潜在的能够影响预测值的变量。**需要注意的是，并不是说用于预测的变量越多，训练的模型就越好，而是正确的变量才对模型有益。**
- PassengerId 和 Ticket，这两个变量是随机生成的唯一值，因此这两个值对结果变量不会有影响，因此在后续分析时，可以排除这两个变量。
- Pclass，是一个有序序列，用于表示社会经济地位，1表示上层人士；2表示中层；3表示底层。
- 

我们导入数据，然后使用 `sample()` 方法来快速的观察一下数据。

```Python
data_raw = pd.read_csv('./input/train.csv')

print(data_raw.head()) # 获取数据前五条
print(data_raw.tail()) # 获取数据后五条
print(data_raw.sample(10)) # 随机获取十条
```

打印结果：
|df_index|PassengerId|Survived|Pclass|Name|Sex|Age|SibSp|Parch|Ticket|Fare|Cabin|Embarked|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|660|661|1|1|Frauenthal, Dr. Henry William|male|50.0|2|0|PC 17611|133.6500|NaN|S|
|382|383|0|3|Tikkanen, Mr. Juho|male|32|0|0|STON/O 2. 3101293|7.925||S
|64|65|0|1|Stewart, Mr. Albert A|male||0|0|PC 17605|27.7208||C
|697|698|1|3|Mullens, Miss. Katherine "Katie"|female||0|0|35852|7.7333||Q
|345|346|1|2|Brown, Miss. Amelia "Mildred"|female|24|0|0|248733|13|F33|S
|525|526|0|3|Farrell, Mr. James|male|40.5|0|0|367232|7.75||Q
|883|884|0|2|Banfield, Mr. Frederick James|male|28|0|0|C.A./SOTON 34068|10.5||S
|487|488|0|1|Kent, Mr. Edward Austin|male|58|0|0|11771|29.7|B37|C
|553|554|1|3|Leeni, Mr. Fahim (Philip Zenni)|male|22|0|0|2620|7.225||C
|241|242|1|3|Murphy, Miss. Katherine "Kate"|female||1|0|367230|15.5||Q

# 参考
[1] https://www.kaggle.com/c/titanic
[2] https://www.kaggle.com/ldfreeman3/a-data-science-framework-to-achieve-99-accuracy/notebook



# TEMP

我们使用流行的机器学习算法类库 scikit-learn。
```Python
#Common Model Algorithms
from sklearn import svm, tree, linear_model, neighbors, naive_bayes, ensemble, discriminant_analysis, gaussian_process
from xgboost import XGBClassifier

#Common Model Helpers
from sklearn.preprocessing import OneHotEncoder, LabelEncoder
from sklearn import feature_selection
from sklearn import model_selection
from sklearn import metrics
```

可视化内容
```Python
#Visualization
import matplotlib as mpl
import matplotlib.pyplot as plt
import matplotlib.pylab as pylab
import seaborn as sns
from pandas.tools.plotting import scatter_matrix

#Configure Visualization Defaults
#%matplotlib inline = show plots in Jupyter Notebook browser
%matplotlib inline
mpl.style.use('ggplot')
sns.set_style('white')
pylab.rcParams['figure.figsize'] = 12,8
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjk0MDAyNzUsLTg3NzE3MTM2Ml19
-->