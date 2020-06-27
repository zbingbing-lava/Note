# Chapter8 分类数据

## 1.Category的创建及其性质

#### 1.1 分类变量的创建

**用Series创建**

`pd.Series(["a", "b", "c", "a"], dtype="category")`

**用DataFrame指定类型创建**

```python
temp_df = pd.DataFrame({'A':pd.Series(["a", "b", "c", "a"], dtype="category"),'B':list('abcd')})
temp_df.dtypes
```

**利用内置Categorical类型创建**

```python
cat = pd.Categorical(["a", "b", "c", "a"], categories=['a','b','c'])
pd.Series(cat)
```

**利用cut函数创建**

**使用区间类型为标签**

`pd.cut(np.random.randint(0,60,5), [0,10,30,60])`

**可指定字符为标签**

```python
pd.cut(np.random.randint(0,60,5), [0,10,30,60], right=False, labels=['0-10','10-30','30-60'])
```

#### 1.2.分类变量的结构

一个分类变量包括三个部分，元素值（values）、分类类别（categories）、是否有序（order），使用cut函数创建的分类变量默认为有序分类变量

**desrcibe方法**

描述了分类序列的情况，包括非缺失值个数、元素值类别数(元素里出现的类别数)，最多出现的元素及其频数

```python
s = pd.Series(pd.Categorical(["a", "b", "c", "a",np.nan], categories=['a','b','c','d']))
s.describe()
```

**categories和ordered属性**

```python
s.cat.categories#查看分类类别
s.cat.ordered#查看是否排序
```

#### 1.3 类别的修改

**利用set_categories修改分类，但本身值不会变化**

**利用rename_categories会把值和分类同时修改**

![image-20200627103429377](../../ProgramFiles/Typora/upload/image-20200627103429377.png)

**利用字典修改值，值和分类也会同时修改**

`s.cat.rename_categories({'a':'new_a','b':'new_b'})`

**利用add_categories添加类，remove_categories移除类,remove_unused_categories删除为出现的分类类型**

```python
s = pd.Series(pd.Categorical(["a", "b", "c", "a",np.nan], categories=['a','b','c','d']))
s.cat.add_categories(['e'])
s.cat.add_categories(['a'])
s.cat.remove_unused_categories()
```

## 2.分类变量的排序

### 2.1 序的建立

**as_ordered()将分类序列转有序变量，as_unordered()转为无序变量**

```python
s = pd.Series(["a", "d", "c", "a"]).astype('category').cat.as_ordered()
```

**利用set_categories方法中的order参数,新设置的分类不需要与原分类为同一集合**

```python
pd.Series(["a", "d", "c", "a"]).astype('category').cat.set_categories(['a','c','d'],ordered=True)
```

**利用reorder_categories方法中的order参数,新设置的分类必须与原分类为同一集合**

```python
s = pd.Series(["a", "d", "c", "a"]).astype('category')
s.cat.reorder_categories(['a','c','d'],ordered=True)
```

### 2.2排序

**值排序**

```python
s = pd.Series(np.random.choice(['perfect','good','fair','bad','awful'],50)).astype('category') s.cat.set_categories(['perfect','good','fair','bad','awful'][::-1],ordered=**True**).head()
s.sort_values(ascending=False).head()
```

**索引排序**

```python
df_sort = pd.DataFrame({'cat':s.values,'value':np.random.randn(50)}).set_index('cat')
df_sort.head()
df_sort.sort_index().head()
```

### 3.分类变量的比较操作

#### 3.1 与标量或等长序列比较

**标量比较，series中的每个值都进行比较，得到一个bool型series**

```
s = pd.Series(["a", "d", "c", "a"]).astype('category') 
s == 'a'
```

**与等长序列比较**

```python
s == list('abcd')
```

#### 3.2 与另一分类变量比较

**两个分类变量的等式判别(等号与不等号)需要满足分类完全相同，否则报错**

```python
s = pd.Series(["a", "d", "c", "a"]).astype('category')
s == s
s != s
s_new = s.cat.set_categories(['a','d','e'])
s == s_new #这句代码会报错
```

**不等式判别：需要分类完全相同且排序完全相同，否则会报错**

## 4.问题与练习

**【问题一】 如何使用union_categoricals方法？它的作用是什么？**

合并无序类别

```python
a = pd.Categorical(["b", "c"])
b = pd.Categorical(["a", "b"])
union_categoricals([a, b])#合并后为[b,c,a,b]
#sort_categories=True参数可以设置排序，如果不设置排序，那么默认排序为出现顺序
```

**【问题二】 利用concat方法将两个序列纵向拼接，它的结果一定是分类变量吗？什么情况下不是？**

不是，取决于原序列的类别

**【问题三】 当使用groupby方法或者value_counts方法时，分类变量的统计结果和普通变量有什么区别？**

分类变量统计结果依旧为分类变量

![image-20200627112402759](https://raw.githubusercontent.com/zbingbing-lava/pic/master/img/20200627205731.png)

**【问题四】 下面的代码说明了Series创建分类变量的什么“缺陷”？如何避免？（提示：使用Series中的copy参数）**

修改值后对应的分类也不会改变

**【练习一】 现继续使用第四章中的地震数据集，请解决以下问题：**
**（a）现在将深度分为七个等级：[0,5,10,15,20,30,50,np.inf]，请以深度等级Ⅰ,Ⅱ,Ⅲ,Ⅳ,Ⅴ,Ⅵ,Ⅶ为索引并按照由浅到深的顺序进行排序。**

```python
df = pd.read_csv('E:/六月组队pandas(下)/joyful-pandas-master/data/Earthquake.csv')
df2 = df.copy()
df2['深度'] = pd.cut(df2['深度'], [-1e-10,5,10,15,20,30,50,np.inf],labels=['Ⅰ','Ⅱ','Ⅲ','Ⅳ','Ⅴ','Ⅵ','Ⅶ'])
df2.set_index('深度').sort_index().head()#设置索引并排序
```

**（b）在（a）的基础上，将烈度分为4个等级：[0,3,4,5,np.inf]，依次对南部地区的深度和烈度等级建立多级索引排序。**

```python
df2['烈度'] = pd.cut(df2['烈度'], [-1e-10,3,4,5,np.inf],labels=['Ⅰ','Ⅱ','Ⅲ','Ⅳ'])
df2.set_index(['深度','烈度']).sort_index().head()
```

**【练习二】 对于分类变量而言，调用第4章中的变形函数会出现一个BUG（目前的版本下还未修复）：例如对于crosstab函数，按照官方文档的说法，即使没有出现的变量也会在变形后的汇总结果中出现，但事实上并不是这样，比如下面的例子就缺少了原本应该出现的行'c'和列'f'。基于这一问题，请尝试设计my_crosstab函数，在功能上能够返回正确的结果。**

这个不会实现，以下放出提供的参考答案

```python
foo = pd.Categorical(['a', 'b'], categories=['a', 'b', 'c'])
bar = pd.Categorical(['d', 'e'], categories=['d', 'e', 'f'])
pd.crosstab(foo, bar)

def my_crosstab(foo,bar):
    num = len(foo)
    s1 = pd.Series([i for i in list(foo.categories.union(set(foo)))],name='1nd var')
    s2 = [i for i in list(bar.categories.union(set(bar)))]
    df = pd.DataFrame({i:[0]*len(s1) for i in s2},index=s1)
    for i in range(num):
        df.at[foo[i],bar[i]] += 1
    return df.rename_axis('2st var',axis=1)
my_crosstab(foo,bar)
```

