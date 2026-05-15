# Pathlib 学习笔记

- `pathlib`比`os.path`更适合处理路径
  
  - `os.path`纯用文本记录路径，`pathlib`使用对象记录路径，可进行更多操作
  - `pathlib`跨平台更兼容

## 定义 & 访问

- 定义路径
  
  - ```python
    from pathlib import Path
    current_working_dir = Path.cwd()
    home_dir = Path.home()
    p = Path("/Users/youruser/Documents/project/data.txt")
    ```

- 路径属性
  
  - ```python
    print(p.name)  # data.txt
    print(p.stem)  # data
    print(p.suffix)  # .txt
    print(p.parent)  # /Users/youruser/Documents/project
    print(p.anchor)  # /, root of the path
    print(p.parts)  # ('/', 'Users', 'youruser', 'Documents', 'project', 'data.txt')
    ```

- 合并路径：`q = 'a' / 'bb' / 'ccc.txt'`

- 确定路径：`p_real = p_relative.resolve()`
  
  - 去掉相对路径、链接等，获得绝对路径

## 处理

- 创建、重命名、删除
  
  - ```python
    from pathlib import Path
    dummy_dir = Path('temp_dir')
    dummy_dir.mkdir(parents=True, exist_ok=True)  # 创建所有不存在的父目录
    print(dummy_dir.exists())
    print(dummy_dir.is_file())
    print(dummy_dir.is_dir())
    
    dummy_file = dummy_dir / 'temp.txt'
    
    # 移动文件。相对于文件所在路径。
    new_dummy_file = dummy_file.rename('dummy.txt')  # 若目标路径存在，则引发FileExistsError
    new_dummy_file = dummy_file.replace('dummy.txt')  # 若目标路径存在，则直接覆写
    
    dummy_dir.rmdir()
    ```

- 读写文件
  
  - ```python
    from pathlib import Path
    file_to_write = Path('my_output.txt')
    file_to_write.write_text('Hello world!')
    content = file_to_write.read_text()
    ```

## 遍历

```python
  from pathlib import Path
  target_dir = Path('test_dir')
  for item in target_dir.iterdir():  # 遍历下一级的所有文件和文件夹
      print(item)
  for txt_file in target_dir.glob('*.txt')  # 遍历下一级所有的.txt文件，不递归搜索
      print(item)
  for recursive_txt_file in target_dir.rglob('*.txt')  # 递归遍历各层级所有的.txt文件
      print(item)
```
