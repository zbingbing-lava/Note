

# Pandas-Chapter7-TextData

## 1.String

**string 与object的区别**：

①字符存取方法：string会返回Nullable类型，而object的数据类型会随缺失值而改变

②某些series在string上不能使用，例如Series.str.decode()

③string在缺失值存储或运算时，类型会广播为pd.NA,而不是浮点型np.nan

**string类型的转换：**

其他类型直接转换为string可能会出错，分两步转换，先转为str型object,再转为string

`pd.Series([1,'a']).astype('str').astype('string')`

## 2.拆分与拼接

**str.split方法**

```python
s = pd.Series(['a_b_c','c_d_e',np.nan,'f_g_h'],dtype="string")
s.str.split('_')#对series中的字符串按分隔符进行分割
```

分割后为object类型，因为结果的Series中包含了list

可以用str[i]进行元素的选择

```python
s.str.split('_',expand=True)#expand控制是否将列拆开
s.str.split('_',n=1)#n代表分割多少次
```

**str.cat方法**

(1)不同对象的拼接模式

①对于单个Series来说，是所有的元素进行字符合并为一个字符串

```python
s = pd.Series(['ab',None,'d'],dtype='string')
s.str.cat()#输出结果为'abd'
```

②用sep选择分隔符，用na_rep选择缺失值代替字符

```python
s.str.cat(sep='')#结果为'ab,d'
s.str.cat(sep=',',na_rep='*')#结果为'ab,*,d'
```

③两个series合并是对应索引的元素进行合并

```python
s = pd.Series(['ab',None,'d'],dtype='string')
s2 = pd.Series(['24',None,None],dtype='string')
s.str.cat(s2)#输出结果为['ab24',None,None]，注意None与任何连接都会变成None
#进行缺失值替换时所有缺失值均会被替换
#同时有了分割符时，None不会影响cat后的字符串
s.str.cat(s2,sep=',',na_rep='*')#输出为['ab,24','*,*','d,*']
```

④多列拼接：表的拼接和多个series的拼接

```python
s.str.cat(pd.DataFrame({0:['1','3','5'],1:['5','b',None]},dtype='string'),na_rep='*')
s.str.cat([s+'0',s*2])
```

（2）cat中的索引对齐

当前版本中，如果两边合并的索引不相同且未指定join参数，默认为左连接，设置join='left',索引没有而导致的空缺会被替换为缺失值替代符

```python
s = pd.Series(['ab',None,'d'],dtype='string')
s2 = pd.Series(list('abc'),index=[1,2,3],dtype='string')
s.str.cat(s2,na_rep='*')
```

<img src="../../ProgramFiles/Typora/upload/image-20200626103610112.png" alt="image-20200626103610112" style="zoom:80%;" />

## 3.替换

**正则表达式的学习**：文档链接

**str.replace正则表达式**

r开头写正则表达式，后面写替换的字符串

```python
s = pd.Series(['A', 'B', 'C', 'Aaba', 'Baca','', np.nan, 'CABA', 'dog', 'cat'],dtype="string")
s.str.replace(r'^[AB]','***')#匹配字符串的开始位置为A和B中的任何一个
```

**子组与函数替换**：

group函数，原文链接：https://blog.csdn.net/jeryjeryjery/article/details/77196497

返回匹配结果中一个或多个group.如果该group函数仅仅有一个参数,那么结果就是单个字符串;如果有多个参数,结果是每一个参数对应的group项的元组.如果没有参数,那么参数group1默认为0(返回的结果就是整个匹配结果).

![image-20200626113401294](../../ProgramFiles/Typora/upload/image-20200626113401294.png)

```python
s.str.replace(r'([ABC])(\w+)',lambda x:x.group(2)[1:]+'*')
```

```python
#利用？<>表达式可以对子组命名调用
s.str.replace(r'(?P<one>[ABC])(?P<two>\w+)',lambda x:x.group('two')[1:]+'*')
```

**str.replace注意事项**

str.replace针对object类型或string类型，默认是以正则表达式为操作，目前暂时不支持DataFrame上使用

replace针对的是任意类型的序列或数据框，如果要以正则表达式替换，需要设置regex=True，该方法通过字典可支持多列替换

**除非需要赋值元素为缺失值（转为object再转回来），否则请使用str.replace方法**

## 4.字符串匹配与提取

**str.extract()函数**

```python
pd.Series(['10-87', '10-88', '10-89'],dtype="string").str.extract(r'([\d]{2})-([\d]{2})')
#使用子组名作为列名
pd.Series(['10-87', '10-88', '-89'],dtype="string").str.extract(r'(?P<name_1>[\d]{2})-(?P<name_2>[\d]{2})')
#利用?正则标记选择部分提取
pd.Series(['10-87', '10-88', '-89'],dtype="string").str.extract(r'(?P<name_1>[\d]{2})?-(?P<name_2>[\d]{2})')
```

**expand参数（默认为True）**
对于一个子组的Series，如果expand设置为False，则返回Series，若大于一个子组，则expand参数无效，全部返回DataFrame
对于一个子组的Index，如果expand设置为False，则返回提取后的Index，若大于一个子组且expand为False，报错

