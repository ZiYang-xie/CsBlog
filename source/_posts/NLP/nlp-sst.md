---
title: NLP 情感分类任务
date: 2021-11-24 01:06:34
index_img: /img/NLP/banner.jpeg
category: [NLP]
tags: [SST, Sentiment Classification]
math: true
---

# NLP PJ-1 基于深度学习的文本分类

## 1. 背景介绍

### 情感分类任务 (Sentiment Classification)

​	文本情感分类任务是NLP众多下游任务的一种，通过深度学习模型来提取文本情感特征来达到对文本情感类型进行分类的目的。其可以是复杂的判别情感极性的多标签分类任务或是回归任务。我们这里是简单的二分类任务，即给定影评文本，确定其评论是积极 (positive) 还是消极 (negative)

### 词嵌入 (Word Embedding)

​	我们知道传统的词向量one-hot表示法虽然能够将词进行向量化，但是没办法很好的表示两个含义相近的词在空间当中的距离。同时在语料库很大的情况下，one-hot向量会变得非常稀疏且维度极高，非常不利于计算。为此我们引入了词嵌入的方法来对词向量进行降维，将词映射到一个相对于词数量较小的高维空间中。一个好的词嵌入应当能够反映不同的词语之间的联系，例如“猫”，“狗”两个词对应的词向量就应当比“猫”，“水杯”两个词对应的词向量距离更近。现在主流的词嵌入方法主要有两种，一种是托马斯·米科洛维的Word2Vec，以及斯坦福大学提出的GloVe

- **Word2Vec**

  Word2Vec是在整个语料库上进行预训练，通过相邻的词的词向量预测中心词的词向量  (*CBOW*)，或通过中心词的词向量预测相邻词的词向量 (*Skip-gram*) 两种方式，对于语料库中的词来做词嵌入。

​	相比CBOW在skip-gram当中，每个词都要收到周围的词的影响，每个词在作为中心词的时候，都要进行K次的预测、调整。因此当数据量较少，或者词为生僻词出现次数较少时， 这种多次的调整会使得词向量相对的更加准确，但是训练时间也要有所增长。

<img src="https://img1.daumcdn.net/thumb/R800x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FphEkS%2FbtqXSoISyn9%2FeI2vpCZ8svhF7X4U3JCTx0%2Fimg.png" style="zoom:70%;" />



- **GloVe**

  我们可以看到Word2Vec仅仅注意到了文本的局部上下文特征，而忽略了全局的文本信息，GloVe在此之上对其做出了改进。GloVe从基于SVD的LSA算法出发，首先要通过滑动窗口计算出共现矩阵。我们知道单词的词向量学习应该跟词共现概率的比率有关，而不是他们的概率本身作者发现用共现概率比也可以很好的体现3个单词间的关联(因为共现概率比符合常理)，所以作者大胆猜想，如果能将3个单词的词向量经过某种计算可以表达共现概率比，那么这样的词向量就与共现矩阵有着一致性，可以体现词间的关系。

  - **主要优化目标:** 	$$J = \sum_{ij}^{V}f(X_{ij})(v_i^T\hat{v_j}+b_i + \hat{b_j}-log(X_{ij}))^2$$

### 循环神经网络 (Recurrent Neural Network)

- **普通RNN**

​	循环神经网络即RNN，其结构设计基于输入序列和上一级状态对当前隐藏状态进行更新，即可以表示为 $p(y_t)=g(y_{t-1},h_t)$，能够比较好的处理序列内容，因此被广泛的应用于NLP领域。但是我们知道，对于普通的RNN来说一旦序列变长，会导致比较明显的网络退化问题，即梯度消失和梯度爆炸难以避免。

- **LSTM** （Long short-term memory）

  ​	为了解决上述问题，就有了LSTM的出现，即添加了一个全局的信息传递通道 $c_t$ ，通过 $sigmoid$ 作为门控来对上一个节点传入的输入进行选择性忘记，对全局状态进行少量改变。进而再进入输入阶段，对输入通过 $sigmoid$ 来做门控选择记忆，以及最后的输出隐层。内部使用 $tanh$ 作为激活函。通过添加全局记忆流的方式很好的避免了梯度消失问题，使得模型拥有更加长期的记忆能力。

  ​	但缺点就是模型比RNN复杂了不少，收敛起来速度更慢。

