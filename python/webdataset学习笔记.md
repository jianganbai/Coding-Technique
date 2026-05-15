# WebDataset 学习笔记

## 原理

### 核心

- 将数据打包成shard，训练时随机选shard，用顺序读取近似随机读取
  
  - 按样本随机读取太慢 => 将一些样本打包成shard
  
  - 训练时随机选shard，再整体读入shard，实现顺序读取
  
  - 顺序读取的速度往往比随机读取的速度快

- 2层shuffle近似随机，效果足够好
  
  - shuffle 1：每个epoch随机shuffle shard列表，读shard顺序不同
  
  - shuffle 2：每个shard内部shuffle，通过buffer实现
  
  - 加入新数据：将新数据打包成新shard，添加到shard list。不需要更改旧shard

### tar

- tar格式：流式文件，可边下载边处理

## 使用

### 基本读写

```python
# write
with wds.writer.TarWriter(tar_file) as sink:
    for path in wav_files:
        with open(path, 'rb') as f:
            wav_bytes = f.read()
        sample = {
            '__key__': os.path.splitext(os.path.relpath(path, top_dir))[0],  # 不能带后缀，否则wav键就保存成wav.wav了
            'wav': wav_bytes  # 保存的是wav的比特流
        }
        sink.write(sample)

# read
dataset = wds.WebDataset(tar_file).shuffle(10).map(
    lambda sample: {
        'audio_info': soundfile.read(io.BytesIO(sample['wav']), dtype='float32')
    }
)
for sample in dataset:
    wav, sr = sample['audio_info']
    print(sample['__key__'])
```

### ShardWriter

```python
with wds.writer.ShardWriter(
    pattern=os.path.join(save_dir, 'shard-%05d.tar'),  # shard命名格式为shard-00001.tar格式
    maxsize=shard_size_mb * (1024 ** 2),  # 控制maxsize或者maxcount
    compress=True,  # 是否使用gz进行压缩
) as sink:
    for _, row in all_df.sample(frac=1).iterrows():  # shuffle
        with open(row['path'], 'rb') as f:
            wav_bytes = f.read()
        sample = {
            '__key__': os.path.splitext(row['file'])[0],
            'wav': wav_bytes
        }
        sink.write(sample)
```
