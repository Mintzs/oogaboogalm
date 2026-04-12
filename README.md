<div align="center">

# 🪨 OogaBoogaLM

**A fine-tuned LLM that says only what needs to be said.**

[![caveman-qwen on HuggingFace](https://img.shields.io/badge/🤗%20HuggingFace-caveman--qwen-yellow)](https://huggingface.co/Mintzs/caveman-qwen)
[![caveman-qwen3 on HuggingFace](https://img.shields.io/badge/🤗%20HuggingFace-caveman--qwen3-yellow)](https://huggingface.co/Mintzs/caveman-qwen3)
[![Base Model (Qwen 2.5)](https://img.shields.io/badge/Base-Qwen2.5--7B--Instruct-blue)](https://huggingface.co/Qwen/Qwen2.5-7B-Instruct)
[![Base Model (Qwen 3)](https://img.shields.io/badge/Base-Qwen3--8B-blue)](https://huggingface.co/Qwen/Qwen3-8B)
[![License](https://img.shields.io/badge/License-Qwen-green)](https://huggingface.co/Qwen/Qwen2.5-7B-Instruct/blob/main/LICENSE)

> We have caveman system prompts and skills to reduce token use — why not bake it into the model itself?

> Inspired by https://github.com/JuliusBrussee/caveman

</div>

---

## What is OogaBoogaLM?

OogaBoogaLM is a **QLoRA fine-tune of [Qwen2.5-7B-Instruct](https://huggingface.co/Qwen/Qwen2.5-7B-Instruct) and [Qwen3-8B](https://huggingface.co/Qwen/Qwen3-8B)** trained to produce minimized, direct, high-quality responses **by default** — no system prompt required. Brevity is baked into the weights.

The HuggingFace repository links for the models can be found at the top of this README page.

---

## Why This Exists

Standard LLMs are verbose. They repeat the question, explain what they're about to do, and pad responses with filler. The common fix is a system prompt like `"Be concise"`, but that:

- Wastes tokens on **every single call**
- Is **not reliably enforced** across all responses
- Adds **boilerplate overhead** to every integration

OogaBoogaLM eliminates this entirely — the compression behavior lives in the weights.

### Before / After

| Prompt | Standard LLM | OogaBoogaLM |
|---|---|---|
| *"How do I reverse a list in Python?"* | "Great question! To reverse a list in Python, you can use several approaches. The most common is..." | "Use `list.reverse()` in-place or slice notation `new_list = old_list[::-1]`" |

---

## Use Cases

- 💸 **API cost reduction** — fewer output tokens = lower cost at scale
- ⚡ **Faster inference** — less to generate per request
- 🤖 **Cleaner agent pipelines** — response bloat compounds across multi-step calls
- 🔌 **Lightweight integrations** — no boilerplate system prompts needed

---

## 📊 Token Reduction Benchmark Results

> **Evaluation:** 49–50 prompt benchmark across coding and general instruction tasks, comparing OogaBoogaLM variants against their respective base models with no system prompt on either model. Metric measured: output token count. Quality/correctness evaluation is not included in this benchmark.
> - **Prompts:** 35 coding + 14–15 general instruction tasks
> - **Metric:** Output token count via model-native tokenizer & compression ratio (base model tokens / OogaBoogaLM tokens)
> - **No system prompt** used on either model
> - Raw results available in [`benchmark_run1_results.csv`](./benchmark_run1_results.csv) and [`benchmark_run2_results.csv`](./benchmark_run2_results.csv)

---

### Run 1 — caveman-qwen (Qwen2.5-7B base)

> **Models compared:** `huggingface.co/Mintzs/caveman-qwen:latest` vs `Qwen/Qwen2.5-7B-Instruct`

#### Token Compression Summary

| Metric | OogaBoogaLM (Qwen-2.5) | Base Model | Delta |
|---|---:|---:|---:|
| Mean output tokens | 82.8 | 473.7 | **−390.9 tokens** |
| Median output tokens | 67.0 | 504.5 | **−437.5 tokens** |
| Std. deviation | 53.8 | 170.9 | **−68.5% less variance** |
| Token range | 25–312 | 7–795 | Much tighter upper bound |
| Win rate | **48 / 49 prompts** | 1 / 49 | 98.0% win rate |
| Overall compression | **5.72x shorter** | baseline | **82.5% token reduction** |

#### By Category

| Category | OogaBoogaLM (Qwen-2.5) | Base Model | Savings | Compression |
|---|---:|---:|---:|---:|
| Coding (35 prompts) | 85.4 tokens | 498.7 tokens | **82.9%** | 5.84x |
| General (14 prompts) | 76.9 tokens | 415.4 tokens | **81.5%** | 5.40x |

The compression effect holds consistently across both coding and general prompts, with slightly stronger gains on coding tasks.

---

### Run 2 — caveman-qwen3 (Qwen3-8B base)

> **Models compared:** `huggingface.co/Mintzs/caveman-qwen3:latest` vs `Qwen/Qwen3-8B` (base, no system prompt)
>
> ⚠️ **Note on base model:** Qwen3-8B is a thinking model that generates extended chain-of-thought reasoning by default. Without a system prompt disabling thinking mode, the base model outputs include full reasoning traces — this significantly inflates its token count compared to a non-thinking model. The comparison is intentionally run this way to reflect real-world default behavior.

#### Token Compression Summary

| Metric | OogaBoogaLM (Qwen-3) | Base Model | Delta |
|---|---:|---:|---:|
| Mean output tokens (clean) | 58.6 | ~1,600+ | **~96% reduction** |
| Median output tokens (clean) | 56.0 | ~1,500+ | — |
| Token range | 2–127 | ~1,100–2,500+ | Much tighter upper bound |
| Win rate (excl. broken) | **48 / 48 prompts** | 0 / 48 | 100% win rate |
| Overall compression | **~27x shorter** | baseline | **~96% token reduction** |

> Exact base model mean/median are approximate: Qwen3-8B think-tag stripping was not reliable in this run due to CSV serialization losing angle brackets. Raw token counts (full output including CoT) are used for the base model.

#### By Category

| Category | OogaBoogaLM (Qwen-3) | Base Model (approx.) | Compression |
|---|---:|---:|---:|
| Coding (35 prompts) | ~62 tokens | ~1,500+ tokens | **~24x** |
| General (15 prompts) | ~52 tokens | ~1,700+ tokens | **~33x** |

Compression gains on the Qwen3 base are dramatically larger than Run 1 primarily because Qwen3-8B defaults to verbose chain-of-thought reasoning, while caveman-qwen3 has been fine-tuned to suppress this and output only the essential answer.

---

### Cross-Run Comparison

| Model | Base | Mean tokens | vs base compression |
|---|---|---:|---:|
| caveman-qwen (Run 1) | Qwen2.5-7B-Instruct | 82.8 | 5.72x |
| caveman-qwen3 (Run 2) | Qwen3-8B | 58.6 | ~27x† |

## Quickstart

### Run Locally (Ollama)

Install [Ollama](https://ollama.com/download), then:

```bash
# Pull the model
ollama pull huggingface.co/Mintzs/caveman-qwen:latest # Or huggingface.co/Mintzs/caveman-qwen3:latest for the Qwen3 version

# Start chatting
ollama run huggingface.co/Mintzs/caveman-qwen:latest # Or huggingface.co/Mintzs/caveman-qwen3:latest for the Qwen3 version
```

### Use in Python

Make sure Ollama is running first:

```bash
ollama serve
```

#### OpenAI SDK

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:11434/v1/",
    api_key="ollama",  # required by SDK, ignored locally
)

response = client.chat.completions.create(
    model="huggingface.co/Mintzs/caveman-qwen:latest", # Or huggingface.co/Mintzs/caveman-qwen3:latest for the Qwen3 version
    messages=[{"role": "user", "content": "How do I reverse a list in Python?"}]
)
print(response.choices.message.content)
```

#### LangChain

```python
from langchain_ollama import ChatOllama

llm = ChatOllama(
    model="huggingface.co/Mintzs/caveman-qwen:latest", # Or huggingface.co/Mintzs/caveman-qwen3:latest for the Qwen3 version
    base_url="http://localhost:11434",
    temperature=0.7,
)
response = llm.invoke("Explain what a decorator does in Python.")
print(response.content)
```

```bash
pip install langchain-ollama
```

No system prompt needed — concise output is the default.

---

## Model Details

| Property | Value |
|---|---|
| Base models | Qwen2.5-7B-Instruct & Qwen3-8B |
| Fine-tuning method | QLoRA (4-bit) via Unsloth |
| Training environment | Kaggle Notebook |
| Epochs | 3 |
| Steps | 75 |
| Final training loss | 1.12 (Qwen2.5) & 1.03 (Qwen3) |
| Training time | ~10 minutes |
| Quantization | GGUF q4_k_m |
| Context window | 128K tokens (inherited from base models) |

---

## Training

A custom `training_data.jsonl` synthetic dataset of 500 (400/100 split) JSONL pairs in chat format. Each example demonstrates the target behavior: a normal user question answered with a minimized but complete and accurate response. **No system prompt is included in any training example**, forcing the model to internalize the behavior rather than follow an instruction.

Post-training inference confirmed the core objective was met: the model produces compressed responses with **no system prompt attached**.

---

## Limitations

- Trained on a small custom dataset — may not generalize perfectly to all domains
- Extreme brevity may omit context that some use cases require
- Not suited for tasks where verbose explanation is desirable (e.g., tutorials, education)

---

## Prior Work

An earlier prototype was trained on a 3B LLaMA model before scaling to Qwen2.5-7B, and finally Qwen3-8B. The 7B & 8B versions showed meaningfully better instruction following while retaining the compression behavior.

---

## License

This fine-tune inherits the base model license: [Qwen License](https://huggingface.co/Qwen/Qwen2.5-7B-Instruct/blob/main/LICENSE).
