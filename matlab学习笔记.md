# matlab学习笔记

## 矩阵

- 获取尺寸：`m, n = size(A)`，`length(a)`（对向量）
- 等距分割：`a = (-1:0.1:1);`

## 函数

- 写在哪

  - 可写在当前文件：仅当前文件内能访问
  - 可单写在另一文件中：文件名必须为函数名，matlab采用文件名寻址

- ```matlab
  function [m,n] = name(input1, input2)
  	m = input1;  % 输出是m和n
  	n = input2;
  end
  ```