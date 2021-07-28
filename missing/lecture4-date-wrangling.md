# 数据清洗

- `data wrangling`：对数据的各种操作，如转化格式、统计分布等

- 例子：查询服务器

  - 查询服务器访问信息：

    - ```bash
      ssh tsp 'journalctl | grep ssh | grep "Disconnected from" | less
      ```

    - `tsp`对应服务器，`ssh`是1种连接方式

    - `'journalctl | grep ssh | grep "Disconnected from"'`是在服务器的命令行中执行该shell指令

      - `journalctl`：返回系统日志
      - `grep ssh | grep "Disconnected from"`：从系统日志中搜索2个关键词

# 常用命令

- `grep`：常用于查找文件中的内容
- `sed`：常用于字符串匹配替换
- `awk`：常用于整合字符串、匹配字符串

## `sed`：字符串匹配

- **匹配并替换**：s参数
  - 可使用`cat`，`echo`等指令将字符串读入命令行
  - 服从正则匹配表达式`Regular Expression`
  - `sed 's/.*Disconnected from/[替代字符为空格]/'`：搜索所有以`Disconnected from`结尾的字符串，然后用空格替代
    - `s`代表substitute，匹配时不重叠
    - `/`用于分开参数，上表达式中有2个参数`.*Disconnected from`和`[空]`
    - `cat ssh.log | sed -E 's/.*Disconnected from//'`：读取`ssh.log`，匹配并替换，再在命令行中显示替代后的结果
  - `sed -E 's/(ab)*//g'`
    - `-E`表示用更新的正则匹配规则
    - `g`代表前面的正则匹配表达式可在多个位置使用





