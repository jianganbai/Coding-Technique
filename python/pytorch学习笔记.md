# Pytorch笔记

## Data

### Dataset & DataLoader

- `torch.utils.data.ConcatDataset([Dataset1, Dataset2])`：将多个数据集拼接起来
  - 若拼接的数据集调用方法不同（如半监督数据集），则需要重写ConcatDataset方法
- DataLoader
  - batch_size：None表示Dataset打batch（拼成torch.tensor并pad）
  - `num_workers`：额外开多少个子进程加载batch，=0则主进程用于加载，>0则主进程不加载，在RAM中找子进程加载好的batch

### Sampler

- Dataset：读入所有数据，每个下标对应1个样本

- Sampler: 给出所有用于训练的数据的下标，默认是[0, 1, 2, 3, ...]

- BatchSampler: 从Sampler给出的下标中，选出要合成1个batch的下标

  - ```python
    class BatchSampler(Sampler):
        def __init__():
        def __len__():
        def __iter__():
    ```

- DistributedSampler：DDP用的Sampler

  - 每个epoch执行1次sample，把数据切分给不同GPU

  - ```python
    class DistributedSampler:
        def __init_(self, shuffle, **):
            xxx
        def set_epoch(self, epoch):  # 每个epoch执行前需要重新设置，否则每轮batch里的数据都一样
            self.epoch = epoch
        def sample(self, data):
            random.Random(self.epoch).shuffle(data)  # Random()创建1个实例，seed为self.epoch，对data做shuffle
            data = data[self.rank::self.world_size]  # 切分出本机对应的数据
            data = data[self.worker_id::self.num_workers]  # 切分出Dataloader多进程加载的数据
            return data
    ```

  - 

## 张量

- `tensor.squeeze()`与`tensor.unsqueeze()`
  - `tensor.squeeze()`：去除所有大小为1的维度
  - `tensor.squeeze(arg)`：若第arg维的维度值为1，则去除之，否则保持原tensor不变
  - `tensor.unsqueeze(arg)`：在第arg维插入维度值为1
  - `a = a[None, :, :] `：None代表在此位置插入维度
  
- Variable与Parameter

  - Variable: 所有输入网络的tensor都会被自动包装为Variable
    - 默认无梯度
  - Parameter: 自动将tensor注册到网络中，默认有梯度，可进行反向创博
  - **仅需要被更新的tensor才需要梯度**
    - 如果常量只是加入了计算图，则不需要梯度

## 梯度

- `zero_grad()`
  - `net.zero_grad()`：将网络所有需要梯度的参数的梯度置零
  - `optim.zero_grad()`将optim优化的所有参数的梯度置零
  - 若optim的参数仅指定为某网络，则`net.zero_grad()`等同于`optim.zero_grad()`
- `loss.backward()`：计算损失函数关于各个参数（网络参数等，不含输入）的梯度
  - loss必须是标量
  - 每调用1次backward，会为所有需要梯度的参数计算梯度，然后累加到该参数的已有梯度上
  - 所以每个batch都需要调用`.zerograd`
- `optimizer.step()`：在给定梯度时有不同梯度下降策略
  - 每个样本都能给出一个梯度
  - 该选择哪些样本的梯度？是否要考虑历史梯度？沿梯度方向下降的学习率如何变化？这些都由optimizer决定
  - **应在将网络移动到cuda后，再构建optimizer**
- `net.eval`与`torch.no_grad`
  - `net.eval()`用于设定dropout层和BN层，但计算过程中仍计算梯度
    - `net.train()`自动设置`self.training=True`，`net.test()`自动设置`self.training=False`
  - `torch.no_grad`用于取消计算梯度
  - `torch.inference_mode`：与`torch.no_grad`基本相同，更推荐
  - 在验证/测试时，最好都写上

## 网络

- 所有网络模块均需继承nn.Module

### 定义方式

