# h5py学习笔记

## HDF5

- 结构：树状结构
  
  - **文件（File）**：HDF5文件是数据的顶级容器
  
  - **组（Group）**：用于组织数据，类似于文件夹，相当于中间节点
  
  - **数据集（Dataset）**：存储实际数据的多维数组，相当于叶子节点
  
  - **属性（Attribute）**：附加到文件、组或数据集的元数据

- 功能
  
  - 高效存储大规模数值
  
  - 支持数据压缩和分块存储
  
  - 支持并行读写

### 缓存机制

- HDF5 格式采用多种缓存机制，提升数据读写性能

- 读缓存
  
  - 元数据直接缓存在内存中，减小磁盘I/O
  
  - 数据全部分chunk 存储，chunk是最小的I/O单元
    
    - 存储时有压缩
    - `a = f['data']`是指针，仅在调用时（`a = f['data'][:]`的`[:]`）才读入内存
  
  - 最近访问的chunk缓存在内存中
    
    - 每个进程单独维护自己的chunk缓存，进程之间不共享
    
    - 随机访问需要更大缓存

- 写缓存
  
  - 多次小写合并在一起，延迟写入

## h5py

### 创建

```python
import h5py
import numpy as np

with h5py.File('data.h5', 'w') as f:  # File
    f.create_dataset('dataset', data=np.arange(100))  # 在根节点创建Dataset
    s0 = f.create_group('split_0')  # 创建Group
    s0.create_dataset('image', data=np.arange(50)  # 创建Group下的dataset
    s0.attrs['description'] = 'Images for split 0'
```

- `f.create_dataset(name, shape, dtype, chunks)`
  
  - `name`：dataset的名字
    
    - 若名字为`a/bb/ccc`，实际存储时为`a['a']['bb']['ccc'`，当然`a['a/bb/ccc']`也可直接读取
  
  - `shape`：dataset的大小，None则为自适应大小
  
  - `dtype`：dataset的数据类型
  
  - `chunks`：分块存储，每块的大小

### 读取

```python
import h5py

with h5py.File('data.h5', 'r') as f:
    data = f['dataset'][:]
    split_0_img = f['split_0/image'][:]
    description = split_0_img.attrs['description']
```