- **GRU** （Gate Recurrent Unit）

  为了解决LSTM训练收敛速度慢的问题，学界又引入了GRU，与LSTM相比，GRU内部少了一个门控即更新门和重置门，参数比LSTM少，在很大程度上提高训练效率。重置门越小代表前一状态被丢弃信息越多，更新门越大代表前一时刻带入信息越多。同时GRU在并不控制和保留内部记忆$c_t$ ，但从设计上能够达到LSTM同样的避免梯度消失的效果。

<img src="https://easy-ai.oss-cn-shanghai.aliyuncs.com/2019-07-03-lstm-gru.png" style="zoom:80%;" />

### Transformer

​	近年来的研究热点一直是在Transformer上，自从谷歌2017年在Attention is all you need一文中提出Transformer模型，其通过内部编码器解码器结构结合多头注意力机制 $QKV$，能够并行的注意到不同单元的特征信息，从而打破了RNN的网络退化训练难题，以BERT为代表的BERT家族模型目前一直是NLP研究的前沿热点模型，因为字数限制在此就不过多介绍。

<div STYLE="page-break-after: always;"></div>

## 2. 实验内容

### 2.1 基于RNN及其变体的分类器

我首先在RNN系列模型上进行了一系列实验，采用统一参数，并且拿出最后一层的输出，做 *drop_out* 后通过一个全连接层得到最后的输出，同时全开bidirection。

​	同时在对比RNN分类器的实验过程中发现，在没有添加 *pretrain* 的 word_embedding，即使用随机初始化的 embedding 进行训练能够更加明显的对比出不同模型的效果。

- **模型主体**

```python
def forward(self, x):
        batch_size, seq_len = x.size()
        x = self.embed_dropout(self.embeddings(x))
        out, _ = self.rnn(x)
        out = out.view(batch_size, seq_len, self.num_labels, self.hidden_size)
        out = torch.cat([out[:, -1, 0, :], out[:, 0, 1, :]], dim=-1)
        out = self.dropout(out)
        logits = self.fc(out)
        return logits
```

- **模型参数及超参数**

```yaml
embed_dim: 300 	# 嵌入维度
hidden_size: 512  # 隐藏层维度
num_labels: 2  # 标签数
num_layers: 2  # 层数
dropout: 0.5  # 全连接层 drop out
embed_dropout: 0.5  # 嵌入层 drop out
WORD_EMBEDDING: Random # 初始对比不加 Word Embedding 效果更加明显
```

```yaml
# 超参数
EPOCH: 30
BATCH_SIZE: 64
OPTIMIZER: # 优化器
  NAME: AdamW
  ARGS:
    lr: 6.0e-5
    eps: 1.0e-6
SCHEDULER: # 无LR调节
  NAME: null
```

#### 2.1.0 Preprocess

在前处理过程中，滤除了语料库当中的非字母符号，对字母进行全小写化。（尝试滤除stop word但其因为会改变很多语义信息，从而最后并未采纳）

```python
def preprocess_data(lines):
    formatted_lines = []
    for line in lines:
        # split into tokens by white space
        tokens = line.split()
        # remove punctuation from each token and convert to lowercase
        table = str.maketrans('', '', string.punctuation)
        tokens = [w.translate(table).lower() for w in tokens]
        # remove remaining tokens that are not alphabetic
        tokens = [word for word in tokens if word.isalpha()]
        # join the new tokens to form the formatted sentence
        sentence = " ".join(tokens)
        formatted_lines.append(sentence)
    return formatted_lines
```

#### 2.1.1 RNN

RNN 在随机初始化的 Word Embedding 上表现较差，在Val上最高的准确率达到 65.30%，后期过拟合现象较为严重。

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwdqmwrpcbj30t60cp3za.jpg" style="zoom:60%;" />

