# einops和einsum学习笔记

## einops

- 简化torch.tensor 维度变换，需要单独装库

- 维度变换
  
  - ```python
    import torch
    from einops import rearrange
    
    x = torch.randn(2, 3, 4)  # A tensor of shape (2, 3, 4)
    y = rearrange(x, 'b h w -> b (h w)')  # Reshape to (2, 12), preserve order
    z = rearrange(x, 'b (h1 h2) w -> b h1 h2 w', h1=3)  # Reshape to (2, 3, 4)
    ```

- 聚合
  
  - ```python
    import torch
    from einops import reduce
    
    x = torch.randn(2, 3, 4)  # A tensor of shape (2, 3, 4)
    mean_value = reduce(x, 'b h w -> b', 'mean')  # Average over h and w, shape (2,)
    ```

- 封装在pytorch 网络中
  
  - ```python
    import torch.nn as nn
    from einops.layers.torch import Rearrange
    model = nn.Sequential(
        nn.Conv2d(3, 6 kernel_size=5),
        nn.ReLU(),
        Rearrange('b c h w -> b (c h w)')
    )
    ```

## einsum

- 爱因斯坦求和，可直接使用`torch.einsum`
  
  - 例：矩阵乘法可表示为`torch.einsum('ik,kj->ij', [a,b])`

- 理论
  
  - 2种索引
    
    - 求和索引：只出现在箭头左侧的索引
    
    - 自由索引：出现在箭头右侧的索引
  
  - 运算规则
    
    - 若求和索引重复出现，则把2个张量在该维度相乘
      
      - 等价于先把其他维度拉平，然后在该维度按元素相乘，再复原回原来的值
    
    - 若求和索引单独出现，则在张量内部求和
    
    - 自由索引的前后顺序可调，对应的是转置

```python
import torch

# 单元素
torch.einsum('ij->ji', [a])  # 矩阵转置
torch.einsum('ii->i', [a])  # 提取矩阵对角线元素
torch.einsum('ij->j', [a])  # 矩阵按列求和
torch.einsum('...ij->...ji', [a])  # 张量转置，...表示之前所有维度


# 双元素
torch.einsum('ik,k->i', [a, b])  # 矩阵向量乘法
torch.einsum('i,j->ij', [a, b])  # 向量外积
torch.einsum('ijk,ikl->ijl', [a, b])  # 等价于torch.bmm
```
