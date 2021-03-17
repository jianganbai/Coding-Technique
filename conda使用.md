# Anaconda使用

## 环境记录

### pyft

- 初始库

### pyrobot

- 智能机器人课专用环境





## 用语

### 基础

- conda --version #查看conda版本，验证是否安装
- conda update conda #更新至最新版本，也会更新其它相关包
- conda update --all #更新所有包
- conda update package_name #更新指定的包
- conda create -n <env_name> python=<版本> #创建名为env_name的新环境
  - 例如：conda create -n python2 python=python2.7 numpy pandas，创建了python2环境，python版本为2.7，同时还安装了numpy pandas包
- source activate env_name #切换至env_name环境
- source deactivate #退出环境
- conda info -e #显示所有已经创建的环境
- conda create --name new_env_name --clone old_env_name #复制old_env_name为new_env_name
- conda remove -n env_name –-all #删除环境
- conda config --show-sources #查看添加的镜像源
- conda list #查看所有已经安装的包

- conda install --name env_name package_name #在指定环境中安装包
- conda remove -- name env_name package #删除指定环境中的包
- conda remove package #删除当前环境中的包

### 提高

- conda create -n tensorflow_env tensorflow

- conda activate tensorflow_env #conda 安装tensorflow的CPU版本

- conda create -n tensorflow_gpuenv tensorflow-gpu

- conda activate tensorflow_gpuenv #conda安装tensorflow的GPU版本

- conda env remove -n env_name #采用第10条的方法删除环境失败时，可采用这种方法

# Pip使用

- pip freeze #查看当前安装的包
- pip install package_name #在当前环境中安装包
- pip install --upgrade pakage_name #升级包
- pip uninstall package_name #删除包
- pip --help #查看帮助

## 手动安装包

- 到镜像源上下载包（tar文件）
- 拷到Linux上，使用tar -zxvf name.tar.gz解压
- cd到解压出的文件夹
  - cd 文件名到下一级
  - ls 查看当前路径下有哪些文件和文件夹
  - cd .. 回到上一级
- python3 setup.py build
- python3 setup.py install

- sudo rm 文件夹 -rf sud #删除文件夹及其内部所有文件