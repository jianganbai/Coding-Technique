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

- 注意
  
  - 使用`h5py`访问时，缓存仅限于`with h5py.File(f, 'r')`的块内。块结束时会清空缓存

### 与pickle对比

- pickle：`np.save`, `torch.save`等采用
  
  - 将数据序列化，整体性写入/读取
  
  - 每个key没有寻址操作
  
  - 整体写入/读取速度比HDF5快，但无法实现增量写入/随机读取，对内存需求高

- HDF5：增量写入/随机读取
  
  - 增量写入：可以将dict 新增的key写入到HDF5中
  
  - 随机读取：可以只读取给定的key
    
    - 每个key都需要单独做寻址，不适合读取大量key
    
    - 如果要全读，更适合用pickle方式保存
    
    - 优化：每个key存大数组，不要数组切细了

## h5py

### 创建

```python
import h5py
import numpy as np

with h5py.File('data.h5', 'w') as f:  # File，可使用'a'进行追加
    f.create_dataset('dataset', data=np.arange(100))  # 在根节点创建Dataset
    s0 = f.create_group('split_0')  # 创建Group，Group是文件夹，Dataset是文件（保存数据）
    s0.create_dataset('image', data=np.arange(50)  # 创建Group下的Dataset
    s0.attrs['description'] = 'Images for split 0'  # 只能保存标量、字符串和小型数组，dict需要做序列化才可保存
```

- `f.create_dataset(name, shape, dtype, chunks, data)`
  
  - `name`：dataset的名字
    
    - 若名字为`a/bb/ccc`，实际存储时为`a['a']['bb']['ccc']`，当然`a['a/bb/ccc']`也可直接读取
  
  - `shape`：dataset的大小，None则为自适应大小
  
  - `dtype`：dataset的数据类型
  
  - `chunks`：分块存储，每块的大小
  
  - `data`：可保存标量、字符串、结构化数组（如`np.ndarray`）
    
    - 不能直接保存 `torch.Tensor`，必须要转化为`np.ndarray`
    
    - 不能直接保存list和dict，需要使用json做序列化`json.dumps(info_dict)`

- 特殊类型
  
  - List[str]
    
    - ```python
      f.create_dataset('path', data=np.array(path_list, dtype='S'))  # 写入：转化为固定长度的ASCII字符
      path_list = [p.decode('utf-8') for p in f['path'][:]]  # 读取：先读取，再转化为utf-8
      ```

### 读取

```python
import h5py

with h5py.File('data.h5', 'r') as f:
    data = f['dataset'][:]  # 立刻完整加载到内存中，不加载其他key对应的dataset
    split_0_img = f['split_0/image'][:]
    description = split_0_img.attrs['description']

with h5py.File('data.h5', 'r') as f:
    dest = f['A']  # 仅获取数据对象，不加载数据
    shape = dest.sshape  # 可在不读取数据之前事先得知数据集大小
    a = dest[0:100]  # 只读取前100行，其它不读入内存
```

- 获取所有dataset 和group
  
  - ```python
    for name in f:
        obj = f[name]
        if isinstance(obj, h5py.Group):
            print(f'Group: {name}')
        elif isinstance(obj, h5py.Dataset):
            print(f'Dataset: {name}')
    ```
