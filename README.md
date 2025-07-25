![image-20250306182014120](/resources/images/project_logo.png)

# KHAOSZ

<div style="text-align: center; font-size: 16px; font-weight: bold;">
  <a href="#english" style="text-decoration: none; margin: 0 10px;">English</a> | 
  <a href="#chinese" style="text-decoration: none; margin: 0 10px;">中文</a>
</div>
 

<h2 id="english">English Version</h2>

This is a Chinese-English bilingual Transformer model supporting both languages. It contains model configurations and training workflows, completing training by loading parameters defined in `params/config.json`. The training script `train.py` parses command-line arguments, including dataset root directory, number of training epochs, batch size, checkpoint interval, and checkpoint directory.

**Model Download Options (Choose One):**

1. Visit [HuggingFace](https://huggingface.co/ViperEk/KHAOSZ) to access **Files and versions**
2. Run `params/download.py` to download parameters

**Demo Video:** [bilibili](https://www.bilibili.com/video/BV1z5RPYHEkd)

Training dataset sources are listed in the **Model Card** section of the HuggingFace download link.

**License:** Code follows Apache-2.0 protocol. Please credit the source code when used.

- **📊 Device Selection:** Code defaults to CUDA training
- **🌐 Performance Optimization:** `dtype=torch.bfloat16` is enabled to accelerate training and reduce memory usage. Ensure hardware supports this feature.
- **🤖 Language Support:** Model supports Chinese and English training. The BBPE tokenizer was trained without multilingual text, so OOV (out-of-vocabulary) issues are minimized for these languages but may exist for others.

### 📌 Training Guide

To train this Transformer model, follow these steps:

**(1). Prepare Dataset:**

Place datasets in the designated root directory. Files should be text documents in Chinese, English, or mixed. Format should align with model input requirements - preferably pre-tokenized token_ids stored as `torch.Tensor` (using `torch.Tensor` saves memory compared to Python lists, which default to 64-bit precision).

**(2). Install Dependencies:**

```bash
conda env create -f environment.yml --name env_name
```

**(3). Run Training Script:**

```bash
python train.py \
--train_type=train_type[seq, sft, dpo] \
--data_root_path=/path/to/dataset \
--n_epoch=5 \
--batch_size=8 \
--max_lr=2e-4 \
--n_iter_ckpt=10000 \
--ckpt_dir checkpoints 
```

**Parameters Explanation:**
- `--train_type`: Training type (seq, sft, dpo)
- `--data_root_path`: Dataset root directory
- `--n_epoch`: Total training epochs
- `--batch_size`: Batch size
- `--n_iter_step`: Number of batches per training step
- `--warning_step`: Warmup steps
- `--max_lr`: Maximum learning rate (uses warmup + cosine decay)
- `--n_iter_ckpt`: Checkpoint saving interval
- `--ckpt_dir`: Checkpoint directory
- `--resume_dir`: Path to resume training from checkpoint

Training logs are saved in `train_log.txt`. Checkpoints will be stored in the specified directory for resuming training or evaluation.

### 👉 Usage Guide

**(1). Chatting with the Model:**

Open `chat.py` or use streaming/non-streaming interfaces:

**Streaming Output:**
```python
import torch
from khaosz import Khaosz

model_dir = "your_model_parameter_dir"
model = Khaosz(model_dir).to(device='cuda', dtype=torch.bfloat16)
history = []

while True:
    query = input(">> ")
    if query == "!exit":
        break
    
    response_size = 0
    for response, history in model.stream_generate(
        query=query, 
        history=history,
        temperature=0.85,
        top_p=0.95,
        top_k=50
    ):
        print(response[response_size:], end="")
        response_size = len(response)       
```

**Non-streaming Output:**
```python
import torch
from khaosz import Khaosz

model_dir = "your_model_parameter_dir"
model = Khaosz(model_dir).to(device='cuda', dtype=torch.bfloat16)
history = []

while True:
    query = input(">> ")
    if query == "!exit":
        break
    
    response = model.generate(
        query=query, 
        history=history,
        temperature=0.85,
        top_p=0.95,
        top_k=50
    )
    print(response)
```

**(2) Retrieval-Augmented Generation (RAG):**

```python
import torch
from khaosz import Khaosz

model_dir = "your_model_parameter_dir"
model = Khaosz(model_dir).to(device='cuda', dtype=torch.bfloat16)

retrieved_content = model.retrieve_generate(
    query=query,
    retrieve_top_k=5,
    temperature=0.6,
    top_k=30,
    top_p=0.95
)
print(retrieved_content)
```

### 📌 Model Specifications

This model is based on a 20-layer Transformer with parameters defined in `config.json`, totaling approximately 400 million (0.40B) parameters.

**Key Design Choices:**
- Weight tying between embedding and final linear layers (standard for small models to save parameters)
- Embedding layer optimization: Without weight tying, a 10,000-word vocabulary would consume ~102M parameters (0.1B)

**Limitations:**
- May struggle with complex language phenomena due to smaller parameter size
- Prone to overfitting on specialized datasets
- Limited multilingual capabilities

**Advantages:**
- Runs efficiently on lower-spec hardware
- Shorter training time compared to larger models

**Training Pipeline:** 
The model has completed pre-training + SFT (Supervised Fine-Tuning) + DPO (Direct Preference Optimization) workflows. All corresponding training code is included in the repository.


<h2 id="chinese">中文版本</h2>

这是一个支持中文和英文双语言的Transformer模型，包含模型设置和训练流程， 通过加载`params/config.json` 中的设定的参数完成训练， 使用`train.py`解析命令行参数，包括数据集根目录、训练轮数、批处理大小、保存检查点的间隔轮数以及检查点保存目录。

模型下载方法(二选一)：

1. 前往 [HuggingFace](https://huggingface.co/ViperEk/KHAOSZ) 查找**Files and versions**
2. 运行 **params/download.py** 下载参数

演示视频：[bilibili](https://www.bilibili.com/video/BV1z5RPYHEkd)

训练数据集来源在huggingface下载链接中的**Model Card**界面查找

代码遵循 Apache-2.0 协议， 使用时请注明代码来源

- **📊设备选择**：当前代码默认使用CUDA进行训练
- **🌐性能优化**：代码中设置了`dtype=torch.bfloat16`来启用训练，这有助于提高训练速度和降低显存消耗，但需确保硬件支持此特性。
- **🤖语言支持**：该模型目前支持在中文和英文数据集上训练， 在训练分词器时没有加入其他语言的文本，BBPE分词器不会存在OOV问题，但是对别的语言支持比较差

### 📌如何训练

要训练这个Transformer模型，您可以按照以下步骤进行操作：

**(1). 准备数据集：**

确保您的数据集位于一个指定的根目录下。数据集应包含用于训练的文本文件，这些文件可以是中文、英文或两者混合。
数据文件的格式应与模型的输入要求一致，最好是经过tokenizer处理过后的token_id, 为了节省内存占用采用torch.Tensor 存储id,(如果使用python的list, 在读取训练数据的时候内存占用大概是原来的两倍以上，因为python似乎是默认采用64位数精度存储的数据， 但是实际上int32足够)

**(2).安装依赖：**

确保您已经安装了所有必要的Python库：

```bash
conda env create -f environment.yml --name env_name
```

**(3).运行训练脚本：**

使用以下命令运行训练脚本，并根据需要调整参数：

```bash
python train.py \
--train_type=train_type[seq, sft, dpo] \
--data_root_path=/path/to/dataset \
--n_epoch=5 \
--batch_size=8 \
--max_lr=2e-4 \
--n_iter_ckpt=10000 \
--ckpt_dir checkpoints 
```
--train_type: 指定训练的类型，可选值有seq, sft, dpo

--data_root_path：指定数据集的根目录路径。

--n_epoch：指定训练的总轮数。

--batch_size：指定每个批次的样本数量。

--n_iter_step： 多少batch迭代一步

--warning_step: 预热步数

--max_lr: 指定过程中最大的学习率（学习率采用的是预热 + 余弦衰减）

--n_iter_ckpt：指定每多少迭代次数保存一次检查点。

--ckpt_dir：指定保存检查点的目录。

--resume_dir: 恢复训练的checkpoint路径

训练过程中，您可以在终端中查看训练日志(train_log.txt)，了解训练进度、损失值等信息。
检查点文件会保存在指定的检查点目录中，您可以使用这些检查点文件来恢复训练或进行评估。


### 👉如何使用

**(1).使用模型完成聊天：**

如果您想使用这个模型进行对话聊天, 请打开 chat.py 文件，并运行它。
或者， 您可以使用流式输出接口/对话生成接口完成对话

```python
import torch
from khaosz import Khaosz

model_dir = "your_model_parameter_dir"
model = Khaosz(model_dir).to(device='cuda', dtype=torch.bfloat16)
histroy = []

while True:
    query = input(">> ")
    if query == "!exit":
        break
    
    response_size = 0
    for response, histroy in model.stream_generate(
        query=query, 
        history=histroy,
        temperature=0.85,
        top_p=0.95,
        top_k=50
    ):
        print(response[response_size:], end="")
        response_size = len(response)       

```

或者您可以使用非流式输出的方式完成对话

```python
import torch
from khaosz import Khaosz

model_dir = "your_model_parameter_dir"
model = Khaosz(model_dir).to(device='cuda', dtype=torch.bfloat16)
histroy = []

while True:
    query = input(">> ")
    if query == "!exit":
        break
    
    response_size = 0
    response =  model.generate(
        query=query, 
        history=histroy,
        temperature=0.85,
        top_p=0.95,
        top_k=50
    )
    print(response)
```

**(2) 使用模型完成向量检索生成(RAG)：**


如果您想在文本分段之后进行检索

```python
import torch
from khaosz import Khaosz

model_dir = "your_model_parameter_dir"
model = Khaosz(model_dir).to(device='cuda', dtype=torch.bfloat16)

# after init database

retrive_content = model.retrieve_generate(
    query=query,
    retrive_top_k=5,
    temperature=0.6,
    top_k=30,
    top_p=0.95
)
print(retrive_content)
```


### 📌其他问题
本模型基于20层的transformer，参数大致设置如`config.json`，参数大小为4亿（0.40b）

模型采用权重绑定， embedding层的权重和最后线性层的权重是共享的（比较小的模型都采用这种方式节省参数大小， 因为不采用权重绑定， embedding层假设有10000单词， 将会占用 10000 * 1024 = 102,400,000 参数， 也就是 0.1b 参数， 因为词表会占用太多的参数， 所以采用权重绑定是小模型的通用方法）

由于模型参数相对较少，在某些任务上可能会出现性能不足的情况，比如对复杂语言现象的理解能力可能不如更大规模的模型。此外，较小的模型也可能更容易过拟合训练数据，导致泛化能力较差。不过，这也意味着该模型可以在较低配置的硬件上运行，并且训练时间相对较短。

目前模型已经完成 pre-train + SFT + DPO 的流程， 相应的训练代码也存储在了项目当中