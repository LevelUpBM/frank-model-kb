# Xiaomi MiMo-V2.5

**Source:** https://huggingface.co/XiaomiMiMo/MiMo-V2.5  
**Type:** Sparse MoE — Native Omnimodal  
**License:** Check HuggingFace repo  
**Developer:** Xiaomi

---

## Specs

| Field | Value |
|---|---|
| Total Parameters | 310B |
| Active Parameters (per token) | 15B |
| Context Window | 1M tokens (base: 256K) |
| Architecture | Sparse MoE, hybrid SWA + Global Attention |
| Layers | 48 (1 dense + 47 MoE) |
| Full Attention Layers | 9 |
| SWA Layers | 39 |
| SWA Window Size | 128 tokens |
| Attention Heads | 64Q / 8KV (GQA) |
| Routed Experts | 256 |
| Experts per Token | 8 |
| Modalities | Text, Image, Video, Audio |
| Vision Encoder | 729M ViT (28 layers) |
| Audio Encoder | 261M (from MiMo-Audio) |
| MTP Modules | 3 layers, 329M params (speculative decoding) |
| Training Tokens | ~48T |
| Training Precision | FP8 mixed precision |

### MiMo-V2.5-Pro (larger variant)
| Field | Value |
|---|---|
| Total Parameters | 1.02T |
| Active Parameters | 42B |
| Layers | 70 |

---

## Key Architecture Features

- **Hybrid Attention:** 5:1 SWA:GA ratio — reduces KV-cache ~6× vs full attention
- **Learnable attention sink bias** for long-context stability
- **Multi-Token Prediction:** 3 MTP modules for speculative decoding acceleration
- **Unified omnimodal:** text + image + video + audio in single architecture
- **Post-training:** SFT + large-scale agentic RL + Multi-Teacher On-Policy Distillation (MOPD)

---

## VRAM Requirements

| Format | VRAM (15B active) |
|---|---|
| BF16 full weights | ~620 GB (310B) |
| FP8 quantised | ~310 GB |
| Multi-GPU minimum | 8× H100 80GB recommended |
| Single GPU | ❌ Not feasible |

---

## Deployment

### SGLang (recommended)
```bash
python3 -m sglang.launch_server \
  --model-path XiaomiMiMo/MiMo-V2.5 \
  --tp-size 8 --dp-size 2 \
  --quantization fp8 \
  --reasoning-parser qwen3 \
  --tool-call-parser mimo \
  --context-length 262144
```

### vLLM
```bash
vllm serve XiaomiMiMo/MiMo-V2.5 --tensor-parallel-size 8
```

### Config Update Note
⚠️ If downloaded before commit `4da2748`, re-pull config files:
```bash
hf download XiaomiMiMo/MiMo-V2.5 config.json tokenizer_config.json --local-dir ./MiMo-V2.5
```

---

## Benchmarks

Benchmark data present on HuggingFace page across:
- Multimodal benchmarks
- Coding & Agent benchmarks  
- Long context benchmarks

(Fetch full tables from source page)

---

## Downloads

| Variant | Context | Link |
|---|---|---|
| MiMo-V2.5 | 1M tokens | https://huggingface.co/XiaomiMiMo/MiMo-V2.5 |
| MiMo-V2.5-Base | 256K tokens | https://huggingface.co/XiaomiMiMo/MiMo-V2.5-Base |
| ModelScope mirror | — | https://modelscope.cn/models/XiaomiMiMo/MiMo-V2.5 |

---

## Frank Evaluation Notes

- **Fit for Frank Lite:** ❌ Way too large — requires 8× H100. Not a local deployment option.
- **Primary strength:** Omnimodal agentic tasks — text, image, video, audio unified
- **1M context** is exceptional — could theoretically ingest entire EBA corpora in one pass
- **Verdict:** API-only for Frank use. Monitor for smaller distilled variants. The 15B active params architecture is interesting — if a standalone 15B text-only distill ships, re-evaluate.
- **Frank relevance:** Low near-term. High interest if Xiaomi releases a smaller instruct variant.

---

## Links

- HuggingFace: https://huggingface.co/XiaomiMiMo/MiMo-V2.5
- Pro variant: https://huggingface.co/XiaomiMiMo/MiMo-V2.5-Pro
- SGLang cookbook: https://docs.sglang.io/cookbook/autoregressive/Xiaomi/MiMo-V2.5
- GitHub: https://github.com/XiaomiMiMo/MiMo-V2-Flash
