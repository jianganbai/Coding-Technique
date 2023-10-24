# gensim学习笔记

- NLP最底层的库，提供了Word2Vec, LDA等底层模型

## 基础概念

- corpus => document => vector
  - corpus：document的集合
  - document：几句话
  - vector：document的向量形式表征
- 基于Numpy实现

## Word2Vec

### 自己训练

```python
from gensim.models import Word2Vec

model = Word2Vec(sentences=语料库，[['i', 'love', 'you']]，每个元素为一个句子split之后的结果,
                 vector_size=embedding维度,
                 window=滑动窗长
                 min_count=忽略小于min_count的词,
                 workers=多少个并行线程，需要先安装Cython)
model.wv.similarity('man', 'woman')  # 给出两个词的相似性，两个词必须都在语料库中
model.wv['man']  # 'man'映射为的embedding
model.save(path)  # 保存网络参数

model = Word2Vec.load(path)  # 加载模型
```

### pretrain

```python
import gensim.download as api
from gensim.test.utils import datapath

info = api.info()  # 所有可下载的数据集和预训练模型
model = api.load("word2vec-google-news-300")
sim = model.evaluate_word_pairs(datapath("wordsim353.tsv"))
```

