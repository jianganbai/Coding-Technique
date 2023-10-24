# TensorFlow学习笔记

- tensorflow2

## 琐碎知识

- shape：tensorflow有2种shape
  - static(inferred) shape: 构建图时确定
  - dynamic(true) shape: 
- tf.nn与tf.layers
  - tf.nn: 最基础的层，需自定义权重（可调整学习率、正则化等）
  - tf.layers: 对tf.nn的高级封装，提供了更高级的卷积层（如转置卷积、分组卷积等）

## 常见函数

- `tf.cast(x, dtype)`：改变x的数据类型，返回新变量
- `tf.tile(x, [第0维复制倍数,第1维复制倍数,...])`：分维度重复x
- `tf.set_shape(x, [第0维维度,第1维维度,...])`：设定参数大小，不符则报错
- `tf.get_shape(x)`：返回tensor x的static shape
- `tf.reshape(x, [第0维维度,第1维维度,...])`：调整大小，返回新变量