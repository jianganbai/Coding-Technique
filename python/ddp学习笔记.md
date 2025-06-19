# DDP学习笔记

- DDP: Distributed Data Parallel

## 概念

- 专业术语
  
  - `rank`：全局进程编号，取值于[0, world_size - 1]。rank=0的为主进程
  - `local_rank`：进程在所处物理机的编号
  - `group_rank`：当前物理机在所有物理机中的编号
  - `world_size`：全局进程总数
  - `local_world_size`：一台物理机上参与训练的进程数
  - `node`：一台物理机
  - `master address`：主节点（rank=0的机器）的IP或主机名
  - `master port`：主节点监听的端口号
  - `process group`：参与DDP训练的所有进程的集合
  - `backend`：通信后端
    - `gloo`：兼容CPU / GPU，仅适合较小规模的数据集
    - `nccl`：只能用于GPU，对通信方法做了优化，最常用的后端

- 训练过程
  
  - 每个进程对应1个GPU，分别构建网络、数据集
  
  - sampler将数据集切开，每个进程分别进行前向
  
  - 所有GPU通过Ring-Reduce机制同步梯度，边计算梯度边传梯度
    
    - Ring-Reduce：所有GPU串成1个环，每个GPU只与环路中下一个GPU通信，传完一圈就同步了梯度

## 核心代码

### 新加入

- ```python
  import torch.distributed as dist
  from torch.utils.data import DistributedSampler
  
  parser.add_argument('--local_rank', type=int)  # 启动器会创建多个进程，每个进程通过local_rank获得进程号
  # torch.cuda.set_device(args.local_rank)
  device = torch.device('cuda', local_rank)  # 之后可使用to(device)
  
  # 在创建网络前初始化
  # nccl适合GPU，gloo适合CPU
  dist.init_process_group(backend='nccl', init_method='env://')
  
  # 每个进程单独构建数据集，内存空间不共享。但num_workers > 1时不同worker采用fork方法生成，会复用主进程的内存空间（有修改时才会复制）
  train_sampler = DistributedSampler(data)  # data是例化后的Dataset
  # 再去掉DataLoader中的shuffle=True，改为sampler=train_sampler，保证不同进程的数据不同
  # DataLoader的batch_size为每个进程的batch_size
  
  # 将BN设置为进程间同步参数
  net = torch.nn.SyncBatchNorm.convert_sync_batchnorm(net).to(device)
  net = torch.nn.parallel.DistributedDataParallel(
      net,
      device_ids=[local_rank],
      output_device=local_rank,
      find_unused_parameters=True  # False则为静态图，要求所有参数都参与前向且网络forward函数的所有输出都参与loss计算
  )  # 使用DDP接口包装
  
  train_loader.sampler.set_epoch(i)  # 每个epoch计算前
  # 各子进程的DistributedSampler以epoch为种子，保证各机器shuffle出的顺序都相同
  
  # 每个epoch内必须1次forward + 1次backward，多次for/back需要合在一起（如GAN）
  
  dist.barrier()  # 用于进程间同步，仅当全部进程都被dist.barrier()阻塞后才会释放
  
  # 保存模型应保存model.module
  ```

- 训练时必须1次forward+1次backward，对于GAN则需要将真样本和生成样本级联在一起送入D（2次forward合为1）

- `DistributedSampler`
  
  - 给出各个进程从dataset取样本的下标，属于`sampler`而非`batch_samplerr`
  
  - 各个进程交替取,：3个进程，进程0取0, 3, 6，进程1取1, 4, 7，进程2取2, 5, 8

### 启动

- ```shell
  CUDA_VISIBLE_DEVICES=4,5,6,7 python -m torch.distributed.launch --nproc_per_node 4 train.py  # 旧版，将废弃
  
  CUDA_VISIBLE_DEVICES=4,5,6,7 torchrun --nproc_per_node 4 -m runner.im.im  # 新版 
  ```

