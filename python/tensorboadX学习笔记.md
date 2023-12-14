# tensorboadX学习笔记

- `tensorboardX`不需要安装tensorflow
- `tensorboardX`与`torch.utils.data.tensorboard`调用方式基本一致

##  SummaryWriter

- SummaryWriter：日志对象

  - ```python
    from tensorboardX import SummaryWriter
    writer = SummaryWriter()  # 未指定保存路径，将默认使用runs/日期时间
    writer = SummaryWriter('runs/example')
    writer = SummaryWriter(comment='abc')  # 保存路径：runs/日期时间-abc
    ```

### add_scalar

- `SummaryWriter.add_scalar(tag, scalar_value, global_step=None, walltime=None)`：添加标量记录
  - tag: 曲线名；scalar_value：值
  - global_step：记录的step；walltime：记录发生的时间，默认为time.time()

### add_graph

- `SummaryWriter.add_graph(model, input_to_model=None, verbose=False, **kwargs)`：可视化神经网络
  - model: 待可视化的网络模型；`input_to_model`：待输入神经网络的变量或一组变量（tuple）

