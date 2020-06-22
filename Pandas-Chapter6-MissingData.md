# Pandas Chapter 6 Missing Data

## 1.缺失观测及其类型

### 1.1 概念：区别None, NaN, Null, NaT

### **None**: 

特殊类型 NoneType对象，不支持任何运算，也没有内建方法。与其他数据比较返回False, 但是None=None

None的bool值为False. 使用None对bool型列表修改不改变数据类型;对数值型列表修改会自动变为np.nan, 对object修改保持不变

```python
s = pd.Series([1,2],dtype='bool')
print(s)#结果为bool
s[0] = None
print(s) #结果为bool

s = pd.Series([1,2],dtype='int')
print(s)#结果为int32
s[0] = None
print(s) #结果为float64

s = pd.Series(["1","2"])
print(s)#结果为 object
s[0] = None
print(s) #结果为 object
```

对None进行比较

```python
s1 = pd.Series([1,None],dtype='bool')
s2 = pd.Series([1,2],dtype='bool')
print(s1.equals(s2)) #结果False
s3 = pd.Series([1,None],dtype='bool')
print(s1.equals(s3)) #结果True
```

### NaN: 

not a number. 有NaN参与的运算，其结果也一定是NaN,  NaN不等于任何东西，也不等于自己  NaN !=NaN

DataFrame之间用equals函数比较时，会略过均为np.NaN的单元格，但是任何值与np.NaN比较都不等

```python
df1 = pd.DataFrame({'col1': [1, 2], 'col2': [np.NaN, 2],'clo3':[1,0]})
df2 = pd.DataFrame({'col1': [1, 2], 'col2': [np.NaN, 2],'clo3':[1,0]})
df3 = pd.DataFrame({'col1': [1, 2], 'col2': [np.NaN, 2],'clo3':[1,np.NaN]})
print(df1.equals(df2)) #结果为True
print(df1.equals(df3)) #结果为False
```

NaN在numpy中为浮点型，原来为数值的列有NaN就会变为浮点型，原来为字符串的列会变为object

```python
print(pd.Series([1,2,3]).dtype) #结果为 int64
print(pd.Series([1,np.NaN,3]).dtype) #结果为 float64
print(pd.Series(['bb','aa',np.NaN]).dtype) #结果为 object
```

对于bool型列表, np.nan会自动填充为True

```python
pd.Series([1,np.nan,3],dtype='bool')
```

使用np.NaN修改列表时，列表的类型会被默认修改为float64

```python
s = pd.Series([True,False],dtype='bool')
s[1]=np.nan
s #输出结果为float64类型
```

### **Null**: 

空字符串 “”

### **NaT**: 

not a time 非时间空值，可以存储在 datetime 数组中以指示未知或缺失的 datetime 值。NaT 返回一个 (NaT) datetime 非时间标量值

可以理解为时间序列的NaN, 其特性参照NaN

### 1.2 判定统计缺失值函数汇总(以下函数针对pandas中的DataFrame)

<u>以下学习针对pandas中的 NaN</u>

- 判断是否为缺失值： `isna()`  `notna()`

- 统计缺失值个数： `df.isna().sum()`

- 查看整体缺失信息：`df.info()`

- 查找缺失值所在行(定位缺失值后，按pandas对 DataFrame 操作进行就可)： `df [ df['Physics'].isna() ]`

- 定位没有缺失值的列：`df[df.notna().all(1)]` ，参数1代表定位column，若改为0则定位row ,参见pandas文档如下：

  https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.all.html?highlight=all#pandas.Series.all

### 1.3 Nullable类型与NA符号

**Nullable** 整型：与原来int的符号区别在于首字母大写 **Int**

好处：前面提到的三种缺失值都会被替换为NA符号，不改变数据类型

**Nullable布尔**，作用类似，记号为**boolean**

**sring类型**，作用类似，可以区分含糊不清的object类型

**调用字符方法后，string类型返回Nullable类型，而object根据缺失数据类型而改变**

### 1.4 NA的特性

**逻辑运算**：直接作为NA进行运算，如果结果与NA有关，则结果为NA，否则正常运算

```python
True | pd.NA  #结果True
True & pd.NA  #结果 NA
```

**算数运算和比较运算**：除了以下两种运算，其他都是NA

```python
pd.NA ** 0 #结果 1
1 ** pd.NA #结果 1
```

### 1.5 Convert_dtypes方法

读取数据时把数据列转换为Nullable类型

```
pd.read_csv('data/table_missing.csv').convert_dtypes().dtypes
```

## 2.缺失数据的运算与分组

### 2.1 运算

加法(缺失值为0)， 乘法(缺失值为1)， 累计函数(忽略缺失值) 

### 2.2 groupby 方法中的缺失值

缺失值单独为一组，自动忽略

## 3.缺失值填充与剔除

### 3.1 fillna方法

```python
s = pd.Series([1,2,3,pd.NA,5],dtype = "Int64")
s.fillna(5)#值填充，填充的值要与原有的dtype一致
s.fillna(method = 'ffill') #用前一个值填充NA
s.fillna(method = 'bfill') #用后一个值填充NA
```

#### 填充的对其特性

### ![image-20200622152248266](https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200622152250.png)

```python
df.dropna(axis=0,how='all',subset=['b','c'])
```

axis=0表示删除有缺失值的行，axis=1代表删除有缺失值的列，how='all'表示全为缺失时去除，how='any'表示存在缺失就去除，subset=['b','c']表示在某几列范围内进行搜索