#### 2.1.2 LSTM

LSTM 则表现较好，最高在Val上的准确率达到 72.84%，同时由于网络表达能力增强，其过拟合现象比RNN更加严重。

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwdqpqi5uhj30ya0dnt9n.jpg" style="zoom:50%;" />

#### 2.1.3 GRU

GRU 同样表现不错，在Val上的准确率最高达到 71.57%, 稍微弱于LSTM，但是其收敛速度快于LSTM

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwdqsaqauoj30st0dl0tk.jpg" style="zoom:60%;" />

#### 2.1.4 小结

​	实验中我们可以看到，单论模型性能 *LSTM* 是表现最好的网络，*GRU* 作为 *LSTM* 的改进版，能够更快的拟合并获得接近 *LSTM* 的效果，而 *RNN* 则由于梯度消失导致的网络退化问题，难以很好的拟合，梯度爆炸导致训练不稳定，性能较差。

| 模型 | 准确率 / % | Epoch / n |
| :--: | :--------: | :-------: |
| RNN  |   65.30    |    16     |
| LSTM |   72.84    |    13     |
| GRU  |   71.57    |    12     |



在实验过程中我们不难得出以下结论

**模型性能上：** $LSTM > GRU > RNN$

<div STYLE="page-break-after: always;"></div>

**收敛速度上：** $GRU>LSTM>RNN$

- **过拟合**

  同时在实验过程中我们很容易观察到，在训练后期模型都不同程度的出现了过拟合现象，这里我们在embedding和fc层都添加了0.5的drop_out，如果不添加的话过拟合现象则会更加研究 (详见后文的ablation study)，我们可以使用 early stop 方法简单的早停止训练来防止过拟合现象发生。我们可以看到三个模型在 15 个 Epoch 左右都达到了其最佳性能，所以我们可以将最大 Epoch 设为 15 来防止后期的过拟合现象发生。



#### 2.1.5 进一步分析

我们还可以继续计算模型的查准率 (Precision), 召回率 (Recall) , F1 score
$$
Precison = \frac{TP}{TP+FP}\\\\
Recall = \frac{TP}{TP+FN}\\\\
F_1 = \frac{2*Precision*Recall}{Precision + Recall}
$$


<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwehwkiqjlj30sp0azwf1.jpg" style="zoom:50%;" />

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwehsbzj9oj30fp07l74j.jpg" style="zoom:80%;" />

- **关于查准率和召回率**

  从图中我们不难看出，模型的查准率和召回率互为拮抗，查准率上升的时候召回率一般会下降，为了平衡衡量这两者来公平衡量二分类模型的准确率，我们可以使用F1 Score

- **F1 Score**

  ​	F1 score作为兼顾了分类模型的准确率和召回率。F1分数可以看作是模型查准率和召回率的一种加权平均，形式表现为查准率和召回率的调和平均。

<div STYLE="page-break-after: always;"></div>

### 2.3 比较不同的 Word Embedding 方式

​	在上述实验的基础上，我选用 LSTM 来进行进一步的实验，为LSTM添加不同的 pretrain 的 Word_Embedding 参数来查看 Word_Embedding 对网络准确率的影响。

#### 2.3.1 Word2Vec

​	首先实验的是Word2Vec, 这里我选用的是 ```GoogleNews-vectors-negative300```

```yaml
WORD_EMBEDDING: 
  NAME: Word2Vec # Word2Vec, GloVe, None
  PATH: Gensim/GoogleNews-vectors-negative300.bin
```

```python
def load_word2vec(text_field, PATH):
    word_to_idx = text_field.vocab.stoi
    pretrained_embeddings = np.random.uniform(-0.25, 0.25, (len(text_field.vocab), 300))
    pretrained_embeddings[0] = 0
    word2vec = load_bin_vec(PATH, word_to_idx)
    for word, vector in word2vec.items():
        pretrained_embeddings[word_to_idx[word]-1] = vector

    return pretrained_embeddings
```

