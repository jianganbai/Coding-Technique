# HuggingFace学习笔记

- HuggingFace：将PLM封装为易调用的接口

## pipeline

- 最高级的封装，扩展性差
  
  - 提供了9个常见任务
  
  - ```python
    from transformers import pipeline
    
    # feature-extraction (get the vector representation of a text)
    # fill-mask
    # ner (named entity recognition)
    # question-answering
    # sentiment-analysis
    # summarization
    # text-generation
    # translation
    # zero-shot-classification
    
    classifier = pipeline("sentiment-analysis")  # 情感分析，下载默认的模型
    classifier(["I've been waiting for a HuggingFace course my whole life.",
                "I hate this so much!"])
    
    unmasker = pipeline("fill-mask")  # 填入<mask>，给出最可能的top_k个
    unmasker("This course will teach you all about <mask> models.", top_k=2)  # 填上<mask>，给出最可能的top_k个
    ```

- 预训练参数默认保存在`C:\users\xxx\.cache\huggingface`或`~/.cache/huggingface`

## Auto

### AutoTokenizer

- 将句子转化为token id序列，并加入特殊token
  
  - ```python
    from transformers import AutoTokenizer
    
    checkpoint = "distilbert-base-uncased-finetuned-sst-2-english"  # 指定下载的模型
    tokenizer = AutoTokenizer.from_pretrained(checkpoint)
    raw_inputs = ["I've been waiting for a HuggingFace course my whole life.",
                  "I hate this so much!"]
    # padding：将1个batch补全至相同长度
    # truncation：将超长句子阶段至最长能接受长度
    # 'pt'代表返回pytorch tensor
    inputs = tokenizer(raw_inputs, padding=True, truncation=True, return_tensors='pt')
    # inputs
    # 返回的input_ids：token id序列，有补上[CLS], [SEP]等特殊token
    # 返回的attention_mask：padding结果
    ```

- 一步步拆解
  
  - ```python
    tokens = [tokenizer.tokenize(sentence) for sentence in sentences]
    ids = [tokenizer.convert_tokens_to_ids(token) for token in tokens]  # 不补零
    
    # 一个batch内的输入长度应当相同，pad至最长的，truncate超过输入上限的
    # 应同时输入attention mask，给出输入中哪些是padding
    ouput = model(all_ids, attention_mask=attention_mask)
    ```

### AutoModel

- 加载预训练模型，输出embedding
  
  - 加载的预训练模型**舍弃了只用于pre-train的参数，仅保留fine-tune会用到的**
  
  - 不包含后续分类头，适合直接用于inference
  
  - ```python
    from transformers import AutoModel
    
    model = AutoModel.from_pretrained(checkpoint)  # 从HuggingFace Hub选择model
    outputs = model(**inputs)
    print(outputs.last_hidden_state.shape)  # torch.tensor类型
    ```

### AutoModelForSequenceClassification

- 加载预训练模型，比`AutoModel`多引入了可训练的分类头，可输出分类头的
  
  - 适合fine-tune，只需在后面手动加Softmax和交叉熵
  
  - ```python
    from transformers import AdamW, AutoModelForSequenceClassification
    
    checkpoint = "distilbert-base-uncased-finetuned-sst-2-english"
    model = AutoModelForSequenceClassification.from_pretrained(checkpoint,
                                                               num_labels=2)
    outputs = model(**inputs)
    print(outputs.logits.shape)
    predictions = torch.nn.functional.softmax(outputs.logits, dim=-1)
    print(predictions)
    
    # 训练
    optimizer = AdamW(model.parameters())
    loss = model(**inputs).loss
    loss.backward()
    optimizer.step()
    ```

## Config

- config: 模型的所有超参数，如数据预处理、网络结构、训练超参数
  
  - 根据config能唯一创建出1个模型
  
  - ```python
    from transformers import BertConfig, BertModel
    
    bert_config = BertConfig.from_pretrained('xxx')  # 可自选哪个check_point
    bert_config = BertConfig.from_pretrained('xxx', num_hidden_layers=10)  # 可改结构·
    bert_model = BertModel(bert_config)
    bert_model.save_pretrained(pth)  # 保存为2个文件：config.json pytorch_model.bin
    
    bert_model.from_pretrained(local_pth)  # 读入存在本地的模型
    ```

