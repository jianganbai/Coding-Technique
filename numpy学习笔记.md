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

## 小技巧积累

- `numpy.reshape(origin, newshape, order='C')`
  - `newshape`有多少项，reshape结果就有多少维
    - `newshape`可为int可为tuple，为int代表reshape成1维
  - `newshape`允许1项为-1，表示该维有多少项最后确定（根据张量元素个数相等确定）
    - 若`newshape=-1`，代表直接展开成1维向量
  - 调整大小的逻辑：先拉平成一维，再调整为预期大小
    - 拉平时从左侧维度到右侧维度逐一填补
- `numpy.transpose(newdim0, newdim1, newdim2, ...)`
  - 交换数组某些维
    - 本质上是改变同一块内存的索引方式（slice方式）
    - 但实际上是开一块新内存来存储旧数据，再改变读取方式
  - `newdim0`，`newdim1`等应在想调的位置输入原来的维度编号
    - 例：矩阵转置`A.transpose(1, 0)`
    - `np.shape`返回的维度元组中，从左至右分别从0,1,2，...开始编号
- `numpy.pad(origin, size, 'constant', constant_values)`
  - 常用于卷积层的padding
  - `size`为n个2元元组组合成的元组，n为`origin`的维度
    - 2元元组表示该维度左右两侧各填充的个数
  - `constant_values`为被填入的值
- `numpy.insert(origin, obj, values, axis)`
  - 在张量的某个维度特定位置插入一列/面/超平面
  - `obj`为在`axis`维度下插入的位置，原数据后移
  - `axis`为张量维度
    - 如`x.shape=(b,c,h,w)`，则`b`属于维度0，`c`属于维度1
- `numpy.sum(a, axis)`
  - 对张量`a`的`axis`维度的所有数求和
  - 最终结果减小1个维度













