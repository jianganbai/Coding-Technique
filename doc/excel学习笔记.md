# excel学习笔记

## 函数

### STD

- 分类
  
  - sample or population
    
    - sample：已知采样结果，估计分布的标准差
      
      - 使用N-1 取平均，保证无偏差
      
      - 结果称为statistics
    
    - population：已知整个数据集，计算数据集的标准差
      
      - 使用N 取平均
      
      - 结果称为parameters
  
  - numeric or non-numeric
    
    - numeric：文本当做0，bool当做0/1
    
    - non-numeric：忽略所有非数字

- 函数
  
  - | FUNC    | Type       | Non-numeric | Moden Replacement |
    |:-------:|:----------:|:-----------:|:-----------------:|
    | STDEV.S | sample     | no          | -                 |
    | STDEVA  | sample     | yes         | -                 |
    | STDEV.P | population | no          | -                 |
    | STDEV   | sample     | no          | STDEV.S           |
    | STDEVPA | sample     | yes         | STDEVA            |
    | STDEVP  | population | no          | STDEV.P           |
  
  - STDEV.S vs STDEVA
    
    - 差别仅在如何应对非数值
    
    - 估计方法相同

- 与其它
  
  - ```python
    np.std(x)  # population level
    np.std(x, ddof=1)  # sample level
    ```