- 使用模型
  
  - ```python
    import torch
    
    sequences = ["Hello!", "Cool.", "Nice!"]
    encoded_sequences = [[101, 7592, 999, 102],
                         [101, 4658, 1012, 102],
                         [101, 3835, 999, 102],]
    model_inputs = torch.tensor(encoder_sequences)
    output = model(model_inputs)
    ```

## datasets

- HuggingFace Hub提供了很多训练数据集
  
  - 可直接使用封装的`datsets`类下载并处理
  
  - ```python
    from datasets import load_dataset
    
    raw_datasets = load_dataset('glue', 'mrpc')  # 返回dict
    # raw_datasets['train'], raw_datasets.features, raw_datasets[0]
    ```
  
  - 数据集保存在disk中，仅需要时才读入内存

- 数据集预处理：例如tokenize
  
  - ```python
    def tokenize_func(example):
        return tokenizer(example["sentence1"], example["sentence2"], truncation=True)
        # 返回input_ids, attention_mask, and token_type_ids
    
    tokenized_datasets = load_dataset('glue', 'mrpc').map(tokenize_func, batched=True)
    # batched=True使用并行处理
    # map是对整个数据集预处理
    
    # 之后直接用**batch输入模型，所以要去掉所有不能输入模型的
    tokenized_datasets = tokenized_datasets.remove_column(['idx', 'sentence1', 'sentence2'])
    # pytorch只认key为labels的为标签，故需调整
    tokenized_datasets = tokenized_datasets.rename_column('label', 'labels')
    tokenized_datasets.set_format('torch')
    
    # 之后可直接使用DataLoader(tokenized_datasets)
    ```

- Dynamic Padding
  
  - 在预处理函数中pad慢
    
    - 需要固定长度的batch（例如TPU），则仍需要在预处理中做
  
  - 更快：先生成batch，再在DataLoader的collect_fn写pad
  
  - ```python
    from torch.utils.data import DataLoader
    from transformers import DataCollatorWithPadding
    
    data_collator = DataCollatorWithPadding(tokenizer)
    train_dataloader = DataLoader(tokenized_datasets['train'],
                                  batch_size=16, shuffle=True,
                                  collate_fn=data_collator)  # 在DataLoader才做pad
    ```

## Trainer

```python
from transformers import Trainer, TrainingArguments
from datasets import load_metric

# 默认使用全部可见GPU
# os.environ['CUDA_LAUNCH_BLOCKING'] = "1"
os.environ["CUDA_DEVICE_ORDER"] = "PCI_BUS_ID"
os.environ["CUDA_VISIBLE_DEVICES"] = "0,1,2"

training_args = TrainingArguments('保存训练超参数和checkpoint的地址')  # 设置超参数等
# per_device_train_batch_size, per_device_eval_batch_size
# per_device_train_batch_size, per_device_eval_batch_size, 默认ddp
# num_train_epochs, learning_rate, weight_decay
# evaluation_strategy='epoch' or 'steps'
# seed

# 固定主干网的参数，仅调classifier
for param in model.bert.parameters():  # 调试模式下查看网络架构，后端的分类网络是model.classifier
    param.requires_grad = False

# 验证集
def compute_metrics(eval_pred):
    logits, labels = eval_pred
    predictions = np.argmax(logits, axis=-1)
    return metric.compute(predictions=predictions, references=labels)

# 训练
trainer = Trainer(model, training_args,
                  training_dataset=tokenized_datasets['train'],
                  eval_dataset=token_datasets['validation'],
                  data_collator=data_collator,
                  tokenizer=tokenizer,
                  compute_metrics=compute_metrics,)
trainer.train()

# 预测
predictions = trainer.predict(tokenizerd_datasets['validation'])
# 包含3个元素: predictions (logits), label_ids (ground truth), metrics
preds = np.argmax(predictions.predictions, axis=-1)
metric = load_metric('glue', 'mrpc')
metric.compute(preds, predictions.label_ids)
```

- Trainer是非常顶层的封装，与pytorch lightning相似
  - 适合写好了在多机多卡上运行，对ddp封装较好
  - 不适合不断改进，这种适合用pytorch写for循环

