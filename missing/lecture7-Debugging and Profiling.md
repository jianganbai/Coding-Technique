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

- 使用：`python -m ipdb xxx.py`
  - 进入debug命令行，类似于C/C++用的gdb
- 常用操作
  - 运行相关：
    - `l`：显示代码
    - `s`：开始运行
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

### cProfile

- 统计python程序的CPU用时
- `python -m cProfile -s tottime xxx.py 1000`：
  - `-s tottime`：根据总用时排序
  - 1000：运行1000次

### line profier

- 能一行一行地分析性能

- `kernprof`
  - `kernprof -l -v xxx.py`
    - 分析结果默认写入aaa.py.lprof文件，加入`-v`参数是为了在命令行显示
  - 需要在代码头部写上`@profile`

### memory_profiler

- `python -m memory_profiler xxx.py`
  - 统计xxx.py每行的资源占用量

### perf





















