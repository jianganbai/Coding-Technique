# subprocess学习笔记

- 在python程序运行外部命令和程序
  
  - 比`os.system`更强大，提供了子进程的更多信息

## subprocess.run

- 推荐使用

```python
import subprocess

# 基本用法
result = subprocess.run(['ls', '-l'], capture_output=True, text=True)
print(result.stdout)

# 带参数的高级用法
result = subprocess.run(
    ['echo', 'Hello, World!'],
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE,
    text=True,
    check=True  # 如果命令返回非零退出码，抛出 CalledProcessError
)
```

- 参数
  
  - `stdin`: 标准输入 (PIPE, 文件对象等)
  
  - `stdout`: 标准输出 (PIPE, 文件对象等)
  
  - `stderr`: 标准错误 (PIPE, STDOUT, 文件对象等)
  
  - `shell`: 是否通过 shell 执行命令 (谨慎使用)
    
    - `shell=False`：subprocess直接调用程序
    
    - `shell=True`：通过shell来解释，尽量不用
  
  - `cwd`: 设置工作目录
  
  - `env`: 设置环境变量
  
  - `timeout`: 超时时间(秒)
  
  - `check`: 如果为 True，非零退出码会引发异常
  
  - `text`(或 `universal_newlines`): 以文本模式处理输入输出

## subprocess.popen

- 更底层的调用

```python
# 启动进程
process = subprocess.Popen(['ping', '-c', '4', 'example.com'], 
                          stdout=subprocess.PIPE,
                          stderr=subprocess.PIPE,
                          text=True)

# 等待进程完成并获取输出
# process.wait()阻塞主进程至子进程结束
stdout, stderr = process.communicate()  # 等待子进程结束，同时处理子进程的输入/输出
print(stdout)

# 检查返回码
if process.returncode == 0:
    print("Command succeeded")
else:
    print(f"Command failed with code {process.returncode}")
```

- 高级使用
  
  - ```python
    # 管道连接多个命令: ps aux | grep python
    ps = subprocess.Popen(['ps', 'aux'], stdout=subprocess.PIPE)
    grep = subprocess.Popen(['grep', 'python'], stdin=ps.stdout, stdout=subprocess.PIPE)
    ps.stdout.close()  # 允许 ps 收到 SIGPIPE 如果 grep 退出
    output = grep.communicate()[0]
    print(output.decode())
    ```
  
  - 
