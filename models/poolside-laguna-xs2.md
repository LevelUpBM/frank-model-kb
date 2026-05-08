# Poolside Laguna XS.2

**Source:** https://huggingface.co/poolside/Laguna-XS.2  
**Type:** Mixture-of-Experts (MoE)  
**License:** Apache 2.0  
**Released:** 2025

---

## Specs

| Field | Value |
|---|---|
| Total Parameters | 33B |
| Active Parameters (per token) | 3B |
| Context Window | 131,072 tokens |
| Layers | 40 (10 global attention + 30 sliding window) |
| Experts | 256 + 1 shared |
| Sliding Window Size | 512 tokens |
| KV Cache | FP8 quantised |
| Modality | Text-to-text |
| Reasoning | Native interleaved thinking |
| Optimizer | Muon |

---

## Benchmarks

| Benchmark | Laguna XS.2 | Devstral Small 2 (24B) | Qwen3.6-35B-A3B | Claude Haiku 4.5 |
|---|---|---|---|---|
| SWE-bench Verified | 68.2% | 68.0% | 73.4% | 73.3% |
| SWE-bench Multilingual | 62.4% | 55.7% | 67.2% | — |
| SWE-bench Pro | 44.5% | — | 49.5% | 39.5% |
| Terminal-Bench 2.0 | 30.1% | 22.5% | 51.5% | 29.8% |

---

## VRAM Requirements

| Format | VRAM |
|---|---|
| BF16 (full) | ~66 GB |
| Q4_K_M | ~18 GB |
| Ollama (Q4) | ~20 GB |
| Mac MLX (36GB unified) | ✅ fits |

---

## Deployment

### Ollama
```bash
ollama pull laguna-xs.2
```

### vLLM
```bash
pip install vllm --pre --extra-index-url https://wheels.vllm.ai/nightly

VLLM_USE_DEEP_GEMM=0 vllm serve \
  --model poolside/Laguna-XS.2 \
  --tool-call-parser poolside_v1 \
  --reasoning-parser poolside_v1 \
  --enable-auto-tool-choice \
  --served-model-name laguna \
  --default-chat-template-kwargs '{"enable_thinking": true}'
```

### Transformers
```python
from transformers import AutoModelForCausalLM, AutoTokenizer
model_id = "poolside/Laguna-XS.2"
tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForCausalLM.from_pretrained(model_id, dtype=torch.bfloat16, device_map="auto")
```

---

## Frank Evaluation Notes

- **Fit for Frank Lite:** ⚠️ Needs ~20GB VRAM — exceeds current 14GB frank-gpu. Would fit on A40 8GB plan or upgraded GPU.
- **Primary strength:** Agentic coding / long-horizon tasks — not the primary Frank use case (IR/legal Q&A)
- **Positives:** 131K context covers full EBAs, Apache 2.0 licence, local-ready via Ollama
- **Verdict:** Evaluate when GPU upgraded. Not a priority over Qwen 2.5 14B or Mistral Small 3.1 for Frank.

---

## Links

- HuggingFace: https://huggingface.co/poolside/Laguna-XS.2
- Ollama: https://ollama.com/library/laguna-xs.2
- Blog: https://poolside.ai/blog/laguna-a-deeper-dive
- API: https://platform.poolside.ai
- pool CLI: https://github.com/poolsideai/pool
