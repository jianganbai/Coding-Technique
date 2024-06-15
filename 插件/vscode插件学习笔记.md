# vscode插件学习笔记

## git

### git graph

- 适合查看commit tree
- 在git graph里比较2个commit
  - 先点击源commit，然后按住ctrl，点击目标commit
  - 源commit里就显示对比结果

### git history

- 适合查看、对比之前的commit
- 查看整体commit tree
  - <img src="imgs/image-20240504151609379.png" alt="image-20240504151609379" style="zoom:20%;" align="left"/>
  - more里，可选择对比2个commit
- 查看单个文件的修改记录
  - 选中文件，右键，Git: view file history

### gitlens

- 大而全的插件，高级功能需要付费
- 每行git blame：自动显示每行上次是谁在哪个commit修改
- 功能集成在vscode git 面板里
  - <img src="imgs/image-20240504151841550.png" alt="image-20240504151841550" style="zoom:80%;" align="left"/>
  - 查看当前修改
  - 比较2个commit: select for compare, compare with selected
  - interactive rebase：交互式选择rebase哪些修改
- 右上角比较不同commit
  - <img src="imgs/image-20240504152059744.png" alt="image-20240504152059744" style="zoom:15%;" align="left"/>
  - alt+点击，可选择比较哪个commit（默认是上一个）
- 切换显示
  - <img src="imgs/image-20240504152218572.png" alt="image-20240504152218572" style="zoom:70%;" align="left"/>
  - toggle file blame：显示每行是谁修改的
  - toggle file heatmap：代码左侧显示热力图，最近修改的为暖色
  - toggle file change：显示修改了什么