- `nn.Sequential`：多个模块级联在一起

  - 指定好forward方式

  - 可使用OrderedDIct同时给出模块和模块名称

  - ```python
    layers = []
    layers.append(nn.Linear)  # 可先使用list包括所有模块
    self.net = nn.Sequential(*layers)  # 最后再转化为nn.Sequential
    ```

- `nn.ModuleList`: 使用list组合多个模块，并将它们的参数注册到网络中

  - ```python
    # 需手动指定forward方式
    class net_modlist(nn.Module):
        def __init__(self):
            super(net_modlist, self).__init__()
            self.modlist = nn.ModuleList([
                           nn.Conv2d(1, 20, 5),
                           nn.ReLU(),
                            nn.Conv2d(20, 64, 5),
                            nn.ReLU()])
    
        def forward(self, x):
            for m in self.modlist:
                x = m(x)
            return x
    ```

  - 使用append添加模块

  - 优点：可循环创建重复度高的网络；forward更灵活；可创建非固定网络（如Progressive GAN）

- `nn.ModuleDict`：每个模块有自己的名字

  - ```python
    class net_moddict(nn.Module):
        def __init__(self):
            self.moddict = nn.ModuleDict()
            self.moddict.update({'0-conv': nn.Conv2d(1, 20, 5)})
            self.moddict.update({'0-relu': nn.ReLU()})
            
        def forward(self, x):
            for n, m in self.moddict.items():
                x = m(x)
            return x
    ```


### 钩子

- 钩子：提取前向传播/反向传播后中间层的输出
  - 通过`net.named_childen()`或`net.named_modules()`找到目标中间层
  - 中间层通过`register_forward_hook()`或`register_full_backward_hook()`绑定一个处理函数
  - 处理函数的输入是中间层的输入和输出，可将其保存到其它变量中
  - 以上在初始化时进行，前向/反向传播后自动调用绑定的函数

## 模型参数

- 模型保存：`torch.save(net.state_dict(), path)`
- 模型加载：`net.load_state_dict(torch.load(path))`
  - 需要先例化一个相同结构的net，再加载参数
  - map_location
- 记录最优参数：`copy.deepcopy(net.state_dict())`
  - 必须深拷贝，浅拷贝只会返回同一套参数的引用

## cuda

- cuda相关
  - 获得gpu个数：`torch.cuda.device_count()`
  - 设定可见gpu数：`os.environ['CUDA_VISIBLE_DEVICES']='4,5,6,7'`：仅物理上的后4张卡可见
    - 程序内部使用逻辑地址指代这些卡，即物理卡4在程序中成为逻辑卡0
    - 在import torch之前
    - 也可以在命令行中输入：`CUDA_VISIBLE_DEVICES=4 python train.py`
  - `device=torch.device('cuda: {逻辑卡号}')`
  - `tensor.to(device)`，`net.to(device)`，`tensor.cuda(逻辑卡号)`
  - 不建议使用torch.cuda.set_device，推荐使用to(device)或CUDA_VISIBLE_DEVICES
- 移至cpu
  - `tensor.cpu()`
  - 需要先移回cpu才能调用.numpy()
- 几个概念
  - CUDA Toolkit：编译环境，可由conda/docker虚拟出来
  - CUDA Driver：运行环境，使用宿主机的
  - GPU Driver：硬件驱动，使用宿主机的，版本>=450.80.02向后兼容


## 多卡训练

- 判断卡：`torch.cuda.is_availabale()`, `torch.cuda.device_count`
- 查看模型在哪：`next(net.parameters()).device`

### DataParallel

- 使用DataParallel包装model, optimizer

  - `model = torch.nn.DataParallel(model, device_ids, out_device)`
    - `device_ids`: 用于训练的卡编号(list of int)，默认使用所有卡
    - `out_device`: 主卡号(int)，主卡用于集中计算loss，负载更大
  - 网络和数据分别需要使用`net.to(device)`和`data.to(device)`
  - 模型保存：`torch.save(net.module.state_dict(), PATH)`

