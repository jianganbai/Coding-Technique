# 文本编辑器

- 常用编辑器：`vscode`（图形界面）+`vim`（命令行）

## vim读写状态

- 状态
  - `normal mode`：原始状态（只读），按`esc`进入
  - `insert mode`：插入写入文件，按`i`进入
  - `replace mode`：覆盖写入文件，按`r`进入
  - `visual mode`：选中范围内字符，按`v`进入
    - `-line`：按`shift+v`进入
    - `-block`：按`ctrl+v`进入
  - `command line mode`：决定文件保存方式等，按`:`进入

## vim内置命令行操作

- 先进入`command line mode`
- `q(quit)`：退出
  - `q`实际是退出vim窗口，若之前是最后1个窗口，则退回至命令行
  - `qa`：退出全部窗口
- `w(write)`：保存
  - `wq`：保存并退出
  - `w!`：强制保存，不退出vim
  - `wq!`：强制保存文件，退出vim
  - `q!`：不保存，强制退出
- `sp`：在2个窗口中显示该文件（实时显示修改）
- `help :[w]`：查找输入`:w`是什么意思

## vim查看

- 先进入`normal mode`
- **光标移动**
  - `h`：向左；`l`：向右；`j`：向下；`k`：向上
  - `w`：逐单词向后移；`e`：逐单次向后移并停留在单词末尾；`b`：逐单词向前移
  - `0`：移动至该行最左侧；`$`：移动至该行最右侧；`^`：移动至该行左侧第1个不是空格的字符
  - `G`：移动至文件末尾；`gg`：移动至文件开头
  - `L`：移动至屏幕内的最后1行；`M`：移动至屏幕内的中间1行；`H`：移动至屏幕内的第1行
  - `f[字母]`：移动到该行第1个该字母位置
  - 可在移动命令前加数字，表示按此方式移动多少次
    - 如`4e`表示做4次移动至单词末尾
- **增删**
  - `d[移动方式]`：删除移动过的任何字符
    - `dw`：删除这个单词（全部or后半部分）
    - 也可以进入按`v`进入visual mode，标出要删除的字符，再按`d`
  - `c[移动方式]`：与`d`基本一致，但最后会处于`insert mode`
    - `ci[`：删除[]内的所有内容（change in [）
    - `ca[`：删除[]及其内部的所有内容
  - `u`：撤销；`ctrl+r`：取消撤销
- **复制粘贴**
  - `y`：复制
    - `y[移动方式]`：复制移动路径上的字符
      - `yw`：复制整个单词（前提是光标在单词开头）
    - 整块复制
      - 按`v`进入`visual mode`，移动光标时自动选中移动过的字符，再按`y`复制
      - 按`shift+v`进入整行选择，移动光标时自动选中移动过的行，再按`y`复制
      - 按`ctrl+v`进入矩形选择
  - `d`：剪切
  - `p`：在光标后粘贴
- **搜索**
  - `/[key]`：搜索文件中名为key的字符串，光标会实时显示
    - 按`n`调转至下一个匹配位点
