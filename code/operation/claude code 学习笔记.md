## 背景

- Vibe Coding：人提需求，大模型修改、调用工具、执行

- MCP: Model Context Protocol
	- 大模型操作外部工具的标准化协议

- 2种操作形式
	- CLI：命令行，`claude` 从shell切换至claude code cli
		- `claude -c`：从上次对话开始
	- IDE：vscode 插件

## 应用

### 通用

- 3种模式：shift+tab 切换模式
	- Plan Mode：给计划，不修改
	- 每次Edit询问
	- 自动Edit
  
  - 所有模式默认执行命令前询问
	  - `claude -c --dangerously-skip-permissions`：执行命令前不再询问

### CLI

- `claude`：从cmd进入claude code
	- `claude "问题"`：直接提问
	- `cat file | claude "..."`：管道输入，处理文件内容
	- `claude -c`：继续当前目录的上一次聊天
	- `claude --resume`：打开当前目录下的聊天记录
- 交互操作
	- ESC 中止当前执行
	- 行尾输入反斜杠`\`，然后按Enter换行
		- shift + enter 会直接执行

### /

- 可在claude code cli 中调用多种内置功能

#### 核心对话

- `/new`：开启新会话

- `/resume`：读取上一次会话

- `/config`：打开claude code的配置

- `/rewind`：回滚到之前对话
	- 可选择hard reset 或者soft reset
	- 已执行的操作无法回滚
	- 建议还是用git 做版本控制

- `/compact`：压缩当前会话的上下文
	- `/compact [prompt]`：说明压缩需求

- `/plan`：查看最新的plan

- `/clear`：清空上下文

- `/review`：代码审查

- `/exit`或`/quit`：退出

- `/help`：查看帮助

#### 状态

- `/status`：查看Claude状态

- `/tasks`：查看后台任务

- `/cost`：查看消费账单

- `/model`：选择模型

- `/context`：查看上下文占用情况

- `/doctor`：系统自检

#### CLAUDE.md

- CLAUDE.md：记录用户需求、项目信息、to-do list等
	- 新会话可继承旧会话的记录
	- 其他合作者可看到，他们使用claude code时可按同样标准处理

- `/init`：让claude code 自行生成CLAUDE.md
	- 后续直接用自然语言告诉cc更新

- `/memory`：用户手动修改CLAUDE.md
	- 2种：用户级别，项目级别
	- 项目级别可通过git 共享给其他合作者

- `/hooks`：设置自动执行
	- 例：代码美化，符合flake8要求
	- 可共享在项目git中，供所有合作者检查

#### Skills

- `/skills`：添加Agent Skills
	- 例：写每日总结
	- 与主会话共享上下文

#### SubAgent

- `/agent`：与`skills`相似，但有独立上下文，不会让主会话上下文变得很臃肿
	- 例：设置code reviewer

## 其它

- `@文件路径`：引用文件
