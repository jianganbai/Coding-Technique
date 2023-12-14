# git使用

- 详见Missing Semester笔记（互相补充看）

## git工作区

- 工作目录：修改文件
- 暂存区：将修改的文件暂存，但没有push到远端
- 远端仓库：远端存储文件

## 工作流程

- 从别人的仓库fork到自己的仓库
- 将自己的仓库clone到本地
- 在本地上修改
- 检查upstream仓库是否有更新，有则rebase
- 将本地的修改push到自己fork的仓库中
- 从自己fork的仓库向别人的仓库发pull request

## git常用指令

- 以下指令在git bash和vscode中均可执行

### 克隆与新建

- `git clone <远端仓库网址> <本地路径>`
  - 将远端仓库clone到本地
  - 在本地目录下打开cmd输入以上指令，本地路径默认为当前路径
  - 远端仓库地址可在github仓库的copy选项卡下找到
  - 被克隆的远端仓库自动成为origin
  - `git clone git@github.com:<用户名>/<仓库名>.git`：克隆私有仓库
    - `git clone -b <分支名> git@github.com:<用户名>/<仓库名>.git`：克隆私有仓库的指定分支
- `git remote add upstream <别人仓库的网页地址>`
  - 将别人的仓库添加为upstream类型（当然也可以把upstream改成origin）
  - 这样发pull request可直接对比origin仓库和upstream仓库
  - `git remote -v`：查看所有远程仓库
- `git branch <新分支名>`  #创建新分支
  - `git branch -m <新名字>`：本地当前分支改名
  - `git branch -d <本地分支名>`：删除本地分支
  - `git push origin :old_branch`：删除远程分支
- `git checkout <分支名>` #切换到某个分支
- `git checkout -b <新分支名>` #创建并切换

### 从working tree到stage和本地库

#### git add

- `git add <文件名>`
  - 将修改从working tree提交到stage（暂存）
  - `git add .`：提交所有文件进stage，考虑gitignore
  - `git add *`：提交所有文件进stage，忽略gitignore
  - `git add -u .`：仅提交已被追踪的文件进stage，不提交新文件
  - `git add -p [文件名]`：交互式提交，逐一询问每个修改是否要加入
- 暂存错了
  - `git reset`
    - `git reset HEAD`：完全回退到git add前
    - `git reset HEAD [不想提交的文件]`

  - `git restore`
    - `git restore --stage [不想提交的文件]`：将文件从stage区移出，但不修改文件


#### git commit

- `git commit -m "<commit信息>"`
  - `git commit`将修改提交到本地库
  - 为从working tree到stage的每次提交写commit 信息，加双引号代表这是1个参数
  - 更推荐直接在vscode的左上角文本框中写commit信息
- 其它参数
  - `git commit -a`：自动先进行add，再进行commit
- `git diff`：比较工作区和暂存区的区别

### 从stage到remote

#### 同步

- `git pull upstream <分支名> --rebase`
  - 从upstream下载远端文件，并与本地已有文件采用rebase方式比较
  - 需要手动解决冲突
    - vscode可比较代码不同的地方，并选择采纳哪个
    - `git rebase --abort` #终止变基
    - `git rebase --continue` #解决完冲突后使用
  - **每次干活前和每次准备提交前都需要rebase**

#### 检查

- `git status` #查看暂存区状态
- `git diff <文件地址>` #查看文件修改内容
  - `git diff <本地分支名>`：比较当前分支与本地分支
  - `git diff <远端名，如origin>/<远端分支名>`：比较当前分支与远端分支
- `git log` #查看别人的提交日志
- `git reset`
  - 用于回退到之前某个分支，本地克隆中该分支之后的所有修改均被舍弃
  - `git reset <commit-id>` #reset到指定id的commit
  - 更推荐的是直接在vscode中使用插件查看分支生长情况，决定reset到哪个分支
  - reset分为soft和hard，一般选soft

#### 推送

- `git push <远端仓库名> <分支名>`
  - `git push -u origin <分支名> ` #push到origin中新创建的分支
  - `git push orgin <分支名> -f` 
    - 强制push，即将远端仓库删光，再拷贝本地仓库上去
    - 不要乱用，一般只用于reset到远端旧版本后的push

### reset

- 2种reset：soft和hard
  - soft：保留新代码，相当于撤销commit但不撤销add
  - hard：回滚至之前的代码，相当于撤销add和commit
- reset本地：`git reset --soft [希望rest到的commit的hash值]`
  - `git reset --soft HEAD^`：返回上一次commit
- reset远端：
  - `git reset --hard [希望reset到的commit的hash值]`
  - `git push origin HEAD --force`

## github action

- 持续集成/持续交付 (CI/CD) :

  - 每个人完成自己部分后就单独测试，不要等所有人都开发完成后再统一测试
  - 提高开发效率

- **github action**：进行push / pull request后，自动对新代码进行测试

  - 测试流程写在`.yml`文件中

  - ```yaml
    name: SCRIPT_NAME
    
    on:  # 触发时机
      workflow_dispatch:  # 手动触发
        push:  # 代码提交触发
          branches:  # 在哪个分支
            - main  # 在main分支
        pull_request:  # PR提交时触发
        
    jobs:  # 工作流
      first_job:  # 工作名称
        runs-on:  ubuntu-latest  # 运行在哪个环境中
        steps:  # 步骤
          - name: Checkout  # 步骤名
            uses: actions/checkout@v3  # 使用market上的脚本包
       
          - names: Setup node
            uses: action/setup-node@v3
            with:  # 设定参数
              node-version: 16.13.x
              cache: npm
             
          - name: Install
            run: npm ci  # 运行脚本命令
    ```


## 其它

- ssh key

  - 生成ssh key：`ssh-keygen -t rsa -C "github绑定的邮箱"`

  - 若有多个ssh key（git、服务器等），则需要专门在`~/.ssh/config`文件中说明用哪个

    - ```yaml
      Host github.com  # 若使用其他名字，则git@github.com中的github.com应替换为该名字
        HostName github.com
        PreferredAuthentications publickey
        IdentityFile [私钥的绝对路径]
      ```

  - 若不行，使用`ssh -i [私钥的绝对路径]`
  - 测试ssh key：`ssh -T git@github.com`，要debug信息可以使用`ssh -vT git@github.com`


- git代理：
  - 查看已设置的代理：`git config --global --list`
  - 添加代理：`git config --global [代理地址]`

