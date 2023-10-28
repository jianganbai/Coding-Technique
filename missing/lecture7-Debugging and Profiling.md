# Debugging and Profiling

## log

### logging库

- `logging`库用于替代print进行debug，常用于python
- 可指定各种消息的打印优先级、打印的风格

### logger

- mac：
  - `log show`：显示系统log
  - `logger "xxx"`：写入系统log

## debugger

### pdb

- 使用：`python -m pdb xxx.py`
  - 进入debug命令行，类似于C/C++用的gdb
- 常用操作
  - 运行相关：
    - `l`：显示代码
    - `s` (step) ：进入函数
    - `restart`
    - `c`：运行全部直到出现bug
    - `q`：退出debugger
  - 断点调试：
    - `p xxx`：打印xxx变量的值
      - `p locals()`：打印当前环境内的所有变量
    - `b x`：在x行设置断点，之后按`c`可运行到断点处

### strace

- 显示某段代码的全部系统调用
  - `sudo strace ls -l > /dev/null`：显示ls和写入操作的全部系统调用
  - `sudo strace -e lstat ls -l > /dev/null`：显示所有lstat系统调用

### pyflakes

- pyflakes是静态检查工具，常用于检查python的语法
  - `pyflakes xxx.py`：检查xxx.py的语法

## profiler

- profiler用于统计用时、资源占用
  - profiler的意思是分析器

### cProfile

- 统计python程序的CPU用时
- `python -m cProfile -s tottime xxx.py 1000`：
  - `-s tottime`：根据总用时排序
  - 1000：运行1000次

### line profier

- 能一行一行地分析性能
- 安装：`pip install line_profiler`
- `kernprof`
  - `kernprof -l -v xxx.py`
    - 分析结果默认写入aaa.py.lprof文件，加入`-v`参数是为了在命令行显示
  - 需要在代码头部写上`@profile`装饰器

### memory_profiler

- `python -m memory_profiler xxx.py`
  - 统计xxx.py每行的资源占用量
  - 在待分析的函数前加上`@profiler`

### perf

- 统计程序的CPU时钟数、缺页数
- 使用：`sudo perf stat stress -c 1`
  - stress是待测程序

## 资源监视器

- `htop`：实时更新的资源监视器，统计cpu利用、内存利用等
- `du`命令：查看硬盘使用大小（disk usage）
  - `du -h [目录]`
- `lsof`命令：整合了netstat和ps，常用于网络数据分析
- `kill`命令：终止进程
  - `kill [进程号]`



















