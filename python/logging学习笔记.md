# logging学习笔记

- 基本概念
  - logger：程序使用的接口
  - handler：将日志转发给流/文件
  - filter
  - formatter：设定日志格式

## 单记录器

- ```python
  import logging
  logging.basicConfig(filename='example.log', level=logging.DEBUG)
  logging.debug('This message should go to the log file')
  logging.info('So should this')
  logging.warning('And this, too')
  ```

- `logging.basicConfig(filename, filemode, format, level)`
  
  - filename: 要写入的日志文件
  - filemode: 'w'为覆盖写入，'a'为追加写入
  - format: 日志格式
    - `%(标识)s`：`%(asctime)s - %(levelname)s - %(message)s`
      - levelname: 日志级别
      - message: 信息
      - asctime: 时间
  - level: 写入日志的最低级别
    - `logging.DEBUG, logging.INFO, logging.WARNING`

- 仅针对单记录器
  
  - 无法同时写入日志和输出流

## 多记录器

- logger：程序使用logger来发送日志
  - 配置
    - `logger1 = logging.getLogger(记录器名称)`：创建logger
    - `logger1.setLevel(级别)`：设置最低级别
    - `logger1.addHandler(handler)`：为logger添加handler
      - 1个logger可添加多个handler，不同logger可添加相同handler
    - 多logger：可由不同来源生成日志
  - 发送
    - `logger.debug(消息)`, `logger.info(消息)`, `logger.warning(消息)`等等
    - 配置好logger的handler、handler的formatter，就可直接使用logger创建消息
- handler：1个handler对应1种日志去向
  - `handler1 = logging.StreamHandler()`：日志打印在屏幕上
  - `handler2 = logging.FileHandler(filename, mode)`：日志写入文件
    - mode='w'：覆盖写入；mode='a'：追加写入
  - `handler.setLevel(级别)`：若logger的消息不低于最低级别，则该handler处理
    - 1个logger可有多个handler，不同handler的写入级别可不同
  - `handler.setFormatter(formatter)`：为handler设置消息格式
- formatter：将handler的消息，格式化写入日志
  - `formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')`
    - asctime是时间，name是logger名字，levelname是日志级别，message是消息
