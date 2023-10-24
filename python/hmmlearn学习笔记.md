# hmmlearn学习笔记

- 三种模型
  - hmm.GaussianHmm：观测概率为高斯
  - hmm.GMMHMM：观测概率为多元高斯
  - hmm.MultinomialHMM：观测概率为离散分布

## 方法

- 学习参数：`model.fit(X)`
  - X为观测样本：sample*series
- 前向估计：`model.predict(X)`
  - X为待估计概率的观测，输出对数似然
- 采样：`X, Z = model.sample(100)`：
  - 计算HMM100个样本的观测X和隐变量Z

## 初始化

```python
model = hmm.GaussianHMM(n_components=3, covariance_type="full")  # 3种隐状态
model.startprob_ = np.array([0.6, 0.3, 0.1])
model.transmat_ = np.array([[0.7, 0.2, 0.1],
...                             [0.3, 0.5, 0.2],
...                             [0.3, 0.3, 0.4]])
model.means_ = np.array([[0.0, 0.0], [3.0, -3.0], [5.0, 10.0]])
model.covars_ = np.tile(np.identity(2), (3, 1, 1))
```

