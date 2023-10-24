# paddle学习笔记

## 网络结构

### 基础概念

- `paddle.nn`定义类，自带参数；`paddle.nn.functional`定义函数，需要手动传入参数（适合不需要参数的模块）
  - `paddle.nn.Layer`提供常见模块的类
- 动态图：命令式编程范式，写一行网络就能得出结果；静态图：声明式编程范式，必须先定义好才能编译执行



















## 数据输入

- `paddle.vision.datasets`封装了很多数据集的读入函数
- `paddle.io.DataLoader`将dataset录入的数据转化为迭代器

## 模型保存

- `paddle.save(model.state_dict()， 'xxx.pdparams')`可保存模型参数
- `paddle.save(opt.state_dict(), 'pdparams')`可保存优化器状态

## 琐碎

- `paddle.vision.datasets`：加载常用数据集
- `VisualDL`可以做训练过程的可视化
- `paddle.set_device(device)`指定训练设备
  - 使用set_device后，每次定义tensor或加载数据时默认将其转移到cuda上
- `paddle.optimizer`中可设定weight_decay参数，实现L1/L2正则化以抑制过拟合
- `paddle.optimizer.lr.PolynomialDecay`可设定带衰减的学习率
- `@paddle.jit.to_static`用于添加装饰器，可使得动态图网络结构在静态图模式下运行