**str.extractall**方法：

extractall会找出所有符合条件的字符串，并建立多级索引（即使只找到一个）

```python
s = pd.Series(["a1a2", "b1", "c1"], index=["A", "B", "C"],dtype="string")
two_groups = '(?P<letter>[a-z])(?P<digit>[0-9])'
s.str.extract(two_groups, expand=True)
s.str.extractall(two_groups)#匹配所有结果
```

**xs方法查看第i层匹配**

```python
s = pd.Series(["a1a2", "b1b2", "c1c2"], index=["A", "B", "C"],dtype="string")
s.str.extractall(two_groups).xs(1,level='match')
```

**str.contains检测是否包含某种正则表达式**(na参数用于填充缺失值)

```python
pd.Series(['1', None, '3a', '3b', '03c'], dtype="string").str.contains(r'[0-9][a-z]')
```

**str.match与其区别在于，match依赖于python的re.match，检测内容为是否从头开始包含该正则模式**

## 5.常用字符串方法

**str.strip()过滤空格**

```python
pd.Series(list('abc'),index=[' space1  ','space2  ','  space3'],dtype="string").index.str.strip()
```

**转换字母大小写str.lower() 和str.upper()**

**str.swapcase()交换字母大小写**

**str.capitalize()大写 首字母**

**isnumeric()检查每一位是否都为数字**

## 6.问题与练习

**问题一： str对象方法和df/Series对象方法有什么区别？**

str.replace针对的是object类型或string类型，默认是以正则表达式为操作，目前暂时不支持DataFrame上使用¶
replace针对的是任意类型的序列或数据框，如果要以正则表达式替换，需要设置regex=True，该方法通过字典可支持多列替换

**【问题二】 给出一列string类型，如何判断单元格是否是数值型数据？**

**【问题三】 rsplit方法的作用是什么？它在什么场合下适用？**

Python rsplit() 方法通过指定分隔符对字符串进行分割并返回一个列表，默认分隔符为所有空字符，包括空格、换行(\n)、制表符(\t)等。类似于 split() 方法，只不过是从字符串最后面开始分割。

**【问题四】 在本章的第二到第四节分别介绍了字符串类型的5类操作，请思考它们各自应用于什么场景？**

拆分：分割单元格

拼接：调整数据格式，将不同单元格连接起来

替换：去除无用字符

匹配：数据筛选，数据过滤

提取：数据筛选

**【练习一】 现有一份关于字符串的数据集，请解决以下问题：**
**（a）现对字符串编码存储人员信息（在编号后添加ID列），使用如下格式：“×××（名字）：×国人，性别×，生于×年×月×日”**

```python
ID = df['姓名']+':'+df['国籍'].astype(str)+'国人，性别'+df['性别']+'，生于'+df['出生年']+'年'+df['出生月']+'月'+df['出生日']+'日'
df.insert(0,'ID',ID) 
```

**（b）将（a）中的人员生日信息部分修改为用中文表示（如一九七四年十月二十三日），其余返回格式不变。**

这里写到一半不会写了，以下为参考答案

**（c）将（b）中的ID列结果拆分为原列表相应的5列，并使用equals检验是否一致。**

```python
dic_year = {i[0]:i[1] for i in zip(list('零一二三四五六七八九'),list('0123456789'))}
dic_two = {i[0]:i[1] for i in zip(list('十一二三四五六七八九'),list('0123456789'))}
dic_one = {'十':'1','二十':'2','三十':'3',None:''}
df_res = df_new['ID'].str.extract(r'(?P<姓名>[a-zA-Z]+):(?P<国籍>[\d])国人，性别(?P<性别>[\w])，生于(?P<出生年>[\w]{4})年(?P<出生月>[\w]+)月(?P<出生日>[\w]+)日')
df_res['出生年'] = df_res['出生年'].str.replace(r'(\w)+',lambda x:''.join([dic_year[x.group(0)[i]] for i in range(4)]))
df_res['出生月'] = df_res['出生月'].str.replace(r'(?P<one>\w?十)?(?P<two>[\w])',lambda x:dic_one[x.group('one')]+dic_two[x.group('two')]).str.replace(r'0','10')
df_res['出生日'] = df_res['出生日'].str.replace(r'(?P<one>\w?十)?(?P<two>[\w])',lambda x:dic_one[x.group('one')]+dic_two[x.group('two')]).str.replace(r'^0','10')
df_res.head()

df_res.equals(df)
```

**【练习二】 现有一份半虚拟的数据集，第一列包含了新型冠状病毒的一些新闻标题，请解决以下问题：**
**（a）选出所有关于北京市和上海市新闻标题的所在行。**

```python
df[df['col1'].str.contains(r'[北京]{2}|[上海]{2}')].head()
```

**（b）求col2的均值。**

直接用mean计算均值会报错，因为里面有不标准的特殊字符

**（c）求col3的均值**

列名问题，特殊字符问题