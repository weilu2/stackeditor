||Geographic Area|City|poverty_rate|
|:-:|:-:|:-:|:-:|
|0|AL|Abanda CDP|78.8|
|1|AL|Abbeville city|29.1|
|2|AL|Adamsville city|25.5|
|3|AL|Addison town|30.7|
|4|AL|Akron town|42|

```Python
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 2603 entries, 0 to 2602
Data columns (total 14 columns):
world_rank                2603 non-null object
university_name           2603 non-null object
country                   2603 non-null object
teaching                  2603 non-null float64
international             2603 non-null object
research                  2603 non-null float64
citations                 2603 non-null float64
income                    2603 non-null object
total_score               2603 non-null object
num_students              2544 non-null object
student_staff_ratio       2544 non-null float64
international_students    2536 non-null object
female_male_ratio         2370 non-null object
year                      2603 non-null int64
dtypes: float64(4), int64(1), object(9)
memory usage: 284.8+ KB
None
```

||world_rank|university_name|country|teaching|international|research|citations|income|total_score|num_students|student_staff_ratio|international_students|female_male_ratio|year|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|0|1|Harvard University|United States of America|99.7|72.4|98.7|98.8|34.5|96.1|20,152|8.9|25%||2011|
|1|2|California Institute of Technology|United States of America|97.7|54.6|98|99.9|83.7|96|2,243|6.9|27%|33 : 67|2011|
|2|3|Massachusetts Institute of Technology|United States of APandas tutorial

# 基本介绍

Pandas 是 Python 语言下的一个用于数据分析的工具类库。使用 Pandas 可以方便的对数据进行处理和分析。


# Data Structures

## Series

## DataFrame

## Index Object

Other Functionalities on Indexes
- Reindexing
- Dropping
- Arithmertica|97.8|82.3|91.4|99.9|87.5|95.6|11,074|9|33%|37 : 63|2011|
|3|4|Stanford University|United States of America|98.3|29.5|98.1|99.2|64.3|94.3|15,596|7.8|22%|42:58:00|2011|
|4|5|Princeton University|United States of America|90.9|70.3|95.4|99.9|-|94.2|7,929|8.4|27%|45:55:00|2011|
|5|6|University of Cambridge|United Kingdom|90.5|77.7|94.1|94|57|91.2|18,812|11.8|34%|46:54:00|2011|
|6|6|University of Oxford|United Kingdom|88.2|77.2|93.9|95.1|73.5|91.2|19,919|11.6|34%|46:54:00|2011|
|7|8|University of California, Berkeley|United States of America|84.2|39.6|99.3|97.8|-|91.1|36,186|16.4|15%|50:50:00|2011|
|8|9|Imperial College London|United Kingdom|89.2|90|94.5|88.3|92.9|90.6|15,060|11.7|51%|37 : 63|2011|
|9|10|Yale University|United States of America|92.1|59.2|89.7|91.5|-|89.5|11,751|4.4|20%|50:50:00|2011|

- 导入数据
- 创建 trace
	- x = x 轴数据
	- y = y 轴数据
	- mode = 绘制标记的类型
	- name = 图例名称
	- marker = 标记的样式
		- color = 线条的颜色，使用 RGB 定义
- text = 坐标的名字
- data = 一个列表，表示要绘制的数据
- layout = 一个字典，表示布局信息
	- title = 图标的标题
	- x axis = 表示 x 轴的样式信息
		- title = x 轴的标签
		- ticklen = x 轴坐标轴上竖线的长度
		- zeroline = 布尔值，是否显示0坐标位置的线条，也就是 y 轴
- fig = 一个包含数据和布局信息的字典
- plot = 绘制图形，这里采用本地离线方式绘制
		-  and Data Alignment

|index|value|
|:-:|:-:|
|0|12|
|1|-4|
|2|7|
|3|9|
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2OTMyNjY1MzksLTMyNjI3MTg0MywxNj
gyMzIxMDMsMTY5MjA2OTQzXX0=
-->