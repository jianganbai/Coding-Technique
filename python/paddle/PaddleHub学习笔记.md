# PaddleHub学习笔记

## Hub serving

- 本地提交数据，服务器进行预测

- 例：hub serving进行句子分割

  - 开启服务：`hub serving start -m lac`

    - 参数：<img src="../imgs/Hub_Serving_Params.png" style="zoom:80%;" />

  - 发送数据：

    - ```python
      # coding: utf8
      import requests
      import json
      
      if __name__ == "__main__":
      
          text = ["今天是个好日子", "天气预报说今天要下雨"]
          data = {"texts": text, "batch_size": 1}
          url = "http://127.0.0.1:8866/predict/lac"
          headers = {"Content-Type": "application/json"}
      
          r = requests.post(url=url, headers=headers, data=json.dumps(data))
      
          # Print results
          print(json.dumps(r.json(), indent=4, ensure_ascii=False))
      ```

  - 结束服务：`hub serving stop --port 8866`


## Hub命令

- 安装相关：
  - `hub install`与`hub uninstall`
  - `hub download xxx`：下载xxx模块
- 显示已安装模块相关：
  - `hub show xxx`：展示xxx模块的信息
  - `hub list`：战士所有本地安装的模块

- 运行相关：
  - `hub run`：运行模块
    - 常用格式：`hub run xxx --input_texts ""`
    - 部分模型不可使用`hub run`，需要通过`hub.Module`封装

## Hub.Module

- 一般流程：通过hub.Module加载预训练网络，加载数据集，设置优化器，创建hub.Trainer，运行trainer.train
- 

