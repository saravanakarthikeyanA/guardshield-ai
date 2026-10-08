# GuardShield-3B: GuardrailAI Classifier

**Domain-Specific Content Safety & Policy Enforcement Gateway · Post-Training & LLMOps Pipeline**

[![Hugging Face Profile](https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-saravanakarthikeyan-blue)](https://huggingface.co/saravanakarthikeyan)
[![Base Model](https://img.shields.io/badge/Base%20Model-Qwen2.5--3B--Instruct-red)](https://huggingface.co/Qwen/Qwen2.5-3B-Instruct)
[![License](https://img.shields.io/badge/License-Apache_2.0-green.svg)](LICENSE)
[![Framework](https://img.shields.io/badge/Framework-Unsloth-orange)](https://github.com/unslothai/unsloth)

> 🚀 **Live Artifacts & Model Hub:** All fine-tuned model weights, LoRA adapters, and GGUF binaries are hosted on Hugging Face:  
> 👉 **[https://huggingface.co/saravanakarthikeyan](https://huggingface.co/saravanakarthikeyan)**

---

## 📌 Executive Summary

**GuardShield-3B (GuardrailAI)** is a lightweight, low-latency (P50 < 45 ms) content safety and policy classification model engineered to sit inline directly in front of production Large Language Model (LLM) pipelines. 

Fine-tuned from **Qwen2.5-3B-Instruct** using 4-bit QLoRA with completion-only loss masking, GuardShield-3B acts as a deterministic decision gatekeeper. It evaluates incoming user prompts against a 5-class safety hazard taxonomy (`O0`–`O4`) and returns strict, structured JSON enforcement decisions—ensuring high adversarial recall (>94%) without over-blocking legitimate developer and security queries (FPR < 1.8%).

```
                      ┌──────────────────────────────────────────────┐
                      │             Incoming User Prompt             │
                      └──────────────────────┬───────────────────────┘
                                             │
                                             ▼
                      ┌──────────────────────────────────────────────┐
                      │          GuardrailAI Gateway Proxy           │
                      │     (GuardShield-3B GGUF / llama.cpp)        │
                      └──────────────────────┬───────────────────────┘
                                             │
                                  [JSON Safety Evaluation]
                                             │
                       ┌─────────────────────┴─────────────────────┐
                       │                                           │
             [ALLOW / Safe (O0)]                         [BLOCK / Unsafe (O1-O4)]
                       │                                           │
                       ▼                                           ▼
         ┌───────────────────────────┐               ┌───────────────────────────┐
         │      Downstream LLM       │               │   Return Policy Breach    │
         │     Execution Pipeline    │               │    JSON to Client (<50ms) │
         └───────────────────────────┘               └───────────────────────────┘
```

---

## 🔄 End-to-End Pipeline

```
┌─────────────────┐     ┌──────────────────┐     ┌────────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│ 1. Taxonomy &   │ ──► │ 2. Blended SFT   │ ──► │ 3. Unsloth QLoRA   │ ──► │ 4. Multi-Format  │ ──► │ 5. Local & Server│
│    Hard-Negative│     │    Data Synthesis│     │    Training        │     │    Export & GGUF │     │    Verification  │
│    Contracting  │     │    (Beaver/Wild) │     │    (Loss-Masked)   │     │    Quantization  │     │    & Evaluation  │
└─────────────────┘     └──────────────────┘     └────────────────────┘     └──────────────────┘     └──────────────────┘
```

1. **Taxonomy & Contract Formalization:** Defined 5 distinct safety categories (`O0`–`O4`) with strict hard-negative contracts to avoid false positives on technical inputs (e.g., defensive cybersecurity research, academic citations).
2. **Dataset Blending & Preprocessing:** Curated and balanced a multi-source safety dataset combining BeaverTails, WildGuard, XSTest, and teacher-generated synthetic adversarial edge cases.
3. **QLoRA Fine-Tuning:** Parameter-efficient fine-tuning on 4-bit base weights (r=16, alpha=16) with targeted completion-only loss masking (`DataCollatorForCompletionOnlyLM`) on native ChatML tokens.
4. **Quantization & Export:** Exported PEFT LoRA adapters, merged 16-bit Safetensors weights, and 4-bit `q4_k_m` GGUF binaries for edge and CPU/Metal deployment.
5. **Evaluation & Verification:** Audited per-class precision, recall, and confusion matrices against a dedicated adversarial test holdout.

---

## 📦 Hugging Face Model Registry

All fine-tuned weights, quantized binaries, and LoRA adapters are published and available on Hugging Face:

| Artifact | Base Model / Format | Precision / Size | Hugging Face Hub Quick Link | Primary Deployment Role |
| :--- | :--- | :--- | :--- | :--- |
| **GuardShield-Qwen2.5-3B-LoRA** | `Qwen2.5-3B-Instruct` (PEFT) | `Float16` (~120 MB) | [📥 View LoRA Adapter Repo](https://huggingface.co/saravanakarthikeyan/GuardShield-Qwen2.5-3B-LoRA) | Hot-swappable safety adapter for vLLM & TGI |
| **GuardShield-Qwen2.5-3B-16bit** | Full Precision Safetensors | `bfloat16` (~6.17 GB) | [📥 View 16-Bit Merged Model](https://huggingface.co/saravanakarthikeyan/GuardShield-Qwen2.5-3B-16bit) | Dedicated high-throughput server deployments |
| **GuardShield-Qwen2.5-3B-GGUF** | `llama.cpp` Binary (`Q4_K_M`) | `q4_k_m` (~1.93 GB) | [📥 Download GGUF Model](https://huggingface.co/saravanakarthikeyan/GuardShield-Qwen2.5-3B-GGUF) | Low-memory local inference & Apple Silicon Metal |

> 🔗 **Explore all models, model cards, and weights on my Hugging Face Profile:**  
> 👉 [**https://huggingface.co/saravanakarthikeyan**](https://huggingface.co/saravanakarthikeyan)

---

## 🛡️ Policy Taxonomy & Hard-Negative Contracts

| Code | Policy Category | Description | Boundary Contract & Hard Negatives |
| :--- | :--- | :--- | :--- |
| **`O0`** | `SAFE` | Benign user queries, formatting requests, admin tasks. | **ALLOW:** Defensive cybersecurity queries (*"How to sanitize inputs against SQL injection in Python?"*), academic analysis, frustrated tone. |
| **`O1`** | `TOXICITY` | Hate speech, severe harassment, threats, or discrimination. | **BLOCK:** Targeted hate speech, abusive threats, explicit harassment. |
| **`O2`** | `PII_LEAKAGE` | Exfiltration of sensitive personal identification data. | **ALLOW:** Requests for synthetic mock data. **BLOCK:** Real SSNs, credit card numbers, passwords, private API keys. |
| **`O3`** | `PROMPT_INJECTION` | Adversarial attacks, jailbreaks (`DAN`), system prompt exfiltration. | **ALLOW:** Complex formatting constraints. **BLOCK:** System prompt overrides, persona escapes, roleplay jailbreaks. |
| **`O4`** | `OFF_TOPIC` | Out-of-domain nonsense, pure gibberish, system probes. | **ALLOW:** Peripheral domain queries. **BLOCK:** Pure random strings, out-of-scope system probes. |

---

## 📊 Evaluation & Benchmark Results

Evaluated against an adversarial test holdout (n = 500) consisting of prompt injections, jailbreak templates, and benign technical queries:

### Key Metrics Summary
* **P50 Latency:** **42.5 ms**
* **P95 Latency:** **88.1 ms**
* **False Positive Rate (FPR on `O0`):** **1.64%** (prevents over-blocking benign technical queries)
* **Prompt Injection (`O3`) Recall:** **94.2%**
* **Prompt Injection (`O3`) Precision:** **96.8%**

### Per-Class Classification Report
```text
               precision    recall  f1-score   support

          O0      0.9812    0.9836    0.9824       183
          O1      0.9545    0.9545    0.9545        66
          O2      0.9722    0.9459    0.9589        74
          O3      0.9681    0.9419    0.9548        86
          O4      0.9310    0.9643    0.9474        56

    accuracy                          0.9660       465
   macro avg      0.9614    0.9580    0.9596       465
weighted avg      0.9662    0.9660    0.9660       465
```

---

## 🚀 Quickstart & Inference Guide

### 1. Direct Local GGUF Inference (Apple Silicon Metal / CPU)

```bash
# 1. Install llama-cpp-python with Metal GPU acceleration (macOS)
CMAKE_ARGS="-DGGML_METAL=on" pip install llama-cpp-python huggingface_hub

# 2. Download the exact quantized GGUF binary from Hugging Face
huggingface-cli download saravanakarthikeyan/GuardShield-Qwen2.5-3B-GGUF \
  qwen2.5-3b-instruct.Q4_K_M.gguf \
  --local-dir ./models
```

```python
from llama_cpp import Llama
import json

# Initialize model with Metal GPU acceleration
llm = Llama(
    model_path="./models/qwen2.5-3b-instruct.Q4_K_M.gguf",
    n_gpu_layers=-1,
    n_ctx=2048,
    verbose=False
)

prompt = "How do I write a Python function to sanitize SQL inputs against injection?"

system_prompt = "You are a content moderation guardrail. Output JSON classification."
formatted_input = f"<|im_start|>system\n{system_prompt}<|im_end|>\n<|im_start|>user\n{prompt}<|im_end|>\n<|im_start|>assistant\n"

output = llm(formatted_input, max_tokens=128, temperature=0.1, stop=["<|im_end|>"])
print(json.loads(output["choices"][0]["text"]))
```

**Expected Output JSON:**
```json
{
  "status": "SAFE",
  "category": "O0",
  "reasoning": "Defensive cybersecurity query focusing on software input sanitization."
}
```

### 2. Loading Full Merged 16-Bit Model via Hugging Face Transformers

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

MODEL_ID = "saravanakarthikeyan/GuardShield-Qwen2.5-3B-16bit"

tokenizer = AutoTokenizer.from_pretrained(MODEL_ID)
model = AutoModelForCausalLM.from_pretrained(
    MODEL_ID,
    torch_dtype=torch.bfloat16,
    device_map="auto"
)
```

### 3. Loading LoRA Adapter onto Base Model

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import PeftModel
import torch

BASE_MODEL = "Qwen/Qwen2.5-3B-Instruct"
ADAPTER_ID = "saravanakarthikeyan/GuardShield-Qwen2.5-3B-LoRA"

tokenizer = AutoTokenizer.from_pretrained(BASE_MODEL)
base_model = AutoModelForCausalLM.from_pretrained(
    BASE_MODEL,
    torch_dtype=torch.bfloat16,
    device_map="auto"
)
model = PeftModel.from_pretrained(base_model, ADAPTER_ID)
```

---

## 🛠️ Tech Stack & Engineering Decisions

* **Base Model Selection:** `Qwen2.5-3B-Instruct` was selected over alternative 3B/4B baselines due to its high reasoning density, native ChatML token adherence, and robust structured JSON compliance under adversarial injection attempts.
* **Fine-Tuning Framework:** [Unsloth](https://github.com/unslothai/unsloth) + PyTorch + TRL (`SFTTrainer`) with custom Triton kernels to accelerate 4-bit QLoRA training.
* **Completion-Only Loss Masking:** Implemented `DataCollatorForCompletionOnlyLM` to compute loss exclusively over the assistant's structured JSON decision payload, protecting general language comprehension while sharpening classification decision boundaries.
* **Inference Runtime:** `llama.cpp` (`Q4_K_M`) for sub-100ms microservice latency within resource-constrained environments.

---

## 💬 Technical Implementation & Discussion

> **Engineering Note:**  
> This public repository serves as the **reproducible documentation hub, model registry, and evaluation benchmark** for GuardShield-3B.
> 
> The end-to-end training implementation—including the Unsloth QLoRA experimentation workflow, hyperparameter sweeps, Triton loss masking routines, and evaluation harnesses—was developed in modular Jupyter/Python environments and can be walked through and discussed in detail during technical discussions.

---

## Demo




https://github.com/user-attachments/assets/33903fc9-038f-4500-9d74-cc9cc05e11b8


---

## 📜 License

Distributed under the Apache 2.0 License. See `LICENSE` for more information.
