# PIL学习笔记

- ```python
  import numpy as np
  from PIL import Image
  
  img = Image.open('image.jpg')  # 读入图片，格式为PIL.Image
  x = np.array(img)  # 转化为numpy
  img = img.convert('L')  # 转化模式为灰度
  ```

- 多种模式

  - 1：二值
  - L：灰度，0为黑，255为白
  - RGB：RGB彩色
  - 