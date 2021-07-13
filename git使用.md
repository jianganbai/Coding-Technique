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

#### 克隆与新建

- `git clone <远端仓库网址>`
  - 将远端仓库clone到本地
  - 在本地目录下打开cmd输入以上指令
  - 远端仓库地址可在github仓库的copy选项卡下找到
  - 被克隆的远端仓库自动成为origin
  - `git clone git@github.com:<用户名>/<仓库名>.git`：克隆私有仓库
    - `git clone -b <分支名> git@github.com:<用户名>/<仓库名>.git`：克隆私有仓库的指定分支
- `git remote add upstream <别人仓库的网页地址>`
  - 将别人的仓库添加为upstream类型（当然也可以把upstream改成origin）
  - 这样发pull request可直接对比origin仓库和upstream仓库
- `git branch <新分支名>`  #创建新分支
  - `git branch -m <新名字>`：本地当前分支改名
  - `git push origin :old_branch`：删除远程分支
- `git checkout <分支名>` #切换到某个分支
- `git checkout -b <新分支名>` #创建并切换

#### 从working tree到stage和本地库

- `git add <文件名>`
  - 将修改从working tree提交到stage（暂存）
  - `git add .`：提交所有文件进stage，考虑gitignore
  - `git add *`：提交所有文件进stage，忽略gitignore
  - 可直接在vscode中点`+`号
- `git commit -m "<commit信息>"`
  - `git commit`将修改提交到本地库
  - 为从working tree到stage的每次提交写commit 信息，加双引号代表这是1个参数
  - 更推荐直接在vscode的左上角文本框中写commit信息
- `git diff`：比较工作区和暂存区的区别

#### 从stage到remote

- `git pull upstream <分支名> --rebase`
  - 从upstream下载远端文件，并与本地已有文件采用rebase方式比较
  - 需要手动解决冲突
    - vscode可比较代码不同的地方，并选择采纳哪个
    - `git rebase --abort` #终止变基
    - `git rebase --continue` #解决完冲突后使用
  - **每次干活前和每次准备提交前都需要rebase**
- `git status` #查看暂存区状态
- `git diff <文件地址>` #查看文件修改内容
- `git log` #查看别人的提交日志
- `git reset`
  - 用于回退到之前某个分支，本地克隆中该分支之后的所有修改均被舍弃
  - `git reset <commit-id>` #reset到指定id的commit
  - 更推荐的是直接在vscode中使用插件查看分支生长情况，决定reset到哪个分支
  - reset分为soft和hard，一般选soft
- `git push <远端仓库类型> <分支名>`
  - `git push -u origin <分支名> ` #push到origin中新创建的分支
  - `git push orgin <分支名> -f` 
    - 强制push，即将远端仓库删光，再拷贝本地仓库
    - 不要乱用，一般只用于reset到远端旧版本后的push


## 其它

- git代理：
  - 查看已设置的代理：`git config --global --list`
  - 添加代理：`git config --global [代理地址]`
- 

