# TensorBoardX学习笔记

- TensorBoardX：将tensorflow的tensorboard适配给pytorch
  - 提供了方便的可视化工具
  - 需要像日志一样记录数据，然后系统进行绘图、可视化等
- torch也适配了tensorboard：`torch.utils.tensorboard`

## SummaryWriter

- ```python
  from tensorboardX import SummaryWriter
  writer = SummaryWriter(logdir)
  
  # 日志操作
  
  writer.close()
  ```
  
  - logdir：保存的文件夹名
    - 默认保存在./runs/

## 操作

### 标量

- ```python
  writer.add_scalar(数据名称, 数据值, 时间戳)
  ```

  - 数据名称可使用`/`分组：`data/scalar1`和`data/scalar2`为同一组
  - 仅相同数据名称，才画在一张图中

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
  writer.add_graph(神经网络(nn.Module), 网络输入示例, verbose)
  ```

### 神经网络参数

- ```python
  for name, param in resnet18.named_parameters():
      writer.add_histogram(name, param.clone().cpu().data.numpy(), n_iter)
  ```

  - 记录网络各层的参数值，并进行可视化

### embedding

- ```python
  writer.add_embedding(mat, metadata, label_img)
  ```

  - mat是2维待可视化的数据点：vec数*vec特征数
  - metadata是每个数据点的标签，可在PCA图上显示，也可监督t-sne降维
  - label_img是每个数据点的图片标记