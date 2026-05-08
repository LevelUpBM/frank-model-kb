# Frank Model Knowledge Base

Self-hosted LLM model specs, benchmarks, and evaluation notes for Frank Lite deployment candidates.

## Models

| File | Model | Params | Context | Fits 14GB? |
|---|---|---|---|---|
| [poolside-laguna-xs2.md](models/poolside-laguna-xs2.md) | Poolside Laguna XS.2 | 33B (3B active) | 131K | ⚠️ ~20GB |
| [xiaomi-mimo-v2.5.md](models/xiaomi-mimo-v2.5.md) | Xiaomi MiMo-V2.5 | 310B (15B active) | 1M | ❌ Multi-GPU |
| [aeon-qwen3.6-27b-ultimate.md](models/aeon-qwen3.6-27b-ultimate.md) | AEON Qwen3.6-27B Ultimate | 27B dense | 128K | ⚠️ ~20GB |
| [ibm-granite-4.1-8b.md](models/ibm-granite-4.1-8b.md) | IBM Granite 4.1 8B | 8B dense | Long-ctx | ✅ ~5GB |
| [mistral-medium-3.5-128b.md](models/mistral-medium-3.5-128b.md) | Mistral Medium 3.5 | 128B dense | 256K | ❌ API only |
| [nvidia-nemotron-3-nano-omni-30b.md](models/nvidia-nemotron-3-nano-omni-30b.md) | NVIDIA Nemotron-3 Nano Omni 30B | 31B (3B active) | 256K | ⚠️ 21GB NVFP4 |

## Analysis

See [ANALYSIS.md](ANALYSIS.md) for full evaluation and recommendations.

**TL;DR:**
- **Now:** IBM Granite 4.1 8B — fits frank-gpu, Apache 2.0, RAG-explicit, benchmark against frank:v3
- **After GPU upgrade:** NVIDIA Nemotron-3 Nano Omni 30B NVFP4 — document intelligence focus, 256K ctx
- **API tier:** Mistral Medium 3.5 — best overall, 256K ctx, configurable reasoning

## Frank GPU Constraints

- Current frank-gpu: 14GB VRAM
- Planned upgrade: A40 ~24GB VRAM
- CPU Worker: 32GB RAM (embedding only, no GPU)