## evaluate

- evaluate库：计算度量标注，需要自己实现预测logits

## 完整训练代码

```python
import evaluate
import argparse
import torch
import numpy as np

from torch.utils.data import DataLoader
from datasets import load_dataset
from transformers import AutoTokenizer, AdamW, get_scheduler
from transformers import DataCollatorWithPadding
from transformers import AutoModelForSequenceClassification


class Classifier:
    def __init__(self, opt) -> None:
        self.opt = opt
        self.device = torch.device(f'cuda:{opt.card_id}')

        MODEL = 'cardiffnlp/twitter-roberta-base-sentiment'
        self.tokenizer = AutoTokenizer.from_pretrained(MODEL)
        # 加载预训练参数，num_labels指定分类头尺寸
        self.model = AutoModelForSequenceClassification.from_pretrained(MODEL, num_labels=5,
                                                                        ignore_mismatched_sizes=True)
        self.model.to(self.device)
        raw_dataset = load_dataset('SetFit/sst5')  # 情感分析，5分类
        self.tokenized_datasets = raw_dataset.map(self.tokenize, batched=True)
        self.tokenized_datasets = self.tokenized_datasets.remove_columns(['text', 'label_text'])
        self.tokenized_datasets = self.tokenized_datasets.rename_column('label', 'labels')
        self.tokenized_datasets.set_format('torch')

        self.data_collator = DataCollatorWithPadding(self.tokenizer)
        self.train_dataloader = DataLoader(self.tokenized_datasets['train'],
                                           batch_size=opt.bs, shuffle=True,
                                           collate_fn=self.data_collator)  # 在DataLoader才做pad
        self.validation_dataloader = DataLoader(self.tokenized_datasets['validation'],
                                                batch_size=opt.bs, shuffle=False,
                                                collate_fn=self.data_collator)
        self.test_dataloader = DataLoader(self.tokenized_datasets['test'],
                                          batch_size=opt.bs, shuffle=False,
                                          collate_fn=self.data_collator)

        self.optimizer = AdamW(self.model.parameters())
        for param in self.model.roberta.parameters():  # 仅调最后的分类头
            param.requires_grad = False
        num_training_steps = self.opt.epoch * len(self.train_dataloader)
        # 学习率在num_warmup_steps步内从0线性增长到定义optim时的lr，再线性递减到0
        self.lr_scheduler = get_scheduler(name='linear', optimizer=self.optimizer,
                                          num_warmup_steps=300, num_training_steps=num_training_steps)

    def tokenize(self, data):
        return self.tokenizer(data['text'], truncation=True, max_length=512)

    def train(self):
        best_acc, best_model = 0, None
        for ep in range(self.opt.epoch):
            self.model.train()
            for batch in self.train_dataloader:
                batch = {k: v.to(self.device) for k, v in batch.items()}
                loss = self.model(**batch).loss  # 字典的key作为参数名，value作为参数值
                self.optimizer.zero_grad()
                loss.backward()
                self.optimizer.step()
                self.lr_scheduler.step()

            result = self.validation()
            if best_acc < result['accuracy']:
                best_acc = result['accuracy']
                best_model = copy.deepcopy(self.model.state_dict())
            print(f"epoch {ep} validation: [accuracy: {result['accuracy']}]")
        self.model.load_state_dict(best_model)
        result = self.test()
        print(f"test result: [accuracy: {result['accuracy']}]")

    def validation(self):
        metric = evaluate.load('accuracy')
        self.model.eval()
        for batch in self.validation_dataloader:
            batch = {k: v.to(self.device) for k, v in batch.items()}
            with torch.no_grad():
                outputs = self.model(**batch)
                logits = outputs.logits
                predictions = torch.argmax(logits, dim=-1)
                metric.add_batch(predictions=predictions, references=batch["labels"])
        result = metric.compute()
        return result

    def test(self):
        metric = evaluate.load('accuracy')
        self.model.eval()
        for batch in self.test_dataloader:
            batch = {k: v.to(self.device) for k, v in batch.items()}
            with torch.no_grad():
                outputs = self.model(**batch)
                logits = outputs.logits
                predictions = torch.argmax(logits, dim=-1)
                metric.add_batch(predictions=predictions, references=batch["labels"])
        result = metric.compute()
        return result


if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('--epoch', default=20)
    parser.add_argument('--bs', default=32)
    parser.add_argument('--lr', default=1e-5)
    parser.add_argument('--card_id', default=2)
    opt = parser.parse_args()

    C = Classifier(opt)
    C.train()
```

