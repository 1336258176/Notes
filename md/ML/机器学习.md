# 传统

- K-means 聚类
- K 邻近聚类
- 决策树
- 随机森林
- SVM

# 新型

- [[深度学习]]

# 非监督学习

>在非监督学习中，模型通过对数据进行聚类、降维、关联规则挖掘等技术，来发现数据中的潜在模式和结构。

## 聚类

聚类是一种无监督学习方法，用于将数据样本划分为不同的组，使得组内的样本相似度较高，而组间的相似度较低。聚类算法通过寻找数据样本之间的内在结构和模式来完成这种划分。

- K-means
	K-means 是一种迭代的、划分的聚类算法。它将数据样本划分为 K 个簇，使得每个样本与所属簇的质心（簇的中心点）之间的距离最小化。算法的步骤包括选择初始质心、分配样本到最近的质心、更新质心位置，直到达到收敛条件。

- 层次聚类
	层次聚类是一种基于数据样本之间相似度的层次分解方法。它可以产生一个聚类的层次结构，从而提供了不同层次的聚类结果。层次聚类有两种主要方法：凝聚（自下而上）和分裂（自上而下）。凝聚方法从单个样本开始，逐步合并相似的样本，形成聚类。分裂方法从所有样本开始，逐步分割聚类，形成更细粒度的聚类。

- 密度聚类
	密度聚类算法通过计算样本周围的密度来划分簇。它根据密度高于某个阈值的区域来确定聚类，而将低密度区域视为噪声或边界。DBSCAN (Density-Based Spatial Clustering of Applications with Noise) 是一种常见的密度聚类算法，它根据样本在特征空间中的密度连接性来划分簇。

- 均值漂移聚类
	均值漂移聚类算法基于样本密度的估计，通过在样本密度较高的区域进行密度估计和漂移来寻找聚类中心。算法从随机选择的样本点开始，通过迭代寻找密度梯度最大的方向，直到收敛到局部最大密度区域的聚类中心。

# 监督学习

> 在监督学习中，模型通过使用带有已知标签或目标值的训练数据集进行训练，以学习输入数据和目标值之间的关系。然后，该模型可以用于预测新的未标记数据的目标值。

- K 近邻
- 决策树
- SVM

# 在 OpenCV 中的体现

## StatModel 基类

`StatModel::train()`  用于训练模型

```cpp
virtual bool train(const Ptr<TrainData> &trainData, int flag=0)
```

其中 `Ptr<TrainData>` 数据类型为机器学习模型训练专用类型，构建：

```cpp
static Ptr<TrainData> cv::ml::TrainData::create(InputArray sample, 
												int layout, 
												InputArray responses,
												InputArray varldx=noArray(),
												InputArray sampleldx=noArray(),
												InputArray sampleWeights=noArray(),
												InputArray varType=noArray())
```

其中 `sample` 样本矩阵数据类型必须为 `CV_32F` 

`StatModel::predict()` 用于预测

```cpp
virtual float cv::ml::StatModel::predict(InputArray samples,
										OutputArray results=noArray(),
										int flags=0)
```