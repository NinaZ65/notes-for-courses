# PPT10 深度学习（四）：Transformer 及大模型

1. **Transformer 本体**

- Transformer = 用 self-attention 处理序列的神经网络结构，主要用于序列建模
- self-attention：模型需要根据上下文动态得到 word embedding。本质上是矩阵计算。
  - Embedding 就是把词、位置、图像块这类对象变成神经网络能计算的向量；
  - Transformer 先用 embedding 表示输入，再用 self-attention 根据上下文更新这些表示。
- Q、K、V：\
  $q$ 是 query，表示当前 token 想找什么信息。\
  $k$ 是 key，表示每个 token 提供什么匹配特征。\
  $v$ 是 value，表示真正被汇总的信息内容。
  - 计算时，每个 token 用自己的 $q$ 去和所有 token 的 $k$ 做匹配，得到相关性分数；
  - 再经过 softmax 变成权重；
  - 最后用这些权重对所有 $v$ 加权求和，得到融合上下文后的新表示。
  - 核心公式：$Attention(Q,K,V)=softmax(\frac{QK^T}{\sqrt d})V$\
    $QK^T$ 算相关性，\
    $\sqrt d$ 防止数值过大，否则 softmax 后可能变得过于极端，训练不稳定\
    得到注意力权重，最后乘 $V$ 汇总信息。
    - 某token：它要更新自己的表示，所以它拿自己的 query：$q_{bank}$，去和所有 token 的 key 做匹配\
    然后加权得到一个新embedding
- **self-attention 的优势**: 具有良好的并行性、扩展性和通用表示能力
- **multi-head attention**
  - 多头注意力为注意力层提供“多个表示子空间”，让模型从多个角度捕捉复杂关系。
  - 比如有的 head 可能关注主谓关系，有的关注指代关系，有的关注局部短语关系。

- **position encoding**
  - self-attention 会丢失序列顺序信息，因此要在输入 $X$ 的词向量中加入位置信息。

- **Transformer 结构**。
  - Encoder 负责理解输入序列，Decoder 负责生成输出序列。
  - 以机器翻译为例，Encoder 编码中文句子，Decoder 根据已经生成的词和 Encoder 输出继续生成英
  - Decoder 里有 masked self-attention，作用是生成当前词时不能偷看未来词。比如生成第 3 个词时，只能看前面已经生成的内容。

2. **大模型与视觉 Transformer**

- 为什么 Transformer 适合做大模型。
  - 性能可以随着参数规模增长而提升；
  - 计算模式**并行性强**，适合大规模 GPU 集群；
  - 自注意力能建立全局连接，不只适用于文本，也能处理图像、音频、视频、蛋白质序列等；
  - Encoder、Decoder 或二者组合也能适配理解、生成、翻译等任务。

  - **BERT**。
    - 偏“理解”的模型，用 Transformer Encoder 做大规模**预训练**。
    - 两个预训练任务： MLM 和 NSP。
      - Masked Language Modeling，类似完形填空：把句子中一部分词遮住，让模型根据上下文预测被遮住的词。
      - Next Sentence Prediction，让模型判断两个句子是否是上下句关系。
      - “预训练 + 微调”的范式

  - **GPT**。
    - 偏“生成”的模型，用 Transformer Decoder 做**自回归语言建模**。
    - 它的核心训练目标是**预测下一个词**。
    - GPT-1 仍然强调预训练和微调；GPT-2 开始把更多任务统一成文本生成问题；GPT-3 依靠更大规模模型和数据，展现出 zero-shot、one-shot、few-shot 和 in-context learning 能力。
  - BERT 更适合理解任务，比如分类、阅读理解、信息抽取。\
    GPT 更适合生成任务，比如续写、问答、对话、代码生成。\
    BERT 看双向上下文，GPT 按从左到右预测下一个 token。

  - **ChatGPT 的关键技术**。
    - LoRA :**轻量级微调**方法，它冻结原始大模型参数 $W_0$，只训练低秩矩阵 $A$ 和 $B$，大幅降低微调成本。
    - RLHF :类人反馈强化学习：先收集**人类偏好**，训练奖励模型，再用强化学习让模型输出更符合人类偏好。
    - CoT(思维链) ，Chain of Thought：引导模型生成**中间推理步骤**，提升复杂推理能力。

  - **DeepSeek / MOSS 大模型案例**。
    - 代表中文大模型和开源大模型的发展。
    - DeepSeek 的 MLA、多头潜在注意力、MoE 架构、低成本训练推理和开源生态。

3. **从 NLP 到 CV：视觉 Transformer**。

- **ViT，Vision Transformer**。
  - ViT 把图像切成一个个 patch，比如 $224 \times 224 \times 3$ 的图像，patch 大小为 $16 \times 16$，就得到 196 个 patch。每个 patch 拉平成向量，再加上位置编码，像处理文本 token 一样送入 Transformer。
  - 把视觉问题转化为 sequence 问题。

- **MAE，Masked Autoencoder**。
  - 借鉴 BERT 的 mask 思路，把图像中大量 patch 遮住，只让 encoder 看可见 patch，再让 decoder 重构被遮住的图像块。
  - 使用较高 mask 比例，比如 75%，并采用非对称 encoder-decoder 设计来缩短预训练时间。

- **SAM，Segment Anything Model**。
  - SAM 是可提示的分割系统。
  - 用户可以给点、框、文本或 mask 作为 **prompt**，模型输出对应分割结果。
  - SAM 基于 MAE 训练好的 ViT，具有 zero-shot transfer 能力，可以对不熟悉的对象和图像进行零样本泛化。
