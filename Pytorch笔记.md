# Pytorch笔记

## 琐碎

- **三大神器**
  - `optimizer.zero_grad()`：将optimizer内的梯度置零
  - `loss.backward()`：计算损失函数关于各个参数（网络参数等，不含输入）的梯度
  - `optimizer.step()`：在给定梯度时有不同梯度下降策略
    - 每个样本都能给出一个梯度
    - 该选择哪些样本的梯度？是否要考虑历史梯度？沿梯度方向下降的学习率如何变化？这些都由optimizer决定
- 如何使用cuda？
  - `torch.cuda.is_available()`给出计算机是否有cuda
  - 将网络迁移到cuda上：`net.cuda()`
  - 将损失函数迁移到cuda上：`loss_func.cuda()`
  - 将所有输入神经网络的数据迁移到cuda上
    - 可使用格式转换符：`torch.cuda.FloatTensor`，然后在所有数据前面强制转换
      - `Tensor()`或`imgs.type(Tensor)`，`Tensor=torch.cuda.FloatTensor`
- 所有手动给定的参数，都要设置成`Variable`形式
  - 在计算梯度时，torch默认对所有这样的参数求偏导
    - `Tensor`不能反向传播，`Variable`能反向传播
  - 若不需要求偏导，则在构造`Variable`时设定`requires_grad=False`
- `net.eval`与`torch.no_grad`
  - `net.eval`用于设定dropout层和BN层，但计算过程中仍计算梯度
  - `torch.no_grad`用于取消计算梯度
- `tensor.squeeze()`与`tensor.unsqueeze()`
  - `tensor.squeeze()`：去除所有大小为1的维度
  - `tensor.squeeze(arg)`：若第arg维的维度值为1，则去除之，否则保持原tensor不变
  - `tensor.unsqueeze(arg)`：在第arg维插入维度值为1







