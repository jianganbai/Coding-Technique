# tqdm学习笔记

- python进度条

- ```python
  from tqdm import tqdm
  
  for i in tqdm(range(10)):
      # do something
  ```

- ```python
  pbar = tqdm(['a', 'b', 'c', 'd'])
  for c in pbar:
      # do something
      pbar.set_description('Processing %s' %c)  # 在进度条最左侧显示提示
      pbar.set_postfix({'a': 1, 'b': 2})  # 在右侧[]内显示该轮的提示
  ```
  
- ```python
  with tqdm(total=100, desc='...') as pbar:  # desc为最左侧的描述
      for i in range(100):
          # do something
          pbar.update(1)
  ```

- ```python
  pbar0 = tqdm(total=100, position=0)  # 多个进度条，使用position区分位置
  pbar1 = tqdm(total=200, position=1)
  ```

- 