- `torchrun`参数
  
  - 核心参数
    
    - `--nproc_per_node`：每个节点启动的进程数
    
    - `--master_port`：通信端口，默认为29500
      
      - 单机训练也需要专门的端口。同物理机若要运行第2个DDP任务，则必须更换端口
    
    - 多机需设置
      
      - `--nnodes`：物理机个数
      
      - `--rdzv_id`：rendezvous id，分布式训练的全局唯一标识符，所有节点必须使用相同的ID
      
      - `--rdzv_backend`：进程组通信后端，默认为`c10d`，一般不需调整
      
      - `--rdzv_endpoint`：主节点的地址和端口，例如`--rdzv_endpoint=192.168.1.1.:29500`
        
        - 等价于旧版用的`--master_addr`和`--master_port`
  
  - 辅助参数
    
    - `--max_restarts`：进程出错后最大的重启次数，默认为3
    
    - `--local_addr`：当前节点的IP地址（多网卡时需显式指定）
  
  - 调试
    
    - `--log_dir`：torch日志保存目录，每个进程会生成独立日志文件

- `torch.distributed.launch`和`torchrun`对比
  
  - `torchrun`自动为每个进程分配`rank`和`world_size`，无需通过命令行参数传递

## 同步

- 仅rank 0 进程才能操作的：所有写log，写tensorboard，操作tqdm，保存模型

- `dist.barrier()`：仅当全部进程都被dist.barrier()阻塞后才会释放

## 通信

- 6种通信方法
  
  - scatter, reduce, gather
  
  - all_reduce, all_gather, broadcast
  
  - <img title="" src="imgs/2024-11-27-16-19-38-image.png" alt="" width="623">

- 要求
  
  - 若是张量，只能同步同名同大小
  
  - 若采用nccl后端，只能同步同名同大小且在GPU的张量，不能调用`dist.gather_object`
  
  - 张量必须存放在连续的存储空间中，使用`tensor.contiguous()`将其转为连续存储

### reduce

```python
# reduce
tensor = torch.tensor([dist.get_rank() + 1], device='cuda')  # 所有进程都要有同名同尺寸的tensor变量
dist.reduce(tensor, dst=0, op=dist.ReduceOp.SUM)  # 求和，结果发送给rank=0的tensor变量上
# 其它聚合操作：ReduceOp.PRODUCT, ReduceOp.MIN, ReduceOp.MAX, ReduceOp.AVG

# all_reduce
tensor = torch.tensor([dist.get_rank() + 1], device='cuda')
dist.all_reduce(tensor, op=dist.ReduceOp.SUM)  # 求和，把结果广播到所有进程的tensor变量
```

### gather

- `dist.gather(tensor, gather_list=None, dst=0)`
  
  - 将所有进程的tensor，汇总到`rank dst`的`gather_list`
    
    - tensor尺寸固定，梯度不可反传
    - 汇总的列表按照rank从小往大排列
  
  - ```python
    tensor = torch.tensor([rank + 1], device='cuda')  # rank0:[1], rank1:[2], ...
    
    if rank == 0:
        gather_list = [torch.zeros_like(tensor) for _ in range(world_size)]     # 主进程预分配接收列表，只能在主进程上创建
        dist.gather(tensor, gather_list=gather_list, dst=0)
        print(f"Rank 0 gathered: {[t.item() for t in gather_list]}")  # 输出: [1, 2, 3, ...]
    else:  # rank0 不走这条
        dist.gather(tensor, dst=0)  # 其他进程只需传入自己的tensor
    ```

- `dist.all_gather(tensor_list, tensor)`
  
  - 将所有进程的tensor，汇集为`gather_list`，再分发给所有进程
    
    - tensor尺寸可变，梯度可反传
    - 汇总的列表按照rank从小往大排列
  
  - ```python
    # 所有进程预分配接收列表
    tensor_list = [torch.zeros_like(tensor) for _ in range(world_size)]
    dist.all_gather(tensor_list, tensor)
    # reduce类保存到同步前的变量，gather类需要预分配接受列表 
    ```

- `dist.gather_object(obj, obj_gather_list, dst=0)`
  
  - 聚合所有可使用pickle序列化的变量，不适合`torch.Tensor`
  
  - 仅gloo后端能调用，nccl后端不能调用。使用方法同`dist.gather()`

- `dist.all_gather_object(obj_list, obj)`
  
  - 聚合所有可使用pickle序列化的变量，不适合`torch.Tensor`
  
  - 仅gloo后端能调用，nccl后端不能调用。使用方法同`dist.gather()`