## Adapter

```python
# 使用预训练RoBERTa，在烂番茄数据上微调情感分析任务
import numpy as np
from datasets import load_dataset
from transformers import RobertaTokenizer
from transformers import RobertaConfig, RobertaModelWithHeads
from transformers import TrainingArguments, AdapterTrainer, EvalPrediction

dataset = load_dataset("rotten_tomatoes")

tokenizer = RobertaTokenizer.from_pretrained("roberta-base")

def encode_batch(batch):
  """Encodes a batch of input data using the model tokenizer."""
  return tokenizer(batch["text"], max_length=80, truncation=True, padding="max_length")

# Encode the input data
dataset = dataset.map(encode_batch, batched=True)
# The transformers model expects the target class column to be named "labels"
dataset.rename_column_("label", "labels")
# Transform to pytorch tensors and only output the required columns
dataset.set_format(type="torch", columns=["input_ids", "attention_mask", "labels"])

config = RobertaConfig.from_pretrained("roberta-base", num_labels=2)
model = RobertaModelWithHeads.from_pretrained("roberta-base", config=config)

# Add a new adapter
model.add_adapter("rotten_tomatoes")
# Add a matching classification head
model.add_classification_head("rotten_tomatoes", num_labels=2, id2label={ 0: "👎", 1: "👍"})
# Activate the adapter
model.train_adapter("rotten_tomatoes")

training_args = TrainingArguments(
    learning_rate=1e-4,
    num_train_epochs=6,
    per_device_train_batch_size=32,
    per_device_eval_batch_size=32,
    logging_steps=200,
    output_dir="./training_output",
    overwrite_output_dir=True,
    # The next line is important to ensure the dataset labels are properly passed to the model
    remove_unused_columns=False,
)

def compute_accuracy(p: EvalPrediction):
  preds = np.argmax(p.predictions, axis=1)
  return {"acc": (preds == p.label_ids).mean()}

trainer = AdapterTrainer(
    model=model,
    args=training_args,
    train_dataset=dataset["train"],
    eval_dataset=dataset["validation"],
    compute_metrics=compute_accuracy)

trainer.train()
trainer.evaluate()
```

## 科学上网

- 目前国内无法直连到huggingface

- 下载huggingface预训练模型
  
  - 挂梯子，从huggingface下载文件
  - 镜像站：`hf-mirror.com`
    - `export HF_ENDPOINT=https://hf-mirror.com`，再调用下载脚本
    - `os.environ['HF-ENDPOINT'] = 'https://hf-mirror.com'`
  - 从国内镜像站找（模型少）：`aliendao.cn`, `modelscope.cn`

- 加载本地预训练模型
  
  - 法1：将模型名更换为下载好的文件夹
  - 法2：手动构建`.cache`
    - 从官网下载的预训练模型，默认存在`~/.cache/huggingface/hub/`下面

- 预训练模型在`.cache`的存储形式
  
  - 文件夹名称：`models--{上传用户名}--{模型名}`
    - 内部包含`refs, blobs, snapshots`三个文件夹
  - `refs`：存放每次commit的hash
    - 例：仅有1个main文件（branch名称），内容是最新的commit的hash（没有`\n`）
      - 删除\n：`tr -d "\n" < refs/main_cp > refs/main`
  - `blobs`：存放实际下载的文件，如config.json和模型参数
    - 从`snapshots`中的文件hard link过来
    - 若为config文件，则文件名改为其git hash
      - `git hash-object $file`
    - 若为模型参数文件 (LFS)，则文件名改为其sha256 hash
      - `openssl sha256 $file`
    - 根据hash值，判断不同commit是否修改了每个文件，若未修改，则不用下载
  - `snapshots`：存放每次commit的文件的软链接（实际是直接拷贝）
    - 子文件夹名：相应commit的hash
    - 子文件夹内容：相应commit的文件，例如config.json, pytorch_model.bin
    - 与官网的一致，不需要修改名称