- 指定主卡

  - ```python
    os.environ["CUDA_DEVICE_ORDER"] = "PCI_BUS_ID"
    os.environ["CUDA_VISIBLE_DEVICES"] = "2, 3"
    
    device_ids = [0, 1]
    net = torch.nn.DataParallel(net, device_ids=device_ids)
    
    optimizer = torch.optim.SGD(net.parameters(), lr=lr)
    optimizer = nn.DataParallel(optimizer, device_ids=device_ids)
    ```

### DDP

- 分布式数据并行

  - 多进程，推荐1个进程对应1张卡
  - 支持多机多卡，但如果能单机，就不要多机
  - 比DataParallel更高效

- 新加入的代码

  - ```python
    import torch.distributed as dist
    from torch.utils.distributed import DistributedSampler
    
    parser.add_argument('--local_rank', type=int)  # 启动器会创建多个进程，每个进程通过local_rank获得进程号
    # torch.cuda.set_device(args.local_rank)
    device = torch.device('cuda', local_rank)  # 之后可使用to(device)
    
    # 在创建网络前初始化
    dist.init_process_group(backend='nccl', init_method='env://')  # nccl适合GPU，gloo适合CPU
    
    train_sampler = DistributedSampler(data)  # data是例化后的Dataset
    # 再去掉DataLoader中的shuffle=True，改为sampler=train_sampler，保证不同进程的数据不同
    # DataLoader的batch_size为每个进程的batch_size
    
    net = torch.nn.SyncBatchNorm.convert_sync_batchnorm(net).to(device)  # 将BN设置为进程间同步参数
    net = torch.nn.parallel.DistributedDataParallel(net,
                                                    device_ids=[local_rank],
                                                    output_device=local_rank,
                                                    find_unused_parameters=True)  # 使用DDP接口包装
    
    train_loader.sampler.set_epoch(i)  # 每个epoch计算前
    # 各子进程的DistributedSampler以epoch为种子，保证各机器shuffle出的顺序都相同
    
    # 每个epoch内必须1次forward + 1次backward，多次for/back需要合在一起（如GAN）
    
    dist.barrier()  # 用于进程间同步，仅当全部进程都被dist.barrier()阻塞后才会释放
    
    # 保存模型应保存model.module
    ```
    
  - 若需要写入日志，则仅进程0才能写入，可使用`dist.get_rank() == 0`获得进程号

  - 训练时必须1次forward+1次backward，对于GAN则需要将真样本和生成样本级联在一起送入D（2次forward合为1）

- 启动

  - ```shell
    CUDA_VISIBLE_DEVICES="4,5,6,7" python -m torch.distributed.launch --nproc_per_node 4 train.py
    ```

  - `CUDA_VISIBLE_DEVICES="4,5,6,7"`：指定需要哪几张卡

  - `--nproc_per_node 4`：指定创建几个进程

#### 概念

- 编号
  - rank：机器编号，主机（启动进程的）rank=0
    - 单机多卡只有1个主机，多机多卡才需要设置rank
  - world_size：GPU总数
  - local_rank：当前主机下的GPU编号
- 每个GPU对应1个进程，有单独的模型
  - 每个GPU分别进行前向
  - 各GPU通过Ring-Reduce机制同步梯度，边计算梯度边传梯度
    - Ring-Reduce：所有GPU串成1个环，每个GPU只与环路中下一个GPU通信，传完一圈就同步了梯度


## 专题

### one-hot

```python
# label为(0,1,2,3,...)的数值标签，num_class为类别总数
# 转化为one_hot
one_hot = torch.zeros((label.shape[0], len(num_class)), device=device)
one_hot.scatter_(1, label.view(-1, 1).long(), 1)
```

## 琐碎

- torch.nn.Module提供了各种封装好的网络模块，torch.nn.functionals提供了各种网络运算（卷积等）
- `optim.param_groups['lr']=new_lr`：改变学习率
- `torch.jit`：just-in-time，即时编译
  - 将动态图转化为静态图，使之能被C++调用，加速推理
  - 两种方法
    - `torch.jit.script`：Scripting编码
    - `torch.jit.tracing`：Tracing追踪，提供输入样例



