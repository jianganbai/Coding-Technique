# pandas学习笔记

## Dataframe操作

### 构建

```python
df = pd.DataFrame({'班级': [1, 2, 3],
                   '姓名': ['A', 'B', 'C'],
                   '成绩': [80, 90, 100]})  # dict的每个key都是一列
# 一般不同列为不同属性，不同行为不同样本
df = pd.DataFrame({'班级': [], '姓名': []})  # 空DataFrame
```

### 读取csv

```python
import pandas as pd

df = pd.read_csv(path, sep, header, index_col)
# sep指定哪个是分隔符
# header指定哪行是表头，header=None表示没有表头
# index_col指定哪列是行索引，index_col=False表示没有行索引
df.columns = ['name1', 'name2']  # 设置表头
```

### 访问

- loc使用索引名（显式索引）；iloc使用数字索引（隐式索引）

- 访问行

  - ```python
    df.iloc[index]  # 查看第index行，支持切片
    df.iloc[24:]  # 查看第24行及之后的全部数据
    df.iloc[:, 8:]  # 切片
    df.iloc[:, 1:].to_numpy()  # 将除表头外转化为numpy
    ```

  - ```python
    df[df['列名'] == '值']  # 特定行
    df.loc[df['column_name'] == some_value]
    df.loc[df['column_name'].isin(some_values)]
    df.loc[(df['column_name'] >= A) & (df['column_name'] <= B)]
    
    # 筛选特定列等于特定值
    require = {'列名': [值1, 值2, 值3]}
    bools = [df[k] in v for k, v in require.items()]
    df[np.all(bools, axis=0)]
    ```

- 访问列

  - ```python
    df.columns  # 查看列名
    df['列名']  # 查看某一列
    df[['列1', '列2']]  # 查看多列
    ```

- apply：对元素逐一进行变换

  - 可先筛选出特定行/列
  
  - ```python
    df = pd.DataFrame({'name': ['a', 'b'], 'age': [40, 50], 'salary': [10, 29]})
    df[['age', 'salary']].apply(np.sum, axis=0)
    # axis=0为column-wise，axis=1为row-wise
    ```

### 添加

- 添加列：添加新属性

  - ```python
    df = pd.DataFrame({'name': ['a', 'b', 'c', 'd', 'e']})
    df['score'] = [1,2,3,4,5]  # 总长度等于已有样本长度
    ```

- 添加行：添加新样本

  - ```python
    df.loc[index] = ['f', 6]  # 若该index已存在，则修改；否则添加在最后
    df.loc[len(df)] = ['f', 6]  # 在最后添加
    ```

  - ```python
    df.iloc[index] = ['f', 6]  # 修改已有index
    ```

  - ```python
    df.append({'name': 'f', 'score': 6}, ignore_index=True)
    ```

- 拼接DataFrame

  - ```python
    # 上下拼接, axis=0为行，ignore_index为忽略左侧index
    df = pd.concat([data_a, data_b], axis=0, ignore_index=False)
    ```

  - ```python
    # append之后将被舍弃，转为concat
    a = pd.DataFrame({'name': ['f'], 'score': [6]})
    df.append(a, ignore_index=True)  # append一般用于将一DataFrame添加到另一DataFrame之后
    ```

### 修改

- loc

  - ```python
    df.loc[df['name'] == 'a', 'score'] = 100  # df.loc[条件1，条件2]，条件可以为切片(:)，值，行索引/列标签名
    ```

- 修改顺序

  - ```python
    df.reindex(columns=['new_index1', 'new_index2', 'new_index3'])
    ```

- 删除

  - ```python
    df.drop('column_name', axis=1, inplace=True)  # axis=0为行，axis=1为列。inplace表示原地操作
    ```

### groupby

```python
groupby(by, axis, as_index, sort)
# by: 按什么分组
# axis: 0则按行切分（默认），1则按列切分
# as_index: 是否将分组值作为索引
# sort: 是否按分组值大小排序
```

```python
# 按某一列的值分组
df.groupby('班级', as_index=False).mean()

# 按某一列的值的分析结果分组
df.groupby(df['姓名'].str[0]).mean()
df.groupby({0: '聪明', 1: '笨笨'}).mean()  # 默认先将第0列送入字典中查询
df.groupby(lambda x: '聪明' if x%2 else '笨笨').mean()
# mean, median, sum, min, max, std, var, count
```

```python
# 分别给出语文和数学的统计结果
df.groupby(['班级', '姓名']).agg({'语文': [np.mean, min], '数学': [np.mean, max]})
```

### 输出

- csv: `df.to_csv(csv_name, sep=',', header=True, index=True)`
  - `header=True`保留每列名字，`index=True`保留每行名字
  - csv：ascii，逗号分隔。excel：二进制，分号分隔

### 其它

- 设置索引栏

  - ```python
    df.set_index('索引栏名称', inplace=True)  # inplace=True代表原地修改
    ```

## Series操作

- Series：一维数组
  - 相比于numpy：可自定义索引
  - 相比于字典：速度更快、可使用切片

### 构建

```python
a = pd.Series([1, 2, 3])
a = pd.Series([1, 2, 3], index=['a', 'b', 'c'])  # 自定义索引
a = pd.Series({'a': 1, 'b': 2, 'c': 3})
```

### 访问

- 按索引值访问

  - ```python
    a = pd.Series({'a': 1, 'b': 2, 'c': 3})
    b = pd.Series([1, 2, 3])
    a['a'] == 1  # 按索引值
    assert b[1] = 2  # 按索引值  
    ```

- 切片运算

  - ```python
    a = pd.Series({'a': 1, 'b': 2, 'c': 3, 'd': 4, 'e': 5})
    a['a': 'c']  # 截取'a'到'c'（含）的元素
    a[0: 2]  # 截取前2个元素（不含右端）
    
    a.loc['a': 'c']  # loc使用索引名（显式索引）
    a.iloc[0: 2]  # iloc使用数字索引（隐式索引）
    ```



