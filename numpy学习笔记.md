# numpy学习笔记

- numpy是数学函数库，支持矩阵运算与高维度数组
- 使得可在python中进行类似于Matlab的数据处理
  - pytorch对应的库是tensor

## Ndarray对象

- Ndarray == N维数组，但数据类型不限于double
- `numpy.array(object, dtype = None, copy = True, order = None, subok = False, ndmin = 0)`
  - object: 用于创建Ndarray的数据
  - dtype: 数组元素的数据类型，例：complex
  - copy: 创建时是否拷贝
  - order: 创建数组的方向，C为行方向，F为列方向，A为任意方向
  - subok: 返回一个与基类类型一致的数组
  - ndmin: 指定生成数组的最小维度