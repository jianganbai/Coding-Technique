# plt学习笔记

- `import matplotlib.pyplot as plt`

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