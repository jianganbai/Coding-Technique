# hydra学习笔记

- 从yaml中自动读取超参数

## 例子

- 文件结构
  
  - ```shell
    conf/
        mnist/
            mnist.yaml  # 子参数文件
        config.yaml  # 主参数文件
    main.py  # 函数文件
    ```

- 超参数配置
  
  - conf/config.yaml
    
    - ```yaml
      defaults:
          - files: mnist  # 从config.yaml所在文件的files子子文件夹中的mnist.yaml中读取
      paths:
          log: ./runs/
          data: ${hydra:runtime.cwd}/../data/raw  # hydra会用当前工作目录替换${hydra:runtime.cwd}
      ```
  
  - conf/mnist/mnist.yaml
    
    - ```yaml
      train_data: train.gz
      test_data: test.gz
      ```

- dataclass定义：方便进行数据类型检查
  
  - 在专门的py文件中定义dataclass，再import进来
  
  - ```python
    # config.yaml
    from dataclasses import dataclass
    
    @dataclass
    class Paths:
        log: str
        data: str
    @dataclass
    class Files:
        train_data: str
        test_data: str
    @dataclass
    class MNISTConfig:
        paths: Paths
        files: Files
    ```

- 调用: `main.py`
  
  - ```python
    import hydra
    from hydra.core.config_store import ConfigStore
    from config import MNISTConfig
    
    cs = ConfigStore.instance()  # 不加这两行，则main函数里会将cfg赋值为MNISTConfig类
    cs.store(name="mnist_config", node=MNISTConfig)
    
    # 使用conf/config.yaml作为主参数文件
    # 在装饰器里，hydra自动读入超参数
    @hydra.main(config_path="conf", config_name="config")
    def main(cfg: MNISTConfig):
        print(cfg.files.train_data)  # 访问超参数
        print(cfg.paths.log)
    
    main()  # 不将cfg作为输入参数
    ```
