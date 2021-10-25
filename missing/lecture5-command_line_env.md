# 命令行环境

- Job Control; Terminal Multiplexiers; Dotfiles; Remote Machines
- 常用工具
  - htop：显示系统资源的使用情况
  - tree：显示文件索引结构

## 信号

- `man signal`查看linux所用信号

- 信号

  - SIGINT：ctrl+c发出，中断进程，int=interrupt
  - SIGSTOP：ctrl+z发出，将进程挂起
  - SIGQUIT：ctrl+\发出，退出程序

- 命令

  - `&`：后台执行，如`sleep 20 &`
  - `jobs`：查看该命令行创建但未结束的进程
  - `bg %进程序号`：开始某进程，进程序号可由jobs命令查看
    - bg: back ground
  - `kill 参数 %进程序号`：向进程发送特定信号
    - `kill -STOP %1`：向进程1发送SIGSTOP信号
    - 默认发送SIGKILL

- python的signal库

  - ```python
    import signal
    def handler(signum, time):  # 该函数处理ctrl+c的异常
      print("I got a SIGINT, but I am not stopping\n")
    signal.signal(signal.SIGINT, handler)  # 当键盘输入ctrl+c时，会调用该行
    # 可通过ctrl+\提交SIGQUIT，以退出
    ```

## 多窗口

- 常用：tmux, screen
- tmux概念
  - sesssion会话：一系列任务
  - window窗口：1个任务
  - pane窗格：显示屏上同时显示多个终端，它们都属于同一窗口
- 快捷键
  - ctrl+b进入tmux快捷键，每个输入1个快捷键前都需要输入1次ctrl+b
    - 默认是ctrl+b，大多数人选择修改为ctrl+a
  - `d`：与当前进程脱离，当前进程仍在运行
  - `c`：创建新窗口
  - `p`：进入上一窗口
  - `n`：进入下一窗口
  - `数字`：进入第几个窗口，从0开始
  - `,`：重命名窗口
  - `"`：水平创建新窗格
  - `%`：竖直创建新窗格
  - `方向键`：在不同窗格间切换
  - `空格键`：切换不同窗格环绕风格
- 命令行中的tmux
  - `tmux new -t foobar`：新开1个名叫foobar的会话
  - `tmux a -t foobar`：进入已创建好的会话
    - ctrl+b d退出
  - `tmux ls`：查看当前所有的会话

## 配置文件

- `alias`：自定义系统命令
  - `alias ll="ls -alh"`：将ls -alh简写为ll，之后输入ll即代表ls -alh
    - 等号前后无空格，否则认为有多个参数
  - `alias mv="mv -i"`：出现写入覆盖时需要再次确认，常用配置
  - `alias 命令名`：展示被alias重定义的命令的实际意义
  - alias可写在.bashrc中，这样每次启动bash时都会自动运行
- .bashrc
  - 每次启动bash时都会运行bashrc中的指令，因而配置命令都写在.bashrc中
- symlink：软链接/符号链接
  - 将.bashrc等配置文件链接至其它文件，使用其它文件代替原.bashrc的作用
  - `ln -s 原文件 替代文件`
    - `ln`：硬链接
    - `ln -s`：软链接

## 远程服务器

- SSH
  - 基础指令
    - `ssh 用户名@IP地址`：命令行执行，执行成功后在本地命令行对服务器命令行操作
    - `ssh 用户名@URL`
    - `logout`：登出
    - `scp 本地文件名 用户名@IP地址:远端文件名`：拷贝文件
  - SSH key：private key保存在本地，public key保存在远端
