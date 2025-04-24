# docker学习笔记

- docker文档：[doc.docker.com/engine/reference/commandline](doc.docker.com/engine/reference/commandline)

## 概念

- docker容器
  
  - | 特性  | 普通虚拟机                        | Docker                                     |
    | --- | ---------------------------- | ------------------------------------------ |
    | 跨平台 | 通常只能在桌面级系统运行，无法在不带图形界面的服务器运行 | 支持的系统非常多，各类windows和linux                   |
    | 性能  | 虚拟出整个系统，占用的资源多               | 只虚拟软件所需运行环境，不同系统上都能模拟出一模一样的环境，保证切换环境时不会出问题 |
    | 自动化 | 手动安装                         | 设置好image后，1条命令即可部署                         |
    | 稳定性 | 稳定性不高，不同系统差异大                | 稳定性好                                       |

- 基本概念
  
  - **打包**：将软件、依赖、第三方库打包成安装包
  - **分发**：把安装包上传到镜像仓库，其他人可方便获取
  - **部署**：通过安装包，运行应用
  - **镜像image**：软件安装包
  - **容器container**：安装好的软件，不同软件的环境彼此隔离

## 查询

- `docker images`：查看拉取下的镜像
- `docker ps`：查看目前在运行的容器
  - `-a`：查看所有容器，包括未运行的
  - 无法查询到镜像，容器是实例化的镜像

## 创建镜像

- 推荐方法：**实例化基础镜像，在基础容器中配置好所有环境，再导出为新镜像**

### dockerfile

- 文件名为`Dockerfile`

- ```dockerfile
  FROM mirrors.tencent.com/star_library/xxx:[版本号]  # 基于哪个基础镜像
  
  WORKDIR /app  # 设置工作目录，也是创建容器后终端默认登录的目录
  EXPOSE 80/tcp  # 暴露容器的哪个端口给外界，tcp为采用tcp协议
  
  ENV [key]=[value]  # 设置环境变量
  
  # ADD命令不能像目录挂载一样同步
  # 格式与cp一致。ADD命令可使用通配符*?等。ADD命令自动解压缩
  ADD [宿主机文件路径] [镜像文件路径]  # 将宿主机文件拷贝到镜像内，也可从URL拷贝文件到镜像内
  COPY [宿主机路径] [镜像内路径]  # 将宿主机文件拷贝到镜像内
  
  # 只能写一个RUN，每执行一个RUN都会在原来的基础上新开一个新镜像
  RUN pip3 install opencv-python=4.2.0.32 \ # 安装python包
      apt-get update && apt-get install wget \ # 安装wget
      wget [url] && cd install && sh install.sh  # 使用wget下载软件并安装
  
  CMD ["/bin/bash"]  # 如果docker exec最后有命令，如/bin/bash，则会覆盖这里给定的命令
  ```

### docker build

- 先创建dockerfile
- 再使用`docker build [参数名] [dockerfile的路径，一般为.]`
  - `-t`：设置镜像名和版本，例如：`docker build -t text:v1 .`

### docker push

- `docker push [镜像url]`
  - 将创建的镜像提交到远程镜像库
  - 例如：`docker push mirrors.tencent.com/[命名空间]/[镜像名]`

### docker-compose

- 组合多个容器，包装为一个容器

- 先定义`docker-compose.yml`
  
  - ```yaml
    version: "3.7"  # 包装后容器的版本
    
    services:
      app:  # 子容器1
        build: ./  # 子容器1通过在本地build创建
        ports:
          - 80:8000
        volumes:
          - ./:/app  # 目录挂载
        environment:
          - TZ=Asia/Shanghai
      redis:  # 子容器2
        image: redis:5.0.13  # 从docker hub下载
        volumes:
          - redis:/data  # 目录
        environment:
          - TZ=Asia/Shanghai
    
    volumes:
      redis:
    ```

- 先切换到`docker-compose.yml`所在目录
  
  - `docker-compose up -d`：按yml文件运行容器（首次运行）
  - `docker-compose ps`：查看该yml文件所有涉及的容器
  - `docker-compose stop`：关闭整个容器
  - `docker-compose start`：再次运行整个容器
  - `docker-compose restart [子容器名]`：重新运行单个子容器，子容器名在yml内定义

