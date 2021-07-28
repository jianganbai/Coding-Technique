# Linux学习笔记

## 基础操作

### 用户相关

- 切换用户：
  - `su`：切换用户（如：`su root`，也可切换至别的用户）
  - 默认root用户是超级用户，具有超级权限；普通用户可使用`sudo`为一条指令申请超级权限，不需要切换为root
- 创建用户：`sudo useradd xxx`
  - 创建用户：`sudo useradd `参数：`-m`代表创建用户主目录~，`-d /home/username`代表用户主目录位置
  - 用户密码：`sudo passwd xxx`，同样适用于root（root初始无密码）
  - 删除用户：`userdel username`
  - 查看当前登录的用户：`who`，`whoami`
  - 查看所有用户：`cat /etc/passwd`，第3个参数500以上就是自建用户

### 包管理软件

- `Ubuntu`: `apt-get`
  - 安装包：`sudo apt-get install xxx`
  - 删除包：`sudo apt-get remove xxx`，仅删除程序，不删除配置文件
    - `sudo apt-get--purge remove xxx`：同时删除程序与配置文件
  - 更新包的索引，之后可更新包：`sudo apt-get update`
  - 更新包：`sudo apt-get upgrade xxx`
    - `sudo apt-get -u upgrade`：升级同时，列出哪些包有升级
- `CentOS`: `yum`=>`yum [option] [command] [package]`
  - `yum check-update`：列出所有可更新的软件清单命令
  - `yum install [包名]`：安装指定包
    - `yum -y install [包名]`：直接安装，不询问y/n
  - `yum update [包名]`：更新指定包
  - `yum remove [包名]`：卸载指定包

### 命令行配置

- 安装`oh-my-zsh`
  - 安装`zsh`：`sudo apt-get install zsh`或`yum install zsh`
- 安装`powerlevel10k`
  - 
- 查看配置：
  - 查看当前shell：`echo $SHELL`

### 查看资源相关

- `lscpu`：查看cpu配置
- `free -m`or`free -g`：查看内存配置
- `df`：以磁盘分区为单位，查看文件系统占地大小
  - `df -h`：查看文件系统占地大小
  - `df -hl`：查看磁盘剩余空间大小
  - `df -h`：查看每个根目录的分区大小

- `du`：disk usage，显示磁盘空间的使用情况
  - `du -sh [目录名]`：返回该目录的大小；`du -sh`：查看当前目录的大小
  - `du -h [目录名]`：查看指定文件夹下所有文件大小（包括子文件）
  - `du [文件名]`：查看指定文件的大小

















## 常用工具

- C编程相关
  - `gcc xxx.c -o yyy`：将C代码编译成`yyy`脚本，之后使用`./yyy`即可执行编译后的代码
  - `g++ xxx.c -o yyy`：将C++代码编译成`yyy`脚本，之后使用`./yyy`即可执行编译后的代码
- wget
  - `wget [url]`：直接下载到当前文件夹
  - `wget -O [name] [url]`：指定下载后文件的名字