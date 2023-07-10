# 异常值检测

## Z-score 方法

如果数据服从正态分布，依据 3σ 法则，异常值被定义与平均值的偏差超过三倍标准差的值。在正态分布下，距离平均值 3σ 之外的值出现的概率为 $P(|x-\mu|>3\sigma)<0.003$ ，属于小概率事件。如果数据不服从正态分布，那么可以用远离平均值的多少倍的标准差来描述，这里的倍数就是 Z-score。Z-score 以标准差为单位去度量某一原始分数偏离平均值的距离，公式如下所示。 $$ z = \frac {X - \mu} {\sigma} $$ Z-score 需要根据经验和实际情况来决定，通常把远离标准差 `3` 倍距离以上的数据点视为离群点

**代码实现**
```python
import numpy as py

def detect_outliers_zscore(data, threshold=3):
	avg = np.mean(data)
	std = np.std(data)
	z_score = np.abs((data - avg) / std)
	return data[z_score > threshold]
```

## IQR

四分位距 (IQR) 是一种衡量变异性的方法，它通过将数据集划分为四分位数来实现。四分位数将一个按等级排序的数据集划分为四个相等的部分。即 $Q_1$（第 1 个四分位数）、$Q_2$（第 2 个四分位数）和 $Q_3$（第 3 个四分位数）。$IQR$ 定义为 $Q_3 - Q_1$，位于 $Q_1 - \frac{3}{2} IQR$ 或 $Q_3 + \frac{3}{2} IQR$ 之外的数据被视为离群值。
![[Pasted image 20230708160715.jpg]]

- 四分位数的选取
- ![[e59b9be58886e4bd8de8b79d-interquartile-range2.webp]]

## 删除异常值
```python
import pandas

emp_df.drop(emp_df[(emp_df.sal > 8000) | (emp_df.sal < 2000)].index)
```

## 替换异常值
```python
import pandas
import numpy as np

avg = np.mean(emp_df.sal).astypr(int)
emp_df.replace({'sal':[1800,9000],'comm':800},{'sal':avg,'comm':1000})
```

# 相关性判定

在统计学中，我们通常使用协方差（covariance）来衡量两个随机变量的联合变化程度。如果变量 $X$ 的较大值主要与另一个变量 $Y$ 的较大值相对应，而两者较小值也相对应，那么两个变量倾向于表现出相似的行为，协方差为正。如果一个变量的较大值主要对应于另一个变量的较小值，则两个变量倾向于表现出相反的行为，协方差为负。简单的说，协方差的正负号显示着两个变量的相关性。方差是协方差的一种特殊情况，即变量与自身的协方差。
$$ cov\left( X,Y \right) = E \left( \left( X- \right) \right) $$

# 数据可视化

## 指南

![[Pasted image 20230709142950.png]]

## matplotlib

### 配置
```python
import matplotlib.pyplot as plt

# 配置支持中文的非衬线字体（默认的字体无法显示中文）
plt.rcParams['font.sans-serif'] = ['SimHei', ]
# 使用指定的中文字体时需要下面的配置来避免负号无法显示
plt.rcParams['axes.unicode_minus'] = False
```

### 折线图
```python
x = np.linspace(-2 * np.pi, 2 * np.pi, 120)
y1, y2 = np.sin(x), np.cos(x)

plt.figure(figsize=(8, 4), dpi=120)
plt.plot(x, y1, linewidth=2, marker='*', color='red')
plt.plot(x, y2, linewidth=2, marker='^', color='blue')
# 定制图表的标注，其中的arrowprops是定制箭头样式的参数
plt.annotate('sin(x)', xytext=(0.5, -0.75), xy=(0, -0.25), fontsize=12, arrowprops={
    'arrowstyle': '->', 'color': 'darkgreen', 'connectionstyle': 'angle3, angleA=90, angleB=0'
})
plt.annotate('cos(x)', xytext=(-3, 0.75), xy=(-1.25, 0.5), fontsize=12, arrowprops={
    'arrowstyle': '->', 'color': 'darkgreen', 'connectionstyle': 'arc3, rad=0.35'
})
plt.show()
```
![[Pasted image 20230709144826.png]]

### 柱状图
```python
x = np.arange(4)
y1 = np.random.randint(20, 50, 4)
y2 = np.random.randint(10, 60, 4)
plt.figure(figsize=(6, 4), dpi=120)
# 通过横坐标的偏移，让两组数据对应的柱子分开
# width参数控制柱子的粗细，label参数为柱子添加标签
plt.bar(x - 0.1, y1, width=0.2, label='销售A组')
plt.bar(x + 0.1, y2, width=0.2, label='销售B组')
# 定制横轴的刻度
plt.xticks(x, labels=['Q1', 'Q2', 'Q3', 'Q4'])
# 定制显示图例
plt.legend()
plt.show()
```
![[Pasted image 20230709145247.png]]

### 饼图
```python
import matplotlib.pyplot as plt  
import numpy as np  
  
y = np.array([35, 25, 25, 15])  
mylabels = ["Apples", "Bananas", "Cherries", "Dates"]  
myexplode = [0.2, 0, 0, 0]  #扇形突出
  
plt.pie(y, autopct='%.1f%%', labels = mylabels, explode = myexplode, shadow = True)  
plt.legend(title='Four Fruits:')
plt.show()
```
![[Pasted image 20230709145745.png]]

### 散点图

```python
import matplotlib.pyplot as plt  
import numpy as np  
  
#day one, the age and speed of 13 cars:  
x = np.array([5,7,8,7,2,17,2,9,4,11,12,9,6])  
y = np.array([99,86,87,88,111,86,103,87,94,78,77,85,86])  
plt.scatter(x, y)  
  
#day two, the age and speed of 15 cars:  
x = np.array([2,2,8,1,15,8,12,9,7,3,11,4,7,14,12])  
y = np.array([100,105,84,105,90,99,90,95,94,100,79,112,91,80,85])  
plt.scatter(x, y)  
  
plt.show()
```
![[Pasted image 20230709140425.png]]

### 直方图

```python
import numpy as np
import matplot.pyplot as plt
x = np.random.normal(170,10,250)
#随机生成一个包含 250 个值的数组，其中值集中在 170 左右，标准差为 10
plt.hist()
plt.show()
```
![[Pasted image 20230709141204.png]]

## Seaborn

### 模板图样

https://seaborn.pydata.org/examples/index.html



# 参考资料

- [Pandas的应用](https://github.com/jackfrued/Python-100-Days/blob/master/Day66-80/72.Pandas%E7%9A%84%E5%BA%94%E7%94%A8-3.md)
- [四分位距](https://cg2010studio.com/2018/05/27/%e5%9b%9b%e5%88%86%e4%bd%8d%e8%b7%9d-interquartile-range/)
- [IQR（Interquartile Range，四分位距）](https://docs.oracle.com/cloud/help/zh_CN/pbcs_common/PFUSU/insights_metrics_IQR.htm#PFUSU-GUID-CF37CAEA-730B-4346-801E-64612719FF6B)