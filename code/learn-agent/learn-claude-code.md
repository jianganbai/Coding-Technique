## 基础概念

- Agent = Model + Harness
	- Model：大模型，提供分析&推理能力
	- Harness：大模型交互环境
		- Harness = Tools + Knowledge + Observation + Action Interfaces + Permissions
	- LLM是通用的、专属于大公司的，但harness是针对领域的、个体可开发的

- 概念
	- skill：文本形式描述的方法论，告诉模型分析思路
	- tool：可供模型调用的工具，属于环境
	- MCP: Model Context Protocol，模型和工具交互的标准接口

- 经验
	- 信任模型，关注harness
	- 不要让模型按固定框架/规则执行，而是让模型自己学出来

## Loop

- Loop：循环进行“感知-思考-行动”，自主解决问题
	- 针对用户的每个输入，循环进行思考和Tool调用，直到超过步数上限or不再调用Tool
	- 1次Loop完成当前所有tool use，再告诉用户

- Tool
	- bash命令千奇百怪，易隐藏有害命令 => 针对常用操作，编写专门的Tool
		- 例如：读文件、写文件
	- 路径沙盒：模型每次请求访问路径时，审查该路径是否逃离工作区
	- 通过`{Tool Name: Tool API}`的dispatch map，告诉模型如何调用
		- 新工具只需要加入这个map，不需要改循环

## 规划与知识

- Todo Write：独立的Todo提醒，防止因上下文过长导致遗忘任务
	- 设立独立于模型的Todo Manager，要求模型给出Todo并记录
		- Todo Manager本质上也是个Tool
	- 控制模型每次只执行1个Todo，并要求每次执行完更新Todo
	- 若模型连续3次未更新Todo，则提醒模型


- Subagent：大任务拆小，让子Agent处理子任务，防止父Agent陷入非核心上下文
	- 子任务仅占用子Agent的上下文。主Agent仅需要知道该任务的执行结果
	- 仅父Agent能创建task（创建子Agent）

- Skill：按需加载
	- Skill定义：在`.skills/`下创建`<skill_name>/SKILL.md`
	- system prompt仅列出所有skill的名称
		- 不提前加载所有skill的描述
		- 处理任务时，需要哪个skill，再现场加载相关skill的描述
			- skill加载也是1个tool

- Context Compact：3层压缩上下文
	- Micro Compact：清除3轮之前的tool result，仅记录当时的tool名称
	- Auto Compact：将所有上下文写入内存，然后让模型对整个context做总结
		- 当上下文长度超过阈值时，自动压缩
	- Manual Compact：用户通过`/compact`要求压缩
		- 压缩方式同Auto Compact

## 持久化

- Task System：任务持久化
	- 将待做任务记录在硬盘，而非Todo Manager（内存）或上下文（显存）
	- Todo Manager：简单记录任务，防止模型遗忘
		- 无任务执行顺序、依赖关系
	- Task System：任务动态调度
		- 每个任务以JSON形式保存在硬盘，记录任务的状态、顺序、依赖
		- Task System再根据JSON，调度任务
			- 什么可以做、什么被卡住、什么做完了

- Background Tasks：不要让慢任务拖Agent后腿
	- 将工具调用任务，以子进程提交并执行
	- 子进程执行完后，将结果提交给通知队列
	- 每次调用模型前，先读取通知队列，将执行结果加入prompt

## 团队

- Agent Teams：主Agent管理 + 信箱通信
	- 3个需求
		- 子Agent持久化：subagent完成任务后就结束
		- 身份和生命周期管理
		- Agent之间通信
	- 主Agent管理队员Agent
		- 设立Teammate Manager，记录所有创建的子Agent
			- 定义、身份、状态：保存在硬盘的config.json，持久化
		- 身份重注入：更改Agent身份（通过信箱机制）
	- 信箱机制进行Agent间通信
		- 每个Agent都有信箱，以JSONL形式记录所有消息