​	使用预训练的参数替换模型中的 embeddings 层的初始值，其余训练参数和超参均同上，进行公平对比。

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwdr76zpusj30sq0dkaau.jpg" style="zoom:60%;" />

可以看到，预训练的词嵌入非常好的指导网络在情感分类任务上的训练拟合，拟合速度和准确率相比随机初始化的参数都有较大幅度的提升，仅在第7个epoch就得到了最好性能。

<div STYLE="page-break-after: always;"></div>

|     模型      | 准确率 / % | Epoch / n |
| :-----------: | :--------: | :-------: |
|  LSTM-Random  |   72.84    |    13     |
| LSTM-Word2Vec |   75.11    |     7     |



#### 2.3.2 GloVe

​	我们换上比Word2Vec更加具有全局信息的GloVe嵌入，这里采用的是 ```glove.840B.300d```

```yaml
WORD_EMBEDDING: 
  NAME: GloVe # Word2Vec, GloVe, None
  PATH: 'glove.840B.300d'
```

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwdrc8z2jqj30sn0dnaat.jpg" style="zoom:60%;" />

GloVe 的词嵌入比 Word2Vec 达到了更好的性能，在Val上达到了79.2%的准确率，可以看到预训练的词嵌入对模型性能的提升有非常大的效果

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwdtwdj6awj30xh0e10u5.jpg" alt="image-20211113211031902" style="zoom:55%;" />

<div STYLE="page-break-after: always;"></div>

|     模型      | 准确率 / % | Epoch / n |
| :-----------: | :--------: | :-------: |
|  LSTM-Random  |   72.84    |    13     |
| LSTM-Word2Vec |   75.11    |     7     |
|  LSTM-GloVe   |    79.2    |     8     |

#### 2.3.4 总结

​	可以看到预训练的词嵌入能够非常大的提升模型的性能，让模型更好的抓住句子和词之间的联系，抽取句子的语义信息。对情感分类问题有着较大的帮助。





### 2.4 基于CNN的分类器

​	对于词嵌入向量，我们也可以使用CNN对其进行操作，通过卷积核来提取局部特征，并融合全局视野，不过相比RNN的模型结构。CNN的方式对语言问题少了一些先验的序列化模式，因此收敛速度要明显慢于RNN。

为和RNN对比，使用了统一参数。卷积核采用 $3 \times embed\_dim$ 大小，同时随机初始化 Word_embedding

```python
class CNN(nn.Module):
  ....
   self.conv_layers = nn.ModuleList([nn.Conv2d(in_channels=1,
                                     out_channels=hidden_size,
                                     kernel_size=(kernel_size, embed_dim))]*num_layers)
  ....
```

#### 2.4.1 实验结果



<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwdrjpqd2yj30so0dvdgn.jpg" style="zoom:55%;" />

<div STYLE="page-break-after: always;"></div>

​	可以看到CNN的拟合速度明显慢于基于RNN的分类器，担其效果不输LSTM甚至略微优于LSTM，在最后一个 Epoch 甚至达到了 73.48% 的准确率，如果继续训练可能会达到更高的准确率，但为了公平对比，因此不再扩大 Epoch 数。

| 模型 | 准确率 / % | Epoch / n |
| :--: | :--------: | :-------: |
| RNN  |   65.30    |    16     |
| LSTM |   72.84    |    13     |
| GRU  |   71.57    |    12     |
| CNN  |   73.48    |    30     |

<div STYLE="page-break-after: always;"></div>

### 2.5 基于BERT及其变体的分类器（Transformer） 

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwds81vpfuj30fe0ndwg2.jpg" style="zoom:60%;" />

#### 2.5.1 BERT

​	BERT全名即 *Bidirectional Encoder Representations from Transformers* 即从模型思路上使用了 Transformer 的 Encoder 部分，来对输入词向量做 Self Attention，但其比普通 Transformer 的 Encoder 更深更窄，BERT-base 有 12 个 encoder 模块，hidden-size 是 768。

