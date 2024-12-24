# einops学习笔记

- 简化torch.tensor 维度变换

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
  
  - 
