# NVIDIA Nemotron 3 Nano Omni 30B A3B Reasoning

**Source (GGUF):** https://huggingface.co/unsloth/NVIDIA-Nemotron-3-Nano-Omni-30B-A3B-Reasoning-GGUF  
**Source (official):** https://huggingface.co/nvidia/Nemotron-3-Nano-Omni-30B-A3B-Reasoning-BF16  
**Type:** Mamba2-Transformer Hybrid MoE — Multimodal  
**License:** NVIDIA Open Model Agreement (commercial use permitted)  
**Developer:** NVIDIA  
**Released:** April 28, 2026  
**Guide:** https://unsloth.ai/docs/models/nemotron-3-nano-omni

---

## Specs

| Field | Value |
|---|---|
| Total Parameters | ~31B |
| Active Parameters (per token) | 3B |
| Architecture | Mamba2-Transformer Hybrid MoE |
| Context Window | 256K tokens |
| Language Support | English only |
| Modalities | Video, Audio, Image, Text → Text |
| Vision Encoder | CRADIO v4-H |
| Speech Encoder | Parakeet |
| Output | Text, JSON, reasoning (CoT), tool calls, word-level timestamps |

### Input Specifications
| Modality | Format | Limits |
|---|---|---|
| Video | mp4 | Up to 2 min; 1080p @ 1 FPS / 128 frames |
| Audio | wav, mp3 | Up to 1 hour, 8kHz+ |
| Image | jpeg, png | RGB |
| Text | String | 256K context |

---

## Inference Modes

| Mode | Temperature | Top-p | Top-k | Max Tokens | Reasoning Budget |
|---|---|---|---|---|---|
| Thinking (reasoning) | 0.6 | 0.95 | — | 20,480 | 16,384 |
| Instruct (fast) | 0.2 | — | 1 | 1,024 | — |

---

## VRAM Requirements

| Format | Disk / VRAM |
|---|---|
| BF16 | ~62 GB |
| FP8 | ~31 GB |
| NVFP4 | ~16 GB |
| **GGUF Q4_K_M** | **~17 GB** |
| **GGUF Q5_K_M** | **~21 GB** |

---

## Supported Hardware

| NVIDIA Generation | Hardware |
|---|---|
| Blackwell | B200, RTX PRO 6000 SE, DGX Spark, Jetson Thor, RTX 5090 |
| Hopper | H100, H200 |
| Ampere | A100 80GB |
| Lovelace | L40S |

---

## Deployment

### Download weights (BF16)
```bash
pip install -U "huggingface_hub[hf_xet]"
hf auth login

hf download nvidia/Nemotron-3-Nano-Omni-30B-A3B-Reasoning-BF16 \
  --local-dir ./Nemotron-3-Nano-Omni-30B-A3B-Reasoning-BF16 \
  --max-workers 8
```

### Ollama (GGUF via Unsloth)
Available via Unsloth GGUF repo — check for Ollama Modelfile.

### vLLM / TensorRT-LLM / SGLang / llama.cpp / NeMo
All supported.

### NGC Container
```
catalog.ngc.nvidia.com/orgs/nim/teams/nvidia/containers/nemotron-3-nano-omni-30b-a3b-reasoning
```

### NVIDIA API
https://build.nvidia.com/nvidia/nemotron-3-nano-omni-30b-a3b-reasoning

---

## All Weight Variants

| Format | HuggingFace |
|---|---|
| BF16 (official) | https://huggingface.co/nvidia/Nemotron-3-Nano-Omni-30B-A3B-Reasoning-BF16 |
| FP8 (official) | https://huggingface.co/nvidia/Nemotron-3-Nano-Omni-30B-A3B-Reasoning-FP8 |
| NVFP4 (official) | https://huggingface.co/nvidia/Nemotron-3-Nano-Omni-30B-A3B-Reasoning-NVFP4 |
| **GGUF (Unsloth)** | **https://huggingface.co/unsloth/NVIDIA-Nemotron-3-Nano-Omni-30B-A3B-Reasoning-GGUF** |

---

## Use Cases (Per NVIDIA)

- Customer service (video/OCR — drive-thru verification, delivery proof)
- Media & Entertainment — dense video captions, search, summarization
- **Document intelligence** — contracts, SOW/MSA, scientific, financial docs
- GUI automation — agentic browser/email agents

---

## Frank Evaluation Notes

- **Fit for Frank Lite:** ⚠️ GGUF Q4 ~17GB — just over frank-gpu 14GB limit. Fits on A40 8GB upgrade.
- **English only** — no multilingual concern for Australian HR/IR use
- **256K context** — handles full EBAs
- **Document intelligence** listed as primary use case — directly relevant to Frank
- **3B active params** — very efficient inference despite 30B total
- **Mamba2 hybrid architecture** — stronger long-context recall than pure transformers
- **Multimodal:** Video/audio/image not needed for Frank but future-proofs for document scanning workflows
- **Verdict:** ★★★★ Strong candidate once GPU upgraded. The document intelligence + 3B active efficiency is compelling. Monitor NVFP4 variant (~16GB) for tight fit on current hardware.

---

## Official NVIDIA Hardware Minimum Requirements

**Source:** https://huggingface.co/nvidia/Nemotron-3-Nano-Omni-30B-A3B-Reasoning-BF16

| Precision | Size | Minimum GPU | Recommended GPU |
|---|---|---|---|
| **BF16** | 62 GB | 1× H100 80GB | 1× B200 / 1× H200 |
| **FP8** | 33 GB | 1× L40S 48GB | 1× RTX Pro 6000 / 1× B200 |
| **NVFP4** | 21 GB | 1× RTX 5090 32GB | DGX Spark / Jetson Thor |

### Component Models (used in training)
- Backbone: [Nemotron-3-Nano-30B-A3B](https://huggingface.co/nvidia/NVIDIA-Nemotron-3-Nano-30B-A3B-BF16)
- Vision encoder: [CRADIO v4-H](https://huggingface.co/nvidia/C-RADIOv4-H)
- Speech encoder: [Parakeet TDT 0.6B v2](https://huggingface.co/nvidia/parakeet-tdt-0.6b-v2)

### NGC NIM Container
```
catalog.ngc.nvidia.com/orgs/nim/teams/nvidia/containers/nemotron-3-nano-omni-30b-a3b-reasoning
```

### NVIDIA API (free trial)
https://build.nvidia.com/nvidia/nemotron-3-nano-omni-30b-a3b-reasoning