## 4.插值

### 4.1 线性插值

interpolate() 插值与索引顺序无关

interpolate(method='index') 依赖索引进行插值，即会收到索引间隔大小的影响

如果索引为时间，可以按照时间长短进行插值 interpolate(method='time')

其它相关参数：limit=2表示最多插入2个值，limit_direction='backward',可选forward,backward,both,默认forward; limit_area='inside'或'outside'限制插值区域，默认None

```python
s = pd.Series([np.nan,np.nan,1,np.nan,np.nan,np.nan,5,np.nan,np.nan])
print(list(s))
print(list(s.interpolate(limit=2)))
print(list(s.interpolate(limit_direction='backward')))
print(list(s.interpolate(limit_direction='forward')))
print(list(s.interpolate(limit_direction='both')))
print(list(s.interpolate(limit_area='inside')))
print(list(s.interpolate(limit_area='outside')))
```

![image-20200622152335465](https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200622152417.png)

## 5.问题与练习

### 5.1 问题

#### (1)删除比例缺失值比例超过25%的列

思路：先统计各列数据的缺失比例，删选超过25%列，删除对应列

```python
df = pd.DataFrame({'A':[1,2,3,None,3,None,None],'B':[1,2,3,6,3,5,None],'C':[1,None,3,None,3,None,None]})
s = df.isna().sum()/len(df) < 0.25
s= s[s]
df = df[s.index]
print(df)
```

#### (2)什么是Nullable类型？引入这个设计的原因

Nullable包括string,boolean,Int. 因为原来的缺失值会影响整列数据的类型，而Nullable不会

#### (3)缺失值，如何深化对他的了解

- 统计缺失值的比例，了解缺失程度
- 查找缺失值的位置，考虑缺失原因
- 进行插值，对缺失值进行补充

### 5.2 练习

【练习一】现有一份虚拟数据集，列类型分别为string/浮点/整型，请解决如下问题：

（a）请以列类型读入数据，并选出C为缺失值的行。

（b）现需要将A中的部分单元转为缺失值，单元格中的最小转换概率为25%，且概率大小与所在行B列单元的值成正比。

```python
data = pd.DataFrame({'A':[1,2,3,4,5],'B':[0.922,0.700,0.503,0.938,0.952],'C':[4,np.NaN,8,4,10]})
data = data.convert_dtypes()
print(data)
#选出C为缺失值的行
data[data['C'].isna()]
#将A中部分值转换为缺失值，B越大A越容易转换为缺失值
#思路，将B进行从大到小排序
df = data.sort_values('B',ascending = False)
df = df.reset_index(drop=True)
num = round(len(df)*0.25)
df['A'][0] = None
data = df
print(data)
```

【练习二】 现有一份缺失的数据集，记录了36个人来自的地区、身高、体重、年龄和工资，请解决如下问题：

（a）统计各列缺失的比例并选出在后三列中至少有两个非缺失值的行。

（b）请结合身高列和地区列中的数据，对体重进行合理插值。

```python
data = pd.DataFrame({'编号':[1,2,3,4,5],'地区':['A','B','C','A','B'],'身高':[157.5,202.0,169.09,166.61,185.19],'体重':[pd.NA,91.8,62.18,59.95,pd.NA],
                     '年龄':[47,25,pd.NA,77,62],'工资':[15905,pd.NA,pd.NA,5434,4242.0]})
print(data.isna().sum()/len(data))#统计缺失
df = data.iloc[:,-3:].isna()
print(data[df.sum(axis=1)>1])
```

```python
#根据地区进行分组，然后按身高插值
data = pd.DataFrame({'编号':[1,2,3,4,5],'地区':['A','B','C','A','B'],'身高':[157.5,202.0,169.09,166.61,185.19],'体重':[np.NAN,91.8,62.18,59.95,np.NAN],
                     '年龄':[47,25,np.NAN,77,62],'工资':[15905,np.NAN,np.NAN,5434,4242.0]})
group = data.groupby('地区')
print(data.head())
for x in group:
    df = x[1]
    data.loc[df.index,'体重'] = df.sort_values(by='身高').interpolate()['体重']
data.head()
```

## 6.作业总结

练习一 (1)√  （2）题意理解似乎有误：此处留下参考答案的借鉴：[https://github.com/datawhalechina/joyful-pandas/blob/master/%E5%8F%82%E8%80%83%E7%AD%94%E6%A1%88.ipynb](https://github.com/datawhalechina/joyful-pandas/blob/master/参考答案.ipynb)

```python
df = pd.read_csv('data/Missing_data_one.csv').convert_dtypes()
total_b = df['B'].sum()
min_b = df['B'].min()
df['A'] = pd.Series(list(zip(df['A'].values
                    ,df['B'].values))).apply(lambda x:x[0] if np.random.rand()>0.25*x[1]/min_b else np.nan)
df.head()
```

练习二 (1) 思路写法正确，然而审题不清，把非缺失值不少于两个，想成了缺失值不少于两个 （2）思路基本正确，与参考答案思路一致，但是奈何结果出不来啊

以下为（2）的参考解题思路：

```python
df_method_1 = df.copy()
for name,group in df_method_1.groupby('地区'):
    df_method_1.loc[group.index,'体重'] = group[['身高','体重']].sort_values(by='身高').interpolate()['体重']
df_method_1['体重'] = df_method_1['体重'].round(decimals=2)
df_method_1.head()
```

PS: 没有跟着学pandas上做作业会有点费劲，而且会绕弯子！~~要补啊