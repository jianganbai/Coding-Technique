# pytorch 序列变长处理学习笔记

- LSTM：跳过pad部分计算

- Transformer：pad部分依旧参与计算，通过加mask去除pad部分影响

## LSTM

```python
import torch
import torch.nn as nn

lstm = nn.LSTM(input_size=10, hidden_size=5, batch_first=True)
# 假设输入序列长度不同
padded_sequences = torch.randn(3, 8, 10)  # batch_size=3, max_len=8
lengths = [8, 5, 3]  # 实际长度
packed = nn.utils.rnn.pack_padded_sequence(
    padded_sequences, lengths, batch_first=True, enforce_sorted=False
)
# LSTM 可接受 PackedSequence
output, (hn, cn) = lstm(packed)  # hn: hidden state; cn: cell state
# 转化为加pad的格式
output_padded, output_lengths = nn.utils.rnn.pad_packed_sequence(
    output, batch_first=True
)
```

- 先手动pad至最长，拼接成batch tensor。pad value 必须为0

- 然后调用`pack_padded_sequence`，转化为`PackedSequence`对象

- `nn.LSTM`可自动识别`PackedSequence`对象，自动跳过padding部分计算
  
  - `hn`：最后一个有效的hidden，因为LSTM自动跳过了pad部分的计算

- 计算完成后，调用`pad_packed_sequence`，转化为带pad的LSTM输出

- 提取最后一个有效帧对应输出

  - ```python
    batch_indices = torch.arange(output_padded.size(0))
    last_indices = output_lengths - 1  # 从0开始索引
    last_hidden_states = output_padded[batch_indices, last_indices, :]
    ```

## Transformer

```python
import torch
import torch.nn as nn

class TransformerForSequence(nn.Module):
    def __init__(self, vocab_size, d_model=512, nhead=8, num_layers=6):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, d_model)
        self.pos_encoder = PositionalEncoding(d_model)
        encoder_layer = nn.TransformerEncoderLayer(
            d_model, nhead, batch_first=True
        )
        self.transformer = nn.TransformerEncoder(encoder_layer, num_layers)
        self.fc = nn.Linear(d_model, vocab_size)
    
    def forward(self, src, src_lengths):
        # src: [batch_size, seq_len]
        # 1. 嵌入
        x = self.embedding(src)
        
        # 2. 添加位置编码
        x = self.pos_encoder(x)
        
        # 3. 生成填充掩码
        # src_key_padding_mask: True 表示要屏蔽的位置
        max_len = src.size(1)
        mask = torch.zeros(src.size(0), max_len, device=src.device)
        for i, l in enumerate(src_lengths):
            mask[i, l:] = 1
        src_key_padding_mask = mask.bool()
        
        # 4. Transformer 编码
        output = self.transformer(
            x, 
            src_key_padding_mask=src_key_padding_mask
        )
        
        return self.fc(output)
```

- Attention中，先算$QK^T$，然后将包含pad的位置置为-inf，再送入softmax

  - ```python
    def scaled_dot_product_attention(Q, K, V, mask=None):
        scores = torch.matmul(Q, K.transpose(-2, -1)) / (K.size(-1) ** 0.5)
        
        if mask is not None:
            # 将 padding 位置的注意力分数设为极小值
            scores = scores.masked_fill(mask == 0, float('-inf'))
        
        attention_weights = torch.softmax(scores, dim=-1)
        output = torch.matmul(attention_weights, V)
        return output, attention_weights
    ```

