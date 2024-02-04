***
# 必须

- 如果浏览官方文档，会发现 `Seaborn` 中还存在大量已大些字母开始的类，例如 `JointGrid`，`PairGrid` 等。实际上这些类只是其对应小写字母的函数 `jointplot`，`pairplot` 的进一步封装。当然，二者可能稍有不同，但并没有本质的区别。
- `despine()` 函数可以用于在 `Seaborn` 中移除图形的边框（也称为“脊柱”），从而改善可视化效果。
- `plt.savefig()` 函数可以将图片保存为 pdf 等格式

```python
plt.rcParams["font.sans-serif"] = "SimHei"  # 指定中文字体  
plt.rcParams["axes.unicode_minus"] = False  # 解决负号显示问题  
sns.set(font='SimHei') # matplotlib 使用 seaborn 格式，设置 seaborn 字体为黑体
#sns.set(context='notebook', style='darkgrid', palette='deep', font='sans-serif', font_scale=1, color_codes=False, rc=None)
```

# 关系图

## reaplot 关系图

* `scatterplot` 和 `lineplot` 的结合版
* [`relplot`](https://seaborn.pydata.org/generated/seaborn.relplot.html) 是 `relational plots` 的缩写，其可以用于呈现数据之后的关系，主要有散点图和条形图两种样式。
* 另外线形态绘制时还会自动给出 95% 的置信区间。

```python
iris = sns.load_dataset("iris")
#可读入多维数据，但要指定要绘制的x，y
# hue-分类 style-改变不同类型的点 data-数据（可以是DataFrame）
# kind-可指定line，绘制lineplot，默认绘制scatterplot
sns.relplot(x="sepal_length", y="sepal_width",
            hue="species", style="species", data=iris)

sns.relplot(x="sepal_length", y="petal_length",
            hue="species", style="species", kind="line", data=iris)
```
![[Pasted image 20230825204823.png]]
![[Pasted image 20230825204849.png]]
## scatterplot 多维分析散点图

## lineplot 多维分析线性图

# 回归图

## regplot

* `regplot` 绘制回归图时，只需要指定自变量和因变量即可，`regplot` 会自动完成线性回归拟合。

```python
sns.regplot(x="sepal_length", y="sepal_width", data=iris)
```
![[Pasted image 20230825213125.png]]

## lmplot

* `lmplot` 同样是用于绘制回归图，但 `lmplot` 支持引入第三维度进行对比，例如我们设置 `hue="species"`。

```python
sns.lmplot(x="sepal_length", y="sepal_width", hue="species", data=iris)
```
![[Pasted image 20230825213230.png]]

# 分布图

- 分布图主要是用于可视化变量的分布情况，一般分为单变量分布和多变量分布。当然这里的多变量多指二元变量，更多的变量无法绘制出直观的可视化图形。

## distplot

- `Seaborn` 快速查看单变量分布的方法是 `distplot`。默认情况下，该方法将会绘制直方图并拟合核密度估计图。
- `distplot` 提供了参数来调整直方图和核密度估计图，例如设置 `kde=False` 则可以只绘制直方图，或者 `hist=False` 只绘制核密度估计图。当然，`kdeplot` 可以专门用于绘制核密度估计图，其效果和 `distplot(hist=False)` 一致，但 `kdeplot` 拥有更多的自定义设置。
```python
sns.distplot(iris["sepal_length"])
```
![[Pasted image 20230825213624.png]]

## joinplot

- `jointplot` 主要是用于绘制二元变量分布图。
- 另外其支持 `kind=` 参数指定绘制出不同样式的分布图。

### 普通散点图
```python
sns.jointplot(x="sepal_length", y="sepal_width", data=iris)
```
![[Pasted image 20230825213841.png]]

### 核密度估计对比图
```python
sns.jointplot(x="sepal_length", y="sepal_width", data=iris, kind="kde")
```
![[Pasted image 20230825214031.png]]

### 六边形计数图
```python
sns.jointplot(x="sepal_length", y="sepal_width", data=iris, kind="hex")
```
![[Pasted image 20230825214252.png]]

### 回归拟合图
```python
sns.jointplot(x="sepal_length", y="sepal_width", data=iris, kind="reg")
```
![[Pasted image 20230825214355.png]]

## pairplot

- `pairplot` 更加强大，其支持一次性将数据集中的特征变量两两对比绘图。**默认情况下，对角线上是单变量分布图，而其他则是二元变量分布图**。

```python
sns.pairplot(iris)
```
![[Pasted image 20230825214555.png]]

```python
sns.pairplot(iris, hue="species")
```
![[Pasted image 20230825214634.png]]

# 矩阵图

## 热力图

- 意如其名，`heatmap` 主要用于绘制热力图。热力图在某些场景下非常实用，例如绘制出变量相关性系数热力图。
```python
import numpy as np

sns.heatmap(np.random.rand(10, 10))
```
![[Pasted image 20230825214954.png]]

## 层次聚类结构图

- `clustermap` 支持绘制层次聚类结构图。如下所示，我们先去掉原数据集中最后一个目标列，传入特征数据即可。当然，你需要对层次聚类有所了解，否则很难看明白图像多表述的含义。
```python
iris.pop("species")
sns.clustermap(iris)
```
![[Pasted image 20230825215046.png]]

# 类别图

- 与关联图相似，类别图的 `Figure-level` 接口是 `catplot`，其为 `categorical plots` 的缩写。
- `catplot()` 函数默认绘制 `kind='strip'` 类型图
- `hue=` 参数可以给图像引入另一个维度。如果一个数据集有多个类别，`hue=` 参数就可以让数据点有更好的区分。
## 分类散点图

- `stripplot()` ( `kind="strip"` )
```python
sns.catplot(x="sepal_length", y="species", data=iris)
```
![[Pasted image 20230826154235.png]]

- `swarmplot()` （ `kind='swarm'` ）
- `kind="swarm"` 可以让散点按照 beeswarm 的方式防止重叠，可以更好地观测数据分布。
```python
sns.catplot(x="sepal_length", y="species", kind="swarm", data=iris)
```
![[Pasted image 20230826154334.png]]

## 箱线图

- `kind='box'`
```python
sns.catplot(x="sepal_length", y="species", kind="box", data=iris)
```
![[Pasted image 20230826154539.png]]

## 小提琴图

- `kind='violin'`
```python
sns.catplot(x="sepal_length", y="species", kind="violin", data=iris)
```
![[Pasted image 20230826154622.png]]

## 增强箱线图

- `kind='boxen'`
```python
sns.catplot(x="species", y="sepal_length", kind="boxen", data=iris)
```
![[Pasted image 20230826154719.png]]

## 点线图

- `kind='point'`
```python
sns.catplot(x="sepal_length", y="species", kind="point", data=iris)
```
![[Pasted image 20230826154828.png]]

## 条形图

- `kind='bar'`
```python
sns.catplot(x="sepal_length", y="species", kind="bar", data=iris)
```
![[Pasted image 20230826154940.png]]

## 计数条形图

- `kind='count'`
```python
sns.catplot(x="species", kind="count", data=iris)
```
![[Pasted image 20230826155028.png]]
# 参考资料

- [Seaborn 数据可视化基础教程](https://huhuhang.com/post/machine-learning/seaborn-basic)
- [官方样例](https://seaborn.pydata.org/examples/index.html)
- [详解 Seaborn，看这一篇就够了](https://blog.csdn.net/qq_42676511/article/details/125178293)