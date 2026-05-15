# deepspeed 学习笔记

## 训练

### 代码

```python
import deepspeed
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset

# ... model, data, criterion definition ...

# 1. Initialize DeepSpeed (usually handled by deepspeed.init_distributed() or deepspeed.launch)
#    This is typically done via a deepspeed launch command or a script that calls deepspeed.init_distributed()

# 2. Define your model, optimizer, and (optionally) learning rate scheduler
model = MyModel()
optimizer = optim.Adam(model.parameters())
criterion = nn.CrossEntropyLoss()

# 3. Initialize DeepSpeed engine
#    ds_config is a dictionary or a path to a JSON configuration file
#    It defines ZeRO stage, mixed precision, and other optimizations
ds_config = {
    "train_batch_size": 16,
    "gradient_accumulation_steps": 1,
    "optimizer": {
        "type": "Adam",
        "params": {
            "lr": 1e-5
        }
    },
    "fp16": {
        "enabled": True
    },
    "zero_optimization": {
        "stage": 3 # ZeRO-3 is often used for very large models
    }
}

model_engine, optimizer, _, _ = deepspeed.initialize(
    model=model,
    model_parameters=model.parameters(),
    optimizer=optimizer,
    config_params=ds_config
)

for epoch in range(num_epochs):
    for batch_idx, (data, target) in enumerate(dataloader):
        # Move data to GPU handled by DeepSpeed's engine
        data = data.to(model_engine.local_rank)
        target = target.to(model_engine.local_rank)

        optimizer.zero_grad()
        output = model_engine(data)
        loss = criterion(output, target)
        model_engine.backward(loss) # DeepSpeed handles gradient reduction across GPUs
        model_engine.step() # DeepSpeed handles optimizer step and parameter updates

        # For checkpointing, etc., DeepSpeed has its own functions
        # model_engine.save_checkpoint(...)
```

### 配置

```python
{
  "train_batch_size": 16,
  "gradient_accumulation_steps": 1,
  "optimizer": {
    "type": "Adam",
    "params": {
      "lr": 1e-5,
      "betas": [0.9, 0.999],
      "eps": 1e-8
    }
  },
  "scheduler": {
    "type": "WarmupLR",
    "params": {
      "warmup_min_lr": 0,
      "warmup_max_lr": 1e-5,
      "warmup_num_steps": 1000
    }
  },
  "fp16": {
    "enabled": true,
    "loss_scale": 0,
    "initial_scale_power": 16,
    "loss_scale_window": 1000,
    "hysteresis": 2,
    "min_loss_scale": 1
  },
  "zero_optimization": {
    "stage": 2,
    "offload_optimizer": {
      "device": "cpu",
      "pin_memory": true
    },
    "overlap_comm": true,
    "contiguous_gradients": true,
    "sub_group_size": 1e9,
    "reduce_bucket_size": 5e8,
    "stage3_prefetch_bucket_size": 5e8,
    "stage3_param_persistence_threshold": 1e4,
    "stage3_max_live_parameters": 1e9,
    "stage3_max_reuse_distance": 1e9,
    "stage3_gather_fp16_weights_on_model_save": true
  },
  "gradient_clipping": 1.0,
  "flops_profiler": {
    "enabled": false,
    "profile_step": 1,
    "module_depth": -1,
    "top_modules": 1,
    "detailed": true,
    "output_file": null
  },
  "elasticity": {
    "enabled": false,
    "max_train_batch_size": 16,
    "micro_batch_size": 1
  }
}
```

### ZeRO

- ZeRO：零冗余优化器
  
  - 将优化器的states, gradients和parameters 分割到多个GPU上
    
    - parameters：神经网络权重
    
    - gradients：权重梯度
    
    - states：动量、每个权重的学习率、滑动平均等
  
  - 使得能训练会爆显存的模型

- ZeRO-1：分割states，不分割gradients和parameters

- ZeRO-2：分割states和gradients，不分割parameters

- ZeRO-3：分割states, gradients 和parameters
  
  - 用于非常大的模型，但通信负担大

## 推理

```python
import deepspeed
model = mymodel(**cfg)
ds_engine = deepspeed.init_inference(
    model=model,
    tensor_parallel={'tp_size': 1},  # 多GPU张量并行
    dtype=torch.float16,
    replace_with_kernel_inject=True,  # 使用DeepSpeed优化后的模块替换
)
model = ds_engine.module

for data in data_loader:
    data = data.to(torch.float16).cuda()  # 需要手动将输入转化为float16
    output = model(data)  # 不再需要torch autocast
```

- 3D并行
  
  - **数据并行**：每个GPU都有完整模型，分别处理不同数据
    
    - 单卡能完整容纳模型。通信占比低
    
    - DP, DDP
  
  - **张量并行**：将每个层权重切分到多个GPU，各个GPU协同计算同一个层，通过通信合并中间结果
    
    - 单卡无法容纳模型，GPU间通信要求高，但延迟比流水线并行低
    
    - 常用于大模型推理
      
      - 不适合训练，因为通信开销翻倍
    
    - DeepSpeed
  
  - **流水线并行**：将模型分到不同的GPU
    
    - 单卡无法容纳模型，通信带宽比张量并行低
    
    - 常用于大模型训练
      
      - 不适合推理，因为延迟巨大
