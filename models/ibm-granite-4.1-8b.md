# IBM Granite 4.1 8B

**Source:** https://huggingface.co/ibm-granite/granite-4.1-8b  
**Type:** Dense instruct  
**License:** Apache 2.0  
**Developer:** IBM Granite Team  
**Released:** April 29, 2026

---

## Specs

| Field | Value |
|---|---|
| Parameters | 8B |
| Architecture | Dense transformer |
| Training | SFT + RL alignment |
| Context Window | Long-context (check model card) |
| Modality | Text |
| Languages | English, German, Spanish, French, Japanese, Portuguese, Arabic, Czech, Italian, Korean, Dutch, Chinese |

---

## Capabilities

- Summarization
- Text classification & extraction
- Question answering
- **RAG (Retrieval Augmented Generation)** — explicitly listed
- Code tasks (FIM code completion)
- **Function/tool calling** — enhanced in 4.1
- Multilingual dialog
- Instruction following

---

## Benchmarks (8B Dense)

### General
| Benchmark | Score |
|---|---|
| MMLU (5-shot) | 73.84 |
| MMLU-Pro (5-shot, CoT) | 55.99 |
| BBH (3-shot, CoT) | 80.51 |
| AGI Eval (0-shot, CoT) | 72.43 |
| GPQA (0-shot, CoT) | 41.96 |

### Alignment
| Benchmark | Score |
|---|---|
| AlpacaEval 2.0 | 50.08 |
| IFEval Avg | 87.06 |
| ArenaHard | 68.98 |
| MT-Bench Avg | **8.61** |

### Math
| Benchmark | Score |
|---|---|
| GSM8K (8-shot) | 92.49 |
| Minerva Math (0-shot, CoT) | 80.10 |
| DeepMind Math (0-shot, CoT) | 80.07 |

### Code
| Benchmark | Score |
|---|---|
| HumanEval (pass@1) | 85.37 |
| MBPP (pass@1) | 87.30 |
| Eval+ Avg | 80.21 |

### Tool Calling
| Benchmark | Score |
|---|---|
| BFCL v3 | 68.27 |

### Safety
| Benchmark | Score |
|---|---|
| SALAD-Bench | **95.80** |
| AttaQ | 81.19 |
| Tulu3 Safety Eval Avg | 75.57 |

---

## VRAM Requirements

| Format | VRAM |
|---|---|
| BF16 | ~16 GB |
| Q4_K_M | ~5 GB |
| Q8 | ~8 GB |

---

## Deployment

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model_path = "ibm-granite/granite-4.1-8b"
tokenizer = AutoTokenizer.from_pretrained(model_path)
model = AutoModelForCausalLM.from_pretrained(model_path, device_map="cuda")
```

### Tool calling
Uses OpenAI function definition schema with `<tool_call>` XML tags.

### Ollama
```bash
ollama pull granite4.1:8b
```

---

## Granite 4.1 Family

| Model | HuggingFace |
|---|---|
| 3B Dense | https://huggingface.co/ibm-granite/granite-4.1-3b |
| **8B Dense** | https://huggingface.co/ibm-granite/granite-4.1-8b |
| 30B Dense | https://huggingface.co/ibm-granite/granite-4.1-30b |

Full collection: https://huggingface.co/collections/ibm-granite/granite-41-language-models  
Blog: https://huggingface.co/blog/ibm-granite/granite-4-1  
GitHub: https://github.com/ibm-granite/granite-4.1-language-models  
Docs: https://www.ibm.com/granite/docs/

---

## Frank Evaluation Notes

- **Fit for Frank Lite:** ✅ Excellent — 8B fits in 5GB VRAM at Q4, leaves plenty of headroom alongside ChromaDB
- **License:** Apache 2.0 — fully commercial, no restrictions
- **RAG-explicit:** One of the few models that lists RAG as a primary capability
- **Safety scores strong:** 95.80 SALAD-Bench — important for enterprise Frank deployments
- **Tool calling:** BFCL v3 68.27 — solid for structured Frank responses
- **MT-Bench 8.61** — comparable to much larger models
- **Verdict:** ★★★★★ Strong Frank Lite candidate. Compact, safe, Apache 2.0, explicitly RAG-optimised. Benchmark against frank:v3 (Llama 3.1 8B) — may outperform on instruction following.