- **预训练任务**

  BERT出彩的地方是其预训练部分做的很好，除了之前所说的普通Word Embedding方式使用的 **Masked LM** 即遮挡预测以外，BERT还引入了下一句预测的预训练模式 **Next Sentence Prediction**，让模型能够更好的理解句子之间的关系

- **实验方法**

  采用载入预训练的BERT checkpoint 再在下游 SST-2 数据集上进行 fine-tune 的方式进行训练

<div STYLE="page-break-after: always;"></div>

- **实验参数**

```yaml
MODEL:
  NAME: BERT # RNN, LSTM, GRU, CNN, BERT
  ARGS:
    model_type: bert-base-uncased # bert-base-uncased, bert-large-uncased
    dropout: 0.5
OPTIMIZER: 
  NAME: AdamW
  ARGS:
    lr: 2.0e-5
    eps: 1.0e-8
```

```python
# 采用 linear_schedule_with_warmup
scheduler = get_linear_schedule_with_warmup(
   optimizer=optimizer,
   num_warmup_steps=0,
   num_training_steps=num_training_steps
)
```

- **实验结果**

我个人分别在 BERT-base-uncased 和 BERT-large-uncased 上进行了训练

- **BERT-base-uncased**

**参数信息：** 12-layer, 768-hidden, 12-heads, 110M parameters

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwdsttr9sbj30s90cr750.jpg" style="zoom:60%;" />

<div STYLE="page-break-after: always;"></div>

- **BERT-large-uncased**

**参数信息：** 24-layer, 1024-hidden, 16-heads, 336M parameters

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwdsnvaegsj30sn0cqt9f.jpg" style="zoom:60%;" />

- **总结**

|        模型        | 准确率 / % | Epoch / n |
| :----------------: | :--------: | :-------: |
| BERT-base-uncased  |   86.97    |     1     |
| BERT-large-uncased |   88.65    |     2     |

可以看到 BERT-large 性能好于 BERT-base 能够达到极高的 88.65% 同时由于参数量的提升，收敛上页略慢于BERT-base，在第2个epoch达到最佳性能。



#### 2.5.4 其他BERT变种

​	我继续使用测试了其他 BERT 变种例如 RoBERTa，以及AlBERT来进一步提升模型的准确率，经过实验发现 RoBERTa-large的准确率可以达到90%左右，而最好的准确率是使用 AlBERT-xxlarge 上可以在 Val 上达到 91%的准确率。

- **训练参数设置：**

```yaml
Scheduler: warmup_cosine
Optimizer: AdamW
LR: 1e-5
Epoch: 5
BatchSize: 8
```

<div STYLE="page-break-after: always;"></div>

- **AlBERT-xxlarge-v2**

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwdszgffm2j30xg0dijsa.jpg" style="zoom:60%;" />

​	准确率最高达到 91.01%，可以看到模型在大约第3个epoch达到最佳性能，后开始过拟合。为此我使用early stop，训练3个epoch

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwefzu7y82j30wt0dfgmh.jpg" style="zoom:60%;" />

​	模型准确率最高达到91.46%

<div STYLE="page-break-after: always;"></div>

### 2.6 训练方法对模型性能的影响

#### 2.3.2 改变BatchSize

​	Batch Size 对模型的准确性没有很大的影响，但在小batch的情况下有可能导致训练不稳定，iter之间的loss差距较大。

​	同时更大的Batch Size能够加快训练速度，更好的在全数据集上拟合，但这也不是说越大越好。过大的batch size可能会导致batch之间梯度变化过小，模型泛化能力缺失。同时也要注意自己使用的是什么Optimizer，如果是SGD优化器，batch size和learning rate之间成线性关系，在更改batch size的时候需要注意更改学习率。

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwdu5o0h46j30sp0cijsn.jpg" style="zoom:60%;" />

#### 2.3.4 改变Drop Out

​	drop out 对网络过拟合现象有比较大的影响，如果drop out过小网络表达能力过强且集中于几个特定神经元上，导致训练很快拟合到训练集上，从而导致模型泛化能力变差，性能减弱。这里我测试了三种drop_out的大小，分别是0.5, 0.2, 0

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwehhhqij9j30nj0cw3zm.jpg" style="zoom:60%;" />

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwehla48ruj30nh0cpt9h.jpg" style="zoom:60%;" />

