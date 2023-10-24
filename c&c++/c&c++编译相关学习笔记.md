# C/C++编译相关学习笔记

- `gcc`：编译&链接C
  - `-c`：只编译不链接
    - 后跟多个源文件时，为每个源文件生成.o文件，文件之间没有连接
  - `-o`：指定输出文件名，如`gcc hello.c -o hi`
  - `-g`：可使用`gdb`调试
- `g++`：编译&链接C++

## make指令

- make指令可按指定流程编译和链接，可处理复杂的库依赖

- ```makefile
  # make指令会自动搜索当前文件夹下的makefile & Makefile
  
  CC = gcc  # 定义变量
  
  all: helloword   # 不给定make哪个时，默认执行all
  
  # target: prerequisite，target都是逻辑名
  helloworld: hi.o   # 想要helloworld，则需要先有hi.o（hi.o只编译，未链接）
  	$(CC) -o exe hi.o  # $(CC)是变量
  
  hi.o: hello.c hello.h   # 多个依赖项，空格隔开，最后留一个空格
  	$(CC) -c -o hi.o hello.c
  
  .PHONY: clean  # 表示伪目标
  clean:  # 清理之前产生的文件，使用make clean调用
  	-rm *.o fuck
  # 上面-表示忽略该指令出错
  ```
  
  - 编译&链接指令：`make helloworld`
  - 清除编译等产生的文件：`make clean`

### 特定字符

- `@`：make时不在命令行打印该命令
  - `@echo "hello"`
- `-`：忽略命令出错
  - `-rm *.out`











