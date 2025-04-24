# Anaconda使用

- conda --version #查看conda版本，验证是否安装
- conda update conda #更新至最新版本，也会更新其它相关包
- conda update --all #更新所有包
- conda update package_name #更新指定的包

## 创建环境

- `conda create --name <env_name> python=<版本>` #创建名为env_name的新环境
  - `-n`等同于`--name`
  - 加入`-i`参数，标识使用临时代理
- `conda create --name new_env_name --clone old_env_name` #由旧环境拷贝出新环境
- `conda activate env_name` #进入env_name环境
- `conda deactivate` #退出环境
- `conda info --envs` #显示所有已经创建的环境
- `conda config --show channels`源

## 修改

- `conda install --name env_name package_name` #在指定环境中安装包
- `conda rename --name old_env_name new_env_name`  # 修改环境名
- `conda remove --name env_name package` #删除指定环境中的包
- `conda remove --name env_name --all` # 删除整个环境 
- `conda remove package` #删除当前环境中的包

## 导出

- `conda env export > xxx.yaml`：将环境中安装的包及版本号导入xxx.yaml中
- `conda env create -f xxx.yaml`：根据xxx.yaml安装相应conda环境

## 换源

- `conda config --show channels`：显示所有源

- `conda config --remove channels [源url]`：删除源



# Pip使用

- conda环境中默认装pip，一般在conda环境中使用pip指令装库（更靠谱）
- `pip install package_name` #在当前环境中安装包
  - `pip install package_name=1.0.1`：指定安装版本，默认为最新版
  - `pip install -r requirements.txt`：在`requirements.txt`中写全部库名，可批量安装
- `pip install --upgrade pakage_name` #升级包
- `pip uninstall package_name` #删除包
- `pip freeze` #查看当前安装的包
  - `pip freeze > "requirements.txt"`：将环境中库列表写入文件
- `pip show package_name`：显示包的安装信息等
- `pip --help` #查看帮助
- 直接从github安装
  - `pip install git+https://github.com/xxx/xxx.git`
  - `pip install git+https://github.com/xxx/xxx.git@main`：指定安装main分支

## 参数

- `pip install --editable [dir_path / URL]`：安装该路径下的源码，库路径为该路径
  - 若修改源码，则会立即生效

## 手动安装包

- `pip install xxx.whl`：先cd到.whl所在文件夹下

- 其它
  
  - 到镜像源上下载包（tar文件）
  
  - 拷到Linux上，使用tar -zxvf name.tar.gz解压
    
    - cd到解压出的文件夹
      - cd 文件名到下一级
      - ls 查看当前路径下有哪些文件和文件夹
      - cd .. 回到上一级
  
  - python3 setup.py build
  
  - python3 setup.py install

- sudo rm 文件夹 -rf sud #删除文件夹及其内部所有文件