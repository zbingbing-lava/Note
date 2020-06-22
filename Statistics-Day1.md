# Statistics-Day1

**学习任务：**

- 做理论知识点的笔记；
- python实现二项分布，协方差和相关系数以及贝叶斯公式；

## 1.随机事件

### 1.1 基本概念

**随机现象**:事先已知有多种可能，但是无法确定哪一种可能会发生

**随机实验E**：使随机现象得以实现和观察的全过程。需要满足三个条件：(1)可以在相同条件下重复进行 (2)结果有多种可能性，并且所有可能结果事先已知 (3)事先无法确定实验出现的结果

**样本空间**：随机试验的所有可能结果组成的空间

**样本点**：每一个可能的结果

**随机事件**：样本空间中满足一定条件的子集

**必然事件**：每次试验中总发生的事件

**不可能事件**：空集，不发生的事件

### 1.2 概率

**概率P(A)**: 事件A发生的可能性

**古典概型**：随机事件的样本空间中只有有限个样本点，且每个样本点的发生是等可能的，每次试验有且仅有一个样本点发生。

**条件概率**：A和B 是两个事件，且P(B)>0, P(A|B)=P(AB)/P(B)为在事件B发生的概率下事件A发生的概率

**全概率公式**：可以理解为把样本空间分为了多个子空间，即多个事件组B，那么事件A在样本空间中发生的概率为在所有样本空间中发生的概率之和：

<img src="https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200622191606.png" alt="image-20200622163932969" style="zoom: 33%;" />

借助因果关系来理解，B为导致A发生的原因，生活中有很多种原因A都有可能发生，所以有B1，B2...  那么当B这个原因发生了，A才会发生，所以P(A)为 **B发生的概率** * **在B发生的基础上A发生的概率**，然后再把所有的B空间进行求和

**贝叶斯公式**：

<img src="https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200622164044.png" alt="image-20200622164042850" style="zoom:50%;" />

继续借助上面的因果关系进行理解，这里是由**果**推导**因**，P(B|A)表示事件A发生了，那么事件A是由事件B导致的可能性有多大。

P(B)为先验概率，P(B|A)为后验概率。也可以理解为P(B)是已知的，而P(B|A)是要求得的。

## 2. 随机变量

### 2.1 随机变量及其分布

**随机变量**：定义在样本空间上，取值在实域上的函数。

**随机变量的分布函数(概率累积函数)**：

![image-20200622165814465](https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200622191629.png)

### 2.2 离散型随机变量

离散型随机变量的分布函数：

![image-20200622170126485](https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200622191636.png)

**伯努利实验**：一个试验只有两种可能的结果A与非A。 伯努利试验独立重复进行n次称为n重伯努利试验。

**分布律**：

![image-20200622170351041](https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200622170352.png)

**分布函数**：

![image-20200622170401118](https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200622170402.png)

**数学期望**(代表了随机变量取值的平均值)：

- 离散型：![image-20200622170634969](https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200622191654.png)
- 连续型：![image-20200622170701602](https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200622170702.png)

- 数学期望的性质：

![image-20200622170833485](https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200622191703.png)

**方差**(描述随机变量取值相对均值的离散度)：

<img src="https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200622170947.png" alt="image-20200622170946058" style="zoom:80%;" />

- 方差的性质：

<img src="https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200622171018.png" alt="image-20200622171017017" style="zoom:80%;" />

**协方差**(描述线性相关程度)：

<img src="https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200622171135.png" alt="image-20200622171133859" style="zoom:80%;" />

为什么协方差可以描述线性相关？因为方差是X=Y时协方差的特例，而X=Y是线性相关的。(个人理解)

- 协方差性质：

![image-20200622180621595](https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200622180633.png)

相关系数（无量纲）：值处于-1~1，小于0代表负相关，大于0代表正相关，绝对值表示相关性大小。

<img src="https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200622180821.png" alt="image-20200622180820201" style="zoom:67%;" />

## 3.代码实现

### 3.1 二项分布

python的numpy中有二项分布采样的函数：#n表示试验次数，p试验得到正例的概率，size采样次数。返回结果为n次试验中成功的次数

`numpy.random.binomial(n,p,size=None)`

```python
import numpy as np
#二项分布
n, p = 10, .5 
s = np.random.binomial(n, p, 1000)
```

### 3.2 协方差

numpy中实现：

```python
numpy.cov(m, y=None, rowvar=True, bias=False, ddof=None, fweights=None, aweights=None)
#参数详见https://numpy.org/doc/stable/reference/generated/numpy.cov.html
```

```python
x = [-2.1, -1,  4.3]
y = [3,  1.1,  0.12]
X = np.stack((x, y), axis=0)#把x和y连接
np.cov(X)
np.cov(x,y)
```

pandas中实现：dataframe.cov(): 计算所有变量之间的协方差

```python
import pandas as pd
import numpy as np
data = pd.DataFrame({
        "A":[8,9,10,9,12],
        "B":[6,3,7,9,2],
        'C':[1,2,3,4,5]
    })
print(data.cov())
```

### 3.3 相关系数

```python
np.corrcoef(a, b)
np.corrcoef(x)#一行代表一个变量
```

```python
data.corr()#计算相关系数，还可指定具体pearson,spearman等
```

### 3.4 贝叶斯公式

scikit-learn朴素贝叶斯类库：GaussianNB(先验为高斯分布)，MultinomialNB(先验为多项式分布)和BernoulliNB(先验为伯努利分布)

原文链接：https://www.cnblogs.com/pinard/p/6074222.html

```python
import numpy as np
X = np.array([[-1, -1], [-2, -1], [-3, -2], [1, 1], [2, 1], [3, 2]])
Y = np.array([1, 1, 1, 2, 2, 2])
from sklearn.naive_bayes import GaussianNB
clf = GaussianNB()
#拟合数据
clf.fit(X, Y)
print "==Predict result by predict=="
print(clf.predict([[-0.8, -1]]))
print "==Predict result by predict_proba=="
print(clf.predict_proba([[-0.8, -1]]))
print "==Predict result by predict_log_proba=="
print(clf.predict_log_proba([[-0.8, -1]]))
```

## 4.总结

PS：记得以后要补上朴素贝叶斯算法的自己实现