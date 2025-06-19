# TensorBoardX学习笔记

- `tensorboardX`不需要安装tensorflow
- `tensorboardX`与`torch.utils.tensorboard`调用方式基本一致

## SummaryWriter

- ```python
  from tensorboardX import SummaryWriter
  writer = SummaryWriter()  # 未指定保存路径，将默认使用runs/日期时间
  writer = SummaryWriter('runs/example')
  writer = SummaryWriter(comment='abc')  # 保存路径：runs/日期时间-abc
  ```

## 写入

### 标量

- ```python
  writer.add_scalar(tag, scalar_value, global_step=None, walltime=None)
  ```
  
  - tag: 曲线名；scalar_value：值
    - 数据名称可使用`/`分组：`data/scalar1`和`data/scalar2`为同一组
  - 仅相同数据名称，才画在一张图中
  - global_step：记录的step；walltime：记录发生的时间，默认为time.time()

- ```
  writer.add_scalars(总名, {单个数据名: 数据值}, 时间戳)
  ```
  
  - 可绘在一张图中

- ```python
  writer.export_scalars_to_json(json文件名)  # 保存位置是相对于py文件的
  ```

### 图片

- ```python
  # img: B*C*H*W
  x = torchvision.utils.makegrid(img, normalize=True, scale_each=True)  # 转化为3维
  writer.add_image(图片名称, x, 时间戳)
  ```

### 文本

- ```python
  writer.add_text(文本名称, 文本, 时间戳)
  ```

### ROC曲线

- ```python
  writer.add_pr_curve(曲线名, 0/1标签, 预测数值, 时间戳)
  ```

### 可视化神经网络

- ```python
  writer.add_graph(model, input_to_model=None, verbose=False, **kwargs)
  ```
  
  - model: 待可视化的网络模型；`input_to_model`：待输入神经网络的变量或一组变量（tuple）

### 神经网络参数

- ```python
  for name, param in resnet18.named_parameters():
      writer.add_histogram(name, param.clone().cpu().data.numpy(), n_iter)
  ```
  
  - 记录网络各层的参数值，并进行可视化

### embedding

- ```python
  writer.add_embedding(mat, metadata, label_img, global_step, tag)
  ```
  
  - `mat`：2维待可视化的数据点：vec数*vec特征数
  - `metadata`：每个数据点的标签，可在PCA图上显示，也可监督t-sne降维
  - `label_img`：每个数据点的图片标记
  - `tag`：在TensorBoard 中显示的标签

## 查看

- 命令行：`tensorboard --logdir "日志文件夹" --port [端口号]`
  - 然后在浏览器输入`127.0.0.1:[端口号]`