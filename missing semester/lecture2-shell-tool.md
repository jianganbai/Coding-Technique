# Shell Tools and Scripting

- 对缩进及其严格

## 基础知识

- **变量**
  - 定义变量：`foo=bar`，**赋值时无空格**
  - 变量引用：`$变量名`or`${变量名}`
    - `echo $foo`显示`foo`变量存储的值；`echo foo`则显示`foo`
  - 临时环境变量：`export VAR=value`，子shell可继承
- **引号**
  - 单引号''被当做纯字符串，双引号""会先替换内部的变量，反引号``被当做命令
  - `echo '$foo'`输出`$foo`；`echo "$foo"`输出`bar`
- **特殊变量**
  - 环境变量：`$PATH`指kezhixing chengxu lujing ，`$PWD`指当前文件夹，`$SHELL`指默认shell
  - 位置参数：`$0`：脚本名；`$1`：第1个参数名；`$2-$9`：第2-9个参数名；`$(10)`：第10个参数
  - `$#`参数个数；`$_`：上1个命令的最后1个参数；`$$`：进程号PID；`$@`：所有参数
  - `$?`：存储错误（异常）代号，无错误（异常）则为0
  - `$([命令])`：取某命令的输出结果
  - `!!`：上1个命令。常用于：`sudo !!`，以超级用户执行上1个命令
- 运行脚本
  - 法1：加载脚本进命令行：`source xxx.sh`；执行脚本：`mcd test`
    - `source`一般用于更新
  - 法2：`./mcd.sh`
  - 法3：`sh mcd.sh`

## 运算符

- 逻辑运算：`&&`，`||`
- 位运算：
  - `<<`，`>>`：左移 or 右移
  - `&`：按位与；`|`：按位或；`~`：按位非；`^`：按位异或

## 分支循环

- 条件：`[ 条件 ]` or `[[ 条件 ]]`（前后都有空格）

  - `-eq`：等于，`-ne`：不等于，`-gt`：大于，`-lt`：小于，`-ge`：大于等于，`-le`：小于等于
    - `[1 -le 2]`

- 分支：

  - ```shell
    if 条件表达式 ; then
    	命令
    elif 条件表达式 ; then
    	命令
    else
    	命令	
    fi
    ```

- for循环：

  - ```shell
    for 变量名 in 取值列表
    do
    	命令
    done
    ```

- while循环：

  - ```shell
    while 条件变量 ; do
    	命令
    done
    ```

## Shell脚本例子

- 命令行可执行很多语言的脚本：shell, bash, python
- `shellcheck`：查看shell脚本语法错误

- ```bash
  mcd () {
  	mkdir -p "$1"  # $1代表输入的第1个参数（文件夹名），相当于argv[0]
  	cd "$1"  # 进入该文件夹
  }
  ```

- ```bash
  echo "Starting program at $(date)"  # $(date)为系统时间
  echo "Running program $0 with $# arguments with pid $$"  # $0：脚本名；$#：参数个数；$$：进程号
  for file in "$@"; do  # $@类似于迭代器
  	grep foobar "$file" > /dev/null 2> /dev/null  # /dev/null是1个写入后无法保存的路径
  	if [["$?" -ne 0]]; then  # -ne是不等于
  		echo "File $file does not have any foobar, adding one"
  		echo "# foobar" >> "$file"  # 在文件最后添加
  	fi
  done
  ```

## Python脚本

- 在脚本第1行加上：`# !/user/bin/env python`，可自动从环境变量中选取python编译器