可以看到网络在 drop out = 0.5 的时候性能最好。在 drop out = 0 的时候过拟合非常严重，性能最差。但因为drop out比较大，拟合上相比0.2和0速度更慢。

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gwehdm8dyvj30g909f3zc.jpg" style="zoom:67%;" />

​	在限制 drop out 为0.5时，其eval上的loss最小，而train上的loss最大，eval和train上的准确率增长较为同步。且过拟合现象最不严重，相比之下，drop out=0 时，过拟合现象十分严重，测试集上loss很大。

- **关于Drop out**

不得不提，如果drop out过大网络的表达能力会被损害，如果drop_out = 1相当于网络中这一层上所有激活全被 drop 掉，网络就无法输出信息了。因此选用合适的drop out十分重要



<div STYLE="page-break-after: always;"></div>

## 3.总结

​	在本次实验中，对比了不同的模型在情感分类任务上的性能，探索了不同Word Embedding对模型性能的影响，比较了不同训练方法对模型的影响。基本了解并探索了NLP的基本任务和实验流程。得到如下实验结果

|        模型        | 准确率 / % | Epoch / n |
| :----------------: | :--------: | :-------: |
|        RNN         |   65.30    |    16     |
|        LSTM        |   72.84    |    13     |
|        GRU         |   71.57    |    12     |
|        CNN         |   73.48    |    30     |
|   LSTM-Word2Vec    |   75.11    |     7     |
|     LSTM-GloVe     |    79.2    |     8     |
| BERT-base-uncased  |   86.97    |     1     |
| BERT-large-uncased |   88.65    |     2     |
|   RoBERTa-large    |   90.02    |     3     |
| AlBERT-xxlarge-v2  |   91.46    |     3     |

#### 3.1 进一步实验方向

- **蒸馏学习**

  可以通过高准确率的模型例如BERT给低准确率的模型例如LSTM进行蒸馏，在训练过程中增加一个loss进行指导训练，来进一步提升小模型的准确率。这在实际工业场景中有很大的应用场景
  $$
  loss = \alpha CrossEntropy(pred, true) + (1-\alpha) MSEloss(pred, bert\_true)
  $$

- **多模型ensemble**

  可以采用多模型 ensemble 的方式进一步提升模型的准确率，即训练多个不同的模型，或即使是相同模型进行不同初始化的结果，最后做简单的投票。这种ensemble方式能够进一步提升模型的性能，目前广泛应用于对及时性要求较少的场景和各大算法竞赛当中（例如Kaggle）

<div STYLE="page-break-after: always;"></div>

## Reference

[1] Ralf C. Staudemeyer, & Eric Rothstein Morris. (2019). Understanding LSTM – a tutorial into Long Short-Term Memory Recurrent Neural Networks.

[2] Sherstinsky, A. (2020). Fundamentals of Recurrent Neural Network (RNN) and Long Short-Term Memory (LSTM) network*. Physica D: Nonlinear Phenomena, \*404\*, 132306.

[3] Jacob Devlin, Ming-Wei Chang, Kenton Lee, & Kristina Toutanova. (2019). BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding.

[4] Victor Sanh, Lysandre Debut, Julien Chaumond, & Thomas Wolf. (2020). DistilBERT, a distilled version of BERT: smaller, faster, cheaper and lighter.

[5] Yinhan Liu, Myle Ott, Naman Goyal, Jingfei Du, Mandar Joshi, Danqi Chen, Omer Levy, Mike Lewis, Luke Zettlemoyer, & Veselin Stoyanov. (2019). RoBERTa: A Robustly Optimized BERT Pretraining Approach.

[6] Dang, N., Moreno-García, M., & Prieta, F. (2020). Sentiment Analysis Based on Deep Learning: A Comparative Study*. Electronics, \*9*(3), 483.