## 下载镜像

- `docker pull [image url]`：从docker hub下载镜像
  - 例如：`docker pull mirrors.tencent.com/[命名空间]/[仓库名]：[tag，版本号]`

## 运行容器

- `docker run [OPTIONS] 镜像名 [COMMAND] [ARG...]` ：创建后首次运行
  - `docker run -d -p 6379:6379 --name redis redis:latest`
  - `redis:latest`：使用的镜像及版本，若没有则下载
- 常用options参数
  - `-it`：交互式运行
  - `-d`：detach，让容器在后台运行
  - `-p [宿主机端口]:[容器端口]` ：指定端口映射
  - `-v [宿主路径]:[容器路径]` ：配置目录挂载（见下）
  - `-w [启动后进入的路径]`：设置启动后进入的路径
  - `--name [容器名]`：容器名
  - `--network [网络名]`：指定网络
  - `--network-alias [网络别名]`：指定网络别名
  - `--shm-size=10G`：指定使用的内存
  - `--gpus all`：指定是否要使用GPU、使用哪个
    - docker 19之前：无法使用GPU，需要通过nvidia-docker或nvidia-docker2（对docker的包装）
  - `-e`：指定环境变量，如`-e NVIDIA_VISIBLE_DEVICES=all`
  - `--rm`：退出容器时自动删除容器
- `docker run [镜像名] [容器内执行的命令]`
  - 容器内执行的命令：如`bash`，`/bin/sh`

## 进入容器

- `docker start [容器名]`：暂停后重新运行
- `docker exec -it [容器id或容器名] [容器内执行的命令]`
  - 容器id：利用`docker ps`查找，不是镜像名

## 目录挂载

- 不加挂载的问题
  
  - 改了代码得重新build和run => 修改后，仅需要重启容器
  - 容器内产生的文件没有保存到本地，删了容器，文件就没了

- 分类
  
  - **bind mount**：直接把宿主机目录映射到容器内，适合挂代码目录和配置文件
    - `docker run -v D:\code:/app`
    - 将`D:\code`中的文件（必须为绝对路径），挂载到容器的`/app`内
    - 数据迁移时，直接找到宿主机目录并拷贝，即可
  - **volume**（推荐）：由容器在宿主机创建并管理，删除容器不会丢失，适合存储数据库数据
    - `docker run -v db-data:/app`
    - db-data为宿主机存储区域的名字，不同容器挂载这个区域时，直接输db-data即可
    - 数据迁移时，只能在容器中操作
      - 打开ubuntu镜像，挂载备份的目录，然后tar打包压缩，cp到备份volume中
  - tmpfs moun（很少用）：适合存储临时文件，存放在宿主机内存中

- **将文件拷贝进容器的方法**
  
  - **创建镜像时拷贝**：在dockerfile文件中用ADD或COPY命令
    
    - 文件存在镜像里，存进后无法同步
  
  - **实例化容器时复制**：bind or mount
    
    - 文件存在宿主机里，可实时同步
  
  - **docker cp命令**：`docker cp [宿主机路径] [容器名]:[容器内路径]`，

## 多容器通信

- 创建虚拟网络：`docker network create test-net`

## 发布

- docker hub：官方的镜像库

- 先在docker hub上创建用户和仓库

- 保存镜像
  
  - `docker commit [容器id] [镜像名]:[版本号]`：将现有容器保存为镜像
  - `docker tag [原镜像名]:[原版本] [目标镜像名]:[目标版本]`：修改镜像名或版本号

- 发布
  
  - `docker login -u [用户名]`：第1次push，需要在命令行登录账号
  - `docker tag test:v1 [用户名]/test:v1`：新建tag
  - `docker push [用户名]/test:v1`：把镜像推上去（同名）

## 关闭

- 关闭容器：`docker stop [容器名]`

## 删除

- 删除镜像：`docker rmi [镜像名]`
  
  - 删除镜像前，需要关闭所有基于它创建的容器

- 删除容器：`docker rm [容器名或容器id]`
  
  - `docker container prune`：删除所有容器