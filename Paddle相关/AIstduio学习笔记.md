# AIstudio学习笔记

## Notebook使用

### 快捷键

- 命令模式
  - `ctrl+上箭头`：向上移动cell，`ctrl+下箭头`：向下移动cell
  - `ctrl+回车`：运行当前cell，`shift+回车`：运行当前cell并移动至下一个cell

- 编辑模式
  - `ctrl+/`：注释整行/取消注释整行
  - `ctrl+左箭头`：前往行首，`ctrl+右箭头`：前往行尾

### Magic命令

- 以`%`开头：行Magic命令；以`%%`开头：单元格Magic命令
- `%%timeit`：统计单元格的时间
- `%matplotlib inline`：运行后显示plt.plot绘图结果
- `%env`：设置环境变量
- `%run xxx.py`：运行python代码，等价于`!python xxx.py`

### pdb调试



### 其它常用功能

- Notebook中使用shell：在shell命令前加`!`，就可用在Notebook中