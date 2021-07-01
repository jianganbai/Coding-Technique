# paddle学习笔记

## 网络结构

- `net = paddle.nn.Sequential()`：定义网络结构
- `model = paddle.Model(net)`：对sequential封装为可调用网络
  - `model.summary(输入大小)`：查看网络中各模块输入输出大小

## 数据输入

- `paddle.vision.datasets`封装了很多数据集的读入函数
- `paddle.io.DataLoader`将dataset录入的数据转化为迭代器

## 琐碎

- `paddle.vision.datasets`：加载常用数据集
- `VisualDL`可以做训练过程的可视化