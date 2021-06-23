# gdb学习笔记

## 开启gdb调试

- 编译：`gcc test.c -o test -g`
  - `-g`表示加入源码，可使用gdb调试
  - `-o`：指定输出文件名
  - `-c`：只编译不连接，产生.o文件
- 开始调试：`gdb test`

## 调试指令

- 查看所在行：`where`

- 全部执行（重新开始）：`run`
- 单步执行：`start`$\rightarrow$`n`或`s`（n为逐过程，s为逐语句）
- 断点调试
  - `b 8`：在第8行放置断点
  - `b 8 if i == 1`：在第8行放置断点，当i=1时触发断点
  - `b [函数名]`：在调用函数处放置断点
  - `run`：运行到断点处；`c`：继续
  - `i breakpoints`or`i b`：查看所有断点
  - `delete breakpoints [断点编号]`：删除断点
- 显示变量值
  - `p int_val`：显示名为int_val的变量值
  - `p *int_array@3`：显示int_array数组的前3个元素