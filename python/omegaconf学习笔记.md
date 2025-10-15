# omegaconf学习笔记

- Hierarchical configuration
  - 统一list / dict / yaml类超参数的读取接口
  - 变量替换、合并配置

## 读取

```python
from omegaconf import OmegaConf

conf = OmegaConf.create(dict(k='v', list=[1, dict(a='1', b='2')]))  # 从Dict读取
conf = OmegaConf.create([1, dict(a=10, b=dict(a=10))])  # 从list创建
conf = OmegaConf.load('source.yaml')  # 从yaml创建

# 从dot-list创建
dot_list = ["a.aa.aaa=1", "a.aa.bbb=2", "a.bb.aaa=3", "a.bb.bbb=4"]
conf = OmegaConf.from_dotlist(dot_list)  # 以'.'区分层级

# 从命令行创建
import sys
sys.argv = ['your-program.py', 'server.port=82', 'log.file=log2.txt']
conf = OmegaConf.from_cli()  # 保存server.port和log.file 2个key
```

## 访问&赋值

- OmegaConf支持多种访问方式
  
  - ```python
    conf.server.port  # 按对象方式访问
    # 若缺失，则弹出omegaconf.MissingMandatoryValue
    conf['server']['port']  # conf.server为dict，按dict方式访问
    # 同理可用.get()方式访问
    conf.client[0]  # conf.client为list，按list方式访问
    ```

- 赋值
  
  - ```python
    conf.server.hostname = "localhost"  # 若缺失，则作为新key
    conf['server']['hostname'] = "localhost"
    conf.database = {'hostname': 'database01', 'port': 3306}  # 添加新dict
    ```

- 表示
  
  - ```python
    OmegaConf.to_yaml(conf)  # 转为yaml
    OmegaConf.to_yaml(conf, resolve=True)  # resolve=True则进行变量替换
    OmegaConf.to_container(conf) # 转为dict
    ```

## 变量替换

- Variable interpolation：1个变量与是另1个变量的函数
  
  - ```yaml
    server:
      host: localhost
      port: 80
    client:
      url: http://${server.host}:${server.port}/  # 访问conf.client.url时，会将${server.host}替换为相应变量
      server_port: ${server.port}
      description: Client of ${.url}
    ```
  
  - ```python
    # 多重替换
    cfg = OmegaConf.create(
        {
            'plans': {'A': 'plan A', 'B': 'plan B'},
            'selected_plan': 'A',
            'plan': "${plans[${selected_plan}]}",  # 先替换key，再替换dict[key]
        }
    )
    print(cfg.plan)
    ```

- 使用环境变量替换
  
  - ```yaml
    user:
      name: ${oc.env:USER}  # 使用运行时'USER'的环境变量替换
      home: /home/${oc.env:USER}
      passwd: ${oc.env:DB_PASSWORD, null}  # 若环境变量中没有'DB_PASSWORD'，则赋值为None
    ```

- 提取string中的python数据
  
  - ```python
    # oc.decode:将后面的string转为相应变量
    cfg = OmegaConf.create(
        {
            'database': {
                'port': "${oc.decode:${oc.env:DB_PORT}}",
                'nodes': "${oc.decode:${oc.env:DB_NODES}}",
                'timeout': "${oc.decode:${oc.env:DB_TIMEOUT,null}}",
            }
        }
    )
    os.environ["DB_PORT"] = '3308'
    os.envrion["DB_NODES"] = '[host1, host2, host3]'
    ```
  
  - 可转的数据类型: bool, int, float, dict, list, None

- 自定义变量替换
  
  - ```python
    OmegaConf.register_new_resolver('plus_10': lambda x: x + 10)
    conf = OmegaConf.create({'key': '${plus_10:990}'})  # 调用plus_10函数，处理990 
    ```

- `II`：引用尚未解析的配置
  
  - ```python
    from omegaconf import OmegaConf, II
    
    config = OmegaConf.create({
        "database": {
            "host": "localhost",
            "port": 5432,
            "url": II("database.host:database.port")
        }
    })
    print(config.database.url)  # 输出: localhost:5432
    
    config = OmegaConf.create({
        "x": 10,
        "y": 20,
        "sum": II("x + y")
    })
    OmegaConf.resolve(config)  # 解析所有配置
    print(config.sum)  # 输出: 30
    ```

- 说明
  
  - OmegaConf 仅在调用时才进行变量替换（如`conf.a`），刚读入/创建时不会
  
  - `OmegaConf.to_container(conf, resolve=True)`：强制进行变量替换

## 合并

- 合并每个模块的配置文件
  
  - ```python
    # 直接合并
    conf = OmegaConf.merge(base_cfg, model_cfg, optimizer_cfg, dataset_cfg)
    
    # 与命令行参数合并
    sys.argv = ['program.py', 'server.port=82']
    conf.merge_with_cli()
    ```
  
  - ```python
    from omegaconf import ListMergeMode
    
    conf = OmegaConf.merge(base_cfg, model_cfg, list_merge_mode=ListMergeMode.EXTEND_UNIQUE)
    # REPLACE：全部替换
    # EXTEND：全部作为新变量
    # EXTEND_UNIQUE：仅将没有的变量作为新的
    ```

## 保存

```python
from omegaconf import OmegaConf
cfg = OmegaConf.load("config.yaml")
OmegaConf.save(cfg, "saved_config.yaml")
```



## 其它

- yaml语法
  
  - 以换行`\n`为分割号，`:`表示dict，`-`表示list
    
    - `:`和`-`后均需空格
  
  - 所有变量小写，不需要加引号
  
  - 可表示list
    
    - ```yaml
      a:
        - 1  # 以2个空格区分层级
        - 2
      # 读取后：{'a': [1, 2]}
      ```
