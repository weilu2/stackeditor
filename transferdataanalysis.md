
# 写入速度分析

分析
绘制数据写入 HBase 的折线图。
```Python
import pandas as pd

df = pd.read_csv("write.csv")

ds = df[df['data/W'] % 100000 == 0]

ds['data/W'] = ds['data/W'] / 10000
ds['time/Min'] = ds['time/Min'] / 60000

import seaborn as sns
import matplotlib.pyplot as plt

plt.rcParams['font.sans-serif'] =['Microsoft YaHei']
f, ax= plt.subplots(figsize = (14, 8))
ax.set_title('Write Speed')

sns.set_style("darkgrid") 
sns.pointplot(x="data/W",y="time/Min",data=ds)
plt.show()
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1NzkyOTkxMV19
-->