# plt学习笔记

- `import matplotlib.pyplot as plt`

### 常用属性

- `plt.scatter()`, `plt.plot()`通用

- 点样式：`marker='o'`
  
  - | 符号    | 描述   | 符号    | 描述   | 符号    | 描述   | 符号    | 描述   |
    | ----- | ---- | ----- | ---- | ----- | ---- | ----- | ---- |
    | `'.'` | 小圆点  | `'o'` | 圆圈   | `'v'` | 倒三角形 | `'^'` | 正三角形 |
    | `'<'` | 左三角形 | `'>'` | 右三角形 | `'s'` | 正方形  | `'p'` | 五边形  |
    | `'1'` | 下箭头  | `'2'` | 上箭头  | `'3'` | 左箭头  | `'4'` | 右箭头  |
    | `'*'` | 星号   | `'+'` | 加号   | `'x'` | 叉号   | `'h'` | 六边形  |

- 线型：`ls='-'`
  
  - | 参数值       | 简写     | 说明     |
    | --------- | ------ | ------ |
    | 'solid'   | `'-'`  | 实线（默认） |
    | 'dashed'  | `'--'` | 虚线     |
    | 'dotted'  | `':'`  | 点线     |
    | 'dashdot' | `'-.'` | 点划线    |

- 颜色：`color='b'`
  
  - 默认顺序：`#1f77b4`, `#ff7f0e`, `#2ca02c`, `#8c564b`, `#9467bd`, `#d62728`

- 其它
  
  - `label='123'`：设置该次绘图的标签

## 多图

- ```python
  fig = plt.figure(figsize=(width, height))
  ax = fig.subplots(2, 2)
  # [行编号, 列编号]
  ax[0, 1].plot(x, y, s=s, c=c, marker='x')  # s是标点的大小，c是颜色
  ax[0, 1].set_xlabel('x')
  ax[0, 1].set_title('...')
  ax[0, 1].set_yscale('log')  # y轴设置为对数坐标
  ax[0, 1].legend()  # 图例
  ```

## librosa & plt

- `plt.imsave(file, spec_arr)`：直接保存谱

- ```python
  fig, ax = plt.subplots(nrows=3, ncols=1, sharex=True)  # 共享x坐标
  
  img1 = librosa.display.specshow(S_db, x_axis='time', y_axis='log', ax=ax[0])
  ax[0].set(title='STFT (log scale)')
  
  img2 = librosa.display.specshow(M_db, x_axis='time', y_axis='mel', ax=ax[1])
  ax[1].set(title='Mel')
  
  img3 = librosa.display.specshow(chroma, x_axis='time', y_axis='chroma', key='Eb:maj', ax=ax[2])
  ax[2].set(title='Chroma')
  
  fig.colorbar(img1, ax=[ax[0], ax[1]])  # 共享colorbar，colorbar是右侧反映强度和颜色的框
  
  fig.colorbar(img3, ax=[ax[2]])
  ```