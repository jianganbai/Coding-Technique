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
df.columns = ['name1', 'name2']  # 设置列名
df.index = [1, 0]  # 访问行名 / 设置行名
df.set_index('索引栏名称', inplace=True)  # 设置行名。inplace=True代表原地修改
```

### 访问

- loc使用索引名（显式索引）；iloc使用数字索引（隐式索引）

- loc包含左右两侧端点，iloc仅包含左端点

- 访问行

  - ```python
    df.iloc[index]  # 查看第index行，支持切片。结果通过索引值访问
    df.iloc[24:]  # 查看第24行及之后的全部数据
    df.iloc[:, 8:]  # 切片
    df.iloc[:, 1:].to_numpy()  # 将除表头外转化为numpy
    df.iloc[-1].to_dict()  # 将最后一行转化为dict
    ```
    
  - ```python
    df[df['列名'] == '值']  # 特定行
    df.loc[df['column_name'] == some_value]
    df.loc[df['column_name'].isin(some_values)]  # 可有多个不同取值
    df.loc[(df['column_name'] >= A) & (df['column_name'] <= B)]
    
    # 筛选特定列等于特定值
    require = {'列名': [值1, 值2, 值3]}
    bools = [df[k] in v for k, v in require.items()]
    df[np.all(bools, axis=0)]
    
    # 筛选特定列非空
    df.loc[df['xxx'].notnull() | (df['xxx'] == '')]
    ```
    
  - ```python
    # 创建示例DataFrame
    df = pd.DataFrame({'A': [1, 2, 3], 'B': ['a', 'b', 'c']})
     
    for index, row in df.iterrows():
        print(f"Index: {index}")  # DataFrame的行标签
        print("Data:")
        for column_name, value in row.items():  # 也可通过row['A'], row['B']访问
            print(f"\t{column_name}: {value}")
        print("\n")
    ```

- 访问列

  - ```python
    df.columns  # 查看列名
    df['列名']  # 查看某一列
    df[['列1', '列2']]  # 查看多列
    ```

- apply：对元素逐一进行变换（单进程）

  - 可先筛选出特定行/列
  
  - ```python
    df = pd.DataFrame({'name': ['a', 'b'], 'age': [40, 50], 'salary': [10, 29]})
    df['age'].apply(lambda x: x + 10)  # df['age']是Series，apply()输出Series，不需要指定axis
    df[['age', 'salary']].apply(np.sum, axis=0)
    # axis=0为column-wise，axis=1为row-wise
    df.apply(lambda x: x['age'] + 10, axis=1)  # 提取年龄，然后加10岁
    # 若apply func返回pd.Series，则.apply()会将它们拼成新dataframe
    ```
    
  - ```python
    # pandarallel库可实现多进程apply
    pandarallel.initialize()
    df.parallel_apply(func, axis=1)  # API与原版apply一致
    ```

### 添加

- 添加列：添加新属性

  - ```python
    df = pd.DataFrame({'name': ['a', 'b', 'c', 'd', 'e']})
    # 原地操作
    df['score'] = [1,2,3,4,5]  # 总长度等于已有样本长度
    # 返回新dataframe，不修改原来的
    df_new = df.assign(new_key=lambda x: x['score'] +1)
    ```

- 添加行：添加新样本

  - ```python
    df.loc[index] = ['f', 6]  # 若该index已存在，则修改；否则添加在最后
    df.loc[len(df)] = ['f', 6]  # 在最后添加
    ```

  - ```python
    df.iloc[index] = ['f', 6]  # 修改已有index
    ```
    
  
- 拼接DataFrame

  - ```python
    # 上下拼接, axis=0为行，ignore_index为忽略左侧index
    df = pd.concat([data_a, data_b], axis=0, ignore_index=True)
    # axis=1代表拼接列，可通过keys指定各列新名称
    df = pd.concat([df1['a'], df2['b']], axis=1, keys=['c', 'd'])
    ```

- 若要插入多行，则先存在list中，每个元素为`pd.DataFrame`，最后一次性拼接

  - ```python
    info_list = []
    for a in range(10):
        info_list.append(pd.DataFrame({'a': a, 'a^2': a * a}))
    df = pd.concat(info_list, axis=0)  # axis=0为行，axis=1为列
    ```

  - 一行一行地写，复杂度太高

### 修改

- loc

  - ```python
    df.loc[df['name'] == 'a', 'score'] = 100  # 不能使用df.loc[i]['score'] = 100
    # df.loc[条件1，条件2]，条件可以为切片(:)，值，行索引/列标签名
    
    df.at[row_index, 'column_name'] = 100  # .at()只能设定单个值，.loc()和.iloc()可设定多个值
    ```

- 修改顺序

  - ```python
    df.reindex(columns=['new_index1', 'new_index2', 'new_index3'])
    ```

- 删除

  - ```python
    df.drop('column_name', axis=1, inplace=True)  # axis=0为行，axis=1为列。inplace表示原地操作
    ```

- 排序

  - ```python
    # 按指定列的值排序
    df.sort_values(['a', 'b'], ascending=[True, False], inplace=True)  # 先按a列升序排，再按b列降序排
    # 按每一行的index排序（适合指定了index的df）
    df = pd.DataFrame([1, 2, 3, 4, 5], index=[100, 29, 234, 1, 150], columns=['A'])
    df.sort_index()
    ```

- **replace**：将所有指定值，替换为另一指定值

  - ```python
    # DataFrame.replace(to_replace, replace), to_replace是原表的指定值，replace是替换后的值
    df.replace(0, 5)  # 将表内所有的0，替换为5
    df.replace(to_replace={'a': 'b', 'c': 'd'}, replace=None)  # 将所有a替换为b，将所有c替换为d
    df.replace(to_replace={'a': 1, 'b': 2}, replace=[3, 4])  # 将a列的1替换为3，将b列的2替换为4
    ```

- 改变索引

  - `df.reset_index()`：将索引列改为正常的一列，适合groupby操作
  
- 筛选非空列

  - ```python
    # .copy()防止原位修改时有冲突
    df = df[df['a'].notnull()].copy()  # 筛选出a列非空的行，若空则该位置为nan
    df = df[df['a'].isnull()].copy()  # 筛选出a列为空的行
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
df.groupby(lambda x: '聪明' if x % 2 else '笨笨').mean()
# mean, median, sum, min, max, std, var, count

# 分别给出语文和数学的统计结果
df.groupby(['班级', '姓名']).agg({'语文': [np.mean, min], '数学': [np.mean, max]})
```

```python
# DataFrameGroupBy是可迭代的
group = df.groupby('班级', as_index=True, sort=True)
for class_name, class_info in group:
    pass
```

- 返回DataFrameGroupBy对象
  - `list(DataFrameGroupBy)`：`[(分组名1， 子DataFrame1), (分组名2， 子DataFrame2), ...]`
    - 子DataFrame保留原来的行index
  - `DataFrameGroupBy.groups`：所有分组名

### 输出

- csv: `df.to_csv(csv_name, sep=',', header=True, index=True)`
  - `header=True`保留每列名字，`index=True`保留每行名字
  - csv：ascii，逗号分隔。excel：二进制，分号分隔

### 其它

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

- 递归访问

  - ```python
    for index, value in series.items():
        xxx
    ```

  - 

