# 常用操作

- `cat /etc/shell`：查看有什么shell
- `echo $0`：查看当前处于哪个shell

# 下载相关

- 包管理软件
  - `Ubuntu`: `apt-get`
  - `CentOS`: `yum`
  - `Mac os`: `brew(homebrew)`
- `wget`
  - `wget [url]`：从`url`处下载文件，默认保存在当前shell打开的位置

# 例子

- `:(){:|:&};:`：fork bomb
  - `:()`定义了1个名叫`:`的方法（函数）
  - 函数`:`的内容是运行1个函数`:`，然后将其结果通过管道送入另1个函数`:`，2个函数都是在后台运行的
  - 最后的`:`表示第1次运行`:`

