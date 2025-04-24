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
