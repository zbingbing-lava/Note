# 第九章 时序数据

## 一、时序的创建

四类时间变量

![image-20200629165008828](https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200629165010.png)

### 1.**时间点的创建**

**to_datatime()**方法：`pd.to_datetime('2020.1.1')`

当格式报错时可以采用参数强制匹配：`pd.to_datetime('2020\\1\\1',format='%Y\\%m\\%d')`

可以用列表转为时间点索引：

```python
pd.Series(range(2),index=pd.to_datetime(['2020/1/1','2020/1/2']))
```

**用to_datetime**将多列dataframe转换为时间点

```python
df = pd.DataFrame({'year': [2020, 2020],'month': [1, 1], 'day': [1, 2]})
pd.to_datetime(df)
```

Timestamp的精度最小到纳秒ns

时间范围：1677-09-21  ~  2262-04-11



**date_range()创建时间点序列**

```python
pd.date_range(start='2020/1/1',end='2020/1/10',periods=3)
pd.date_range(start='2020/1/1',end='2020/1/10',freq='D')
```

periods指时间点个数，freq指间隔方法，以下为freq的选项

![image-20200629165643486](https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200629165645.png)



**bdate_range**依据weekmask参数和holidays参数定制时间序列
它的freq中有一个特殊的'C'/'CBM'/'CBMS'选项，表示定制，需要联合weekmask参数和holidays参数使用
例如现在需要将工作日中的周一、周二、周五3天保留，并将部分holidays剔除

```python
weekmask = 'Mon Tue Fri'
holidays = [pd.Timestamp('2020/1/%s'%i) for i in range(7,13)]
#注意holidays
pd.bdate_range(start='2020-1-1',end='2020-1-15',freq='C',weekmask=weekmask,holidays=holidays)
```



### 2. DataOffset对象

**DataOffset与Timedelta的区别**

Timedelta绝对时间差的特点指无论是冬令时还是夏令时，增减1day都只计算24小时
DataOffset相对时间差指，无论一天是23\24\25小时，增减1day都与当天相同的时间保持一致

这个涉及时区的时候需要注意一下，但是只要把tz = ' '参数去掉二者就一致了

**增减一段时间，直接用加减法**

DateOffset的可选参数包括years/months/weeks/days/hours/minutes/seconds

`pd.Timestamp('2020-01-01') + pd.DateOffset(minutes=20) - pd.DateOffset(weeks=2)`

## 二、时序的索引及属性

**索引切片**

```python
rng = pd.date_range('2020','2021', freq='W')
ts = pd.Series(np.random.randn(len(rng)), index=rng)
ts.head()
#合法字符能够自动转换为时间点
ts['2020-01-26':'20200726'].head()#ts['2020-01-26':'2020-07-26'].head()
#混合形态索引
ts['2011-1':'20200726'].head()
```

**子集索引**：例如从数据集中提取某一个月份的数据

```python
ts['2020-7'].head()
```

**时间点属性**:例如获得是第几周，第几天

```python
pd.Series(ts.index).dt.week.head()
pd.Series(ts.index).dt.day.head()
```

**strftime修改时间格式**

```python
pd.Series(ts.index).dt.strftime('%Y-间隔1-%m-间隔2-%d').head()
```

**datatime对象通过属性获取信息**

```python
pd.date_range('2020','2021', freq='W').month
```



## 三、重采样

**resample参数**

```python
df_r = pd.DataFrame(np.random.randn(1000, 3),index=pd.date_range('1/1/2020', freq='S', periods=1000),columns=['A', 'B', 'C'])
r = df_r.resample('3min')#参数参照前面的offset字符
r.sum()
```

**采样聚合**

```python
r = df_r.resample('3T')
r['A'].agg([np.sum, np.mean, np.std])
#lambda表达式计算极差
r.agg({'A': np.sum,'B': lambda x: max(x)-min(x)})
```

**采样组的迭代**

```python
small = pd.Series(range(6),index=pd.to_datetime(['2020-01-01 00:00:00', '2020-01-01 00:30:00', '2020-01-01 00:31:00','2020-01-01 01:00:00','2020-01-01 03:00:00','2020-01-01 03:05:00']))
resampled = small.resample('H')
for name, group in resampled:
    print("Group: ", name)
    print("-" * 27)
    print(group, end="\n\n")
```

## 4.窗口函数

```python
s = pd.Series(np.random.randn(1000),index=pd.date_range('1/1/2020', periods=1000))
```

### 4.1 rolling

所谓rolling方法，就是规定一个窗口，它和groupby对象一样，本身不会进行操作，需要配合聚合函数才能计算结果

```python
s.rolling(window=50)
s.rolling(window=50,min_periods=3).mean().head()
#min_periods是指需要的非缺失数据点数量阈值
```

使用apply聚合时，只需记住传入的是window大小的Series，输出的必须是标量即可，比如如下计算变异系数

```python
s.rolling(window=50,min_periods=3).apply(lambda x:x.std()/x.mean()).head()
```

基于时间进行rolling,可选closed='right'（默认）\'left'\'both'\'neither'参数，决定端点的包含情况

```python
s.rolling('15D', closed='right').sum().head()
```

### 4.2 Expanding函数

普通的expanding函数等价与rolling(window=len(s),min_periods=1)，是对序列的累计计算,但是cumsum/cumprod/cummax/cummin都是特殊expanding累计计算方法

```python
s.rolling(window=len(s),min_periods=1).sum().head()
s.expanding().sum().head()
```

**元素漂移**/递推：shift/diff/pct_change都是涉及到了元素关系
①shift是指序列索引不变，但值向后移动
②diff是指前后元素的差，period参数表示间隔，默认为1，并且可以为负
③pct_change是值前后元素的变化百分比，period参数与diff类似