# VLM

[TOC]

**Input:**

- **`input_ids`**: the tokenized text sequence (IDs from the vocabulary)

- **`labels`**: Target sequence for loss computation. Usually a clone of `input_ids`, but with padding (or ignored positions) replaced by `-100`. This ensures those positions are excluded from loss calculation.

- **`attention_mask`**:  Indicates which tokens are real (`1`) and which are padding (`0`). Prevents the model from attending to padding tokens.

- **`pixel_values`**: Image tensors from the vision encoder (after resizing, normalization, etc.).

- **`token_type_ids`** (in BERT-like models): Segment embeddings marking sentence A vs. sentence B.

  **`position_ids`**: Optional position indices; usually auto-generated.

**Output:**

- **`loss`**: Computed if `labels` are provided (cross-entropy loss over vocabulary).
- **`logits`**: Tensor of shape `(batch_size, seq_len, vocab_size)`, representing probability distributions for the next token.
- **`hidden_states`** (optional): Embeddings from each transformer layer (useful for fine-tuning or analysis).
- **`attentions`** (optional): Attention weights from each layer (useful for visualization and debugging).

### 训练细节

对视觉语言模型进行预训练的方法很多。主要技巧是统一图像和文本表征以将其输入给文本解码器用于文本生成。最常见且表现最好的模型通常由图像编码器、用于对齐图像和文本表征的嵌入投影子模型 (通常是一个稠密神经网络) 以及文本解码器按序堆叠而成。至于训练部分，不同的模型采用的方法也各不相同。

LLaVA 由 CLIP 图像编码器、多模态投影子模型和 Vicuna 文本解码器组合而成。

- pre-trainining：仅通过给模型馈送图像与问题并将模型输出与描述文本进行比较来训练多模态投影子模型，从而达到对齐图像和文本特征的目的。
- fine-tuning：解冻文本解码器，然后继续对解码器和投影子模型进行训练

![VLM Structure](https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/blog/vlm/vlm-structure.png)

KOSMOS-2 选择了端到端对模型进行完全训练，pretraining比llava昂贵不少。预训练后，用纯语言指令对模型进行finetune以对齐。

Fuyu-8B 甚至都没有图像编码器，直接把图像块馈送到投影子模型，然后将其输出与文本序列直接串接送给自回归解码器。

![VLM Structure](https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/blog/vlm/proj.jpg)

### 在 transformers 中使用视觉语言模型

使用`LlavaNext`模型对Llava进行推理

首先，我们初始化模型和数据处理器。

```python
from transformers import LlavaNextProcessor, LlavaNextForConditionalGeneration
import torch

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
processor = LlavaNextProcessor.from_pretrained("llava-hf/llava-v1.6-mistral-7b-hf")
model = LlavaNextForConditionalGeneration.from_pretrained(
    "llava-hf/llava-v1.6-mistral-7b-hf",
    torch_dtype=torch.float16,
    low_cpu_mem_usage=True
)
model.to(device)
```

现在，将图像和文本提示传给数据处理器，然后将处理后的输入传给 `generate` 方法。请注意，每个模型都有自己的提示模板，请务必根据模型选用正确的模板，以避免性能下降。

```python
from PIL import Image
import requests

url = "https://github.com/haotian-liu/LLaVA/blob/1a91fc274d7c35a9b50b3cb29c4247ae5837ce39/images/llava_v1_5_radar.jpg?raw=true"
image = Image.open(requests.get(url, stream=True).raw)
prompt = "[INST] <image>\nWhat is shown in this image? [/INST]"

inputs = processor(prompt, image, return_tensors="pt").to(device)
output = model.generate(**inputs, max_new_tokens=100)
```

调用 `decode` 对输出词元进行解码。

```python
print(processor.decode(output[0], skip_special_tokens=True))
```

### Fine-tuning Vision Language Models with TRL

We will now initialize our model and tokenizer.

```python
from transformers import AutoTokenizer, AutoProcessor, TrainingArguments, LlavaForConditionalGeneration
import torch

model_id = "llava-hf/llava-1.5-7b-hf"
tokenizer = AutoTokenizer.from_pretrained(model_id)
tokenizer.chat_template = LLAVA_CHAT_TEMPLATE
processor = AutoProcessor.from_pretrained(model_id)
processor.tokenizer = tokenizer

model = LlavaForConditionalGeneration.from_pretrained(model_id, torch_dtype=torch.float16)
```

Let’s create a data collator to combine text and image pairs.

```python
class LLavaDataCollator:
    def __init__(self, processor):
        self.processor = processor

    def __call__(self, examples):
        texts = []
        images = []
        for example in examples:
            messages = example["messages"]
            text = self.processor.tokenizer.apply_chat_template(
                messages, tokenize=False, add_generation_prompt=False
            )
            texts.append(text)
            images.append(example["images"][0])

        batch = self.processor(texts, images, return_tensors="pt", padding=True)

        labels = batch["input_ids"].clone()
        if self.processor.tokenizer.pad_token_id is not None:
            labels[labels == self.processor.tokenizer.pad_token_id] = -100
        batch["labels"] = labels

        return batch

data_collator = LLavaDataCollator(processor)
```

Load our dataset.

```python
from datasets import load_dataset

raw_datasets = load_dataset("HuggingFaceH4/llava-instruct-mix-vsft")
train_dataset = raw_datasets["train"]
eval_dataset = raw_datasets["test"]
```

Initialize the SFTTrainer, passing in the model, the dataset splits, PEFT configuration and data collator and call `train()`. To push our final checkpoint to the Hub, call `push_to_hub()`.

```python
from trl import SFTTrainer

trainer = SFTTrainer(
    model=model,
    args=training_args,
    train_dataset=train_dataset,
    eval_dataset=eval_dataset,
    dataset_text_field="text",  # need a dummy field
    tokenizer=tokenizer,
    data_collator=data_collator,
    dataset_kwargs={"skip_prepare_dataset": True},
)

trainer.train()
```



Save the model and push to the Hugging Face Hub.

```python
trainer.save_model(training_args.output_dir)
trainer.push_to_hub()
```