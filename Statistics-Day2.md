# 数理统计与描述性分析

**理论部分**

- 统计量与抽样；常用统计量；
- 数据集中与离散趋势的度量；
- 分布特征，偏度与峰度；

**练习部分**

- 做理论知识点的笔记；
- python实现数据各维度的描述性分析；

## 1.数理统计

总体、个体、样本观测值

常用统计量：样本均值、方差、K阶原点矩、K阶样本中心矩、顺序统计量

## 2.描述性统计

### 2.1数据集中趋势

平均数(受极值影响)、中位数(缺乏敏感性)、众数(数据有明显集中趋势时代表性较好)、百分位数

```python
import numpy as np
np.mean()#均值
np.median()#中位数
#利用numpy计数求众数,适用于非负数据集,nums为数据列表
counts = np.bincount(nums)
np.argmax(counts)
#利用scipy下的stats，nums为数据列表
from scipy import stats
stats.mode(nums)[0][0]
np.percentile(nums, 25)#求分位数，a为一个array
```

### 2.2 离散趋势

方差、标准差、极差、变异系数、四分位差

<img src="https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200624165829.png" alt="image-20200624165820646" style="zoom: 67%;" />

```python
import numpy as np
np.var()
np.std()
np.std()/np.mean()#变异系数
```

### 2.3 分布特征

概率密度函数：连续性随机变量的概率函数

分布函数：概率累积函数<img src="https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200624170220.png" alt="image-20200624170148493" style="zoom: 50%;" />

正态分布：<img src="https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200624170325.png" alt="image-20200624170323638" style="zoom:50%;" />

### 2.4 偏度/峰度

偏度：统计数据分布非对称性。均值偏度系数为0，右偏为正，左为负

峰度：刻画分布函数的集中和分散程度

```python
s = pd.Series(data)
s.skew()#偏度
s.skurt()#峰度
```

