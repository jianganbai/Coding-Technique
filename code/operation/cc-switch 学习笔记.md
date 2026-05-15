- 快速调整Claude Code 的接入模型
## CLI

### 安装

- 将`.sh`放在`~/.local/bin/`下'

- 在`~/.bashrc`中添加`export PATH="$HOME/.local/bin $PATH"`，则仅为用户安装
### 常用

- `cc-switch provider add`：添加供应商
  
  - `--app claude`; `--app codex`; `--app gemini` 选择配置哪个工具

- `cc-switch provider list`：展示所有供应商

- `cc-switch provider switch [供应商id]`：切换供应商，供应商id 来自于list

- `cc-switch provider edit [供应商id]`：修改供应商配置

- `cc-switch provider delete [供应商id]`：删除供应商
