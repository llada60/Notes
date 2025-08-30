[TOC]

## Special Tokens

### 1. `<pad>` / `pad_token`

- **作用**：
   用来把 batch 里的不同长度序列补齐到相同长度。
- **训练/推理时处理**：
  - `attention_mask=0`（屏蔽注意力）
  - `labels=-100`（loss 忽略）
- **备注**：
   很多 LLaMA 类模型没有定义 pad_token → 常用 `<eos>` 来替代。

------

## 2. `<eos>` / `eos_token` (End of Sequence)

- **作用**：
   表示序列结束。训练时，模型学会在生成末尾输出 `<eos>`。
- **用途**：
  - 停止生成的标志（推理时 `generate()` 遇到 `<eos>` 就会停止）。
  - 在没有 pad_token 时，经常被用来做 padding。

------

## 3. `<bos>` / `bos_token` (Beginning of Sequence)

- **作用**：
   表示序列开始，一些模型会在输入最前面加 `<bos>`。
- **用途**：
  - 给 decoder 一个统一的起始符号。
  - 并非所有模型都会用，比如 LLaMA 默认有 `<s>` (start)，但实际训练时是否用要看实现。

------

## 4. `<unk>` / `unk_token` (Unknown Token)

- **作用**：
   当 tokenizer 遇到词表里不存在的字符串时，就会转成 `<unk>`。
- **用途**：
  - 保证任何输入都有映射。
  - 不希望太常出现，说明 tokenizer 词表覆盖不足。

------

## 5. `<cls>` (Classification Token)

- **作用**：
   在一些模型（如 BERT）里用来聚合整句语义，最后的 hidden state 输入分类头。
- **用途**：
  - NLP 任务（分类/句子对匹配等）。
  - 在 decoder-only 模型（LLaMA）中一般没有。

------

## 6. `<sep>` (Separator Token)

- **作用**：
   在一些模型（如 BERT、RoBERTa）里，用来分隔两段句子。
- **用途**：
  - “句子对”任务。
  - LLaMA 系列通常没有 sep。

------

## 7. `<mask>` (Mask Token)

- **作用**：
   预训练 Masked LM（如 BERT）时用来替换被遮掉的 token。
- **用途**：
  - MLM 任务：模型要预测 `<mask>` 位置的真实 token。
  - Decoder-only 模型一般不需要。