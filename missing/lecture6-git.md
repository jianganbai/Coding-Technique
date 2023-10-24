# Git版本控制

- 以下命令既可以在shell/cmd中跑，也可在git bash中跑

## git原理

- 文件系统

  - `directory=>tree`；`file=>blob`

  - 每次修改后的文件称为状态`state`

    - 不同开发路径$\Rightarrow$从1个节点拷贝出分叉
    - 多功能合并$\Rightarrow$多个分叉合并`merge`
    - 每个节点带有提交者、提交说明等信息
      - 实际采用哈希：`anything=>string`

  - ```c
    type commit = struct {  // 每次提交的数据结构
        parents: array;
        author: string;
        message: string;
        snapshot: tree;  // 开发流程树
    }
    ```

- git存储区
  - `working zone`工作区
  - `staged zone`暂存区：git add后
  - `repository`：git commit后

## git语法

### git分支相关

- `git branch`：展示所有分支名
  - `git branch -vv`：展示各个分支的一些信息
  - `git branch -r`：查看远端仓库的所有分支
  - `git branch [name]`：创建名为name的分支
  - `git branch -d [分支名]`：删除本地已合并的分支，若未合并，将`-d`替换为`-D`强行删除
- `git checkout [name]`：切换进名为name的分支
  - `git checkout -b [name]`：创建名为name的分支并切换进
  - `git checkout [hash]`：输入某个节点的hash值，切换至该分支
    - hash值不需要输入全，可输入前缀
  - `git checkout [branch_name]`：输入某个节点的名字，切换至该分支
    - 可查看之前的文件，但不能修改
  - `git checkout [<commit>] -- [<path>]`：用指定commit的`<path>`路径下的文件，覆盖当前工作区对应文件（不建议使用）
    - 若未指定`<path>`，则用指定commit全部覆盖当前工作区
    - 若未指定`<path>`，则用暂存区覆盖
  - `checkout`将`HEAD`指针移动到某个分支
- git查看历史
  - `git log`：给出文字版的提交记录
    - `git log --all --graph --decorate`：更好看的文字提交记录
    - `git log --all --graph --decorate --oneline`：显示继承关系
  - `git diff`：对比未提交与已提交
    - `git diff [file_name]`：查看某文件的提交记录
    - `git diff [hash] [file_name]`：查看某文件在hash分支与当前分支的区别
    - `git diff <本地分支名>`：比较本地当前分支与本地另一分支的区别
    - `git diff origin/<远程分支名>`：比较本地当前分支与远程分支的区别
- `git merge`：合并分支（两支合并）
  - `git merge [name]`：将名为name的分支合并入当前head分支
  - **Merge Conflict**：一般是由于语法
    - `git merge --abort`：返回merge前的状态
    - 出现冲突标识`<<`，`==`，`>>`
    - 修改时直接delete文件中的标识，然后手动更改代码
    - `git merge --continue`：继续合并
- `git rebase`：变基合并
  - `git rebase master`：将feature支（当前处于）拼接在master支后面，仍为master支
  - 优点：没有分支，结构清晰；缺点：commit顺序不再按时间排序
    - 不要在共享的分支上rebase，否则pull同一个分支的都需要重新pull

- `git cherry-pick`：选几个commit，拼接到当前分支
  - `git cherry-pick <hashA> <hashB> <hashC> ...`：将选定的多个commit拼接到当前分支上
  - 需要逐文件检查冲突


### git本地缓存

- `git stash`
  - `git stash`：将没有commit的代码移除至栈中，将文件回退到上一次提交的状态
  - `git stash pop`：将之前缓存到栈中的代码再复原回文件中
- `git restore`：丢弃unstaged的修改
  - `git restore [file]`：丢弃file中的修改

### git提交相关

- 创建git
  - `git init`：在当前文件夹内创建git
  - `git help ""`：查看某命令的使用规则

#### 存入暂存区

- `git add`：将修改的文件存入`staged zone`
  - `git add .`：stage全部unstaged文件
  - `git add [file]`：stage某个文件
    - 支持*匹配多个文件
  - `git add -p [file]`：对修改的代码，逐个决定是否stage
    - `s=split`：将多个修改分开决定是否stage；`y=yes`：stage这行；`n=no`：unstage这行
  - `git add -u .`：仅提交已被追踪的文件进stage，不提交新文件
- 暂存错了
  - `git reset`
    - `git reset HEAD`：完全回退到git add前
    - `git reset HEAD [不想提交的文件]`
  - `git restore`
    - `git restore --stage [不想提交的文件]`：将文件从stage区移出，但不修改文件

#### 推入远程仓库

- `git commit`：将`staged zone`提交至库中

  - `git commit`：会打开vim编辑提交信息
  - `git commit -m "xxx"`：直接将xxx作为提交信息

  - `git checkout`：切换进别的分支/之前提交记录

### git多人合作

- `git remote`
  - `git remote`：展示所有与本仓库关联的仓库
  - `git remote add [name] [url]`：添加地址为url的仓库作为远程仓库，为其取别名为name（origin，upstream等）
- `git push [remote name] [local branch name]:[remote branch name]`
  - `remote name`：远程仓库的别名；`local branch name`：要提交的本地分支名；`remote branch name`：远程仓库的分支名
    - `git push origin main`默认将本地的main分支提交到远端的main分支
  - **每次修改都需要提交到本地库，但不一定要提交到远端**
  - 每次输入仓库名态麻烦，可简化
    - `git branch --set-upstream-to=origin/master`：之后直接默认推送到该地址
  - 删除远端分支：`git push :[remote branch name]`
- `git blame`：查看谁提交了这条代码
  - `git blame [file]`：查看这个文件的修改记录，每个修改编码为哈希值
  - `git show [hash]`：查看哈希值为hash的修改的内容

### git下载相关

- `git fetch [remote]`：在本地创建1个新分支，并将远端分支下载到该分支
- `git pull`：`git fetch`+`git merge`
  - 下载远端分支+将下载结果与本地结果合并
  - `git pull origin [远程分支]:[本地分支]`：将远程指定分支拉取到本地指定分支
  - `git pull origin [远程分支]`：将远程指定分支拉取到本地当前分支
- `git clone`：将远程仓库下载到本地
  - `git clone [url] [folder name]`：仓库地址为url，下载仓库到folder name
  - `url`可以是网络地址，也可以是本地路径
  - `git clone -b [远端某个分支] [仓库SSH]`：仓库SSH从绿色Code键里面找

## git文件

- `.gitignore`
  - 屏蔽规则
    - 直接写文件路径，则屏蔽该路径的文件
    - `*.jpg`：屏蔽所有jpg文件
  - 生效范围
    - 对没有commit的都生效（即使.gitignore还没有提交）
    - 若被屏蔽的文件已经提交，则必须在本版本下先删除该文件，才能提交

## git自定义

- `.gitconfig`文件记录了git配置信息，修改该文件可进行git自定义

