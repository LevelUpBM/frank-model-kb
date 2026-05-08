# Frank Model KB — Analysis & Recommendations
**Date:** 2026-05-08  
**Updated:** Dual-objective evaluation — Frank Lite inference + background processing

---

## The Two Jobs

### Job 1 — Frank Lite Chat / RAG Inference
Real-time user-facing responses. User asks a legal/HR question, Frank retrieves from corpus, generates a grounded, cited answer.

**Requirements:**
- Long context (128K+ to handle full EBAs in context)
- RAG quality — grounds answers in retrieved chunks, cites sources
- Instruction following — structured responses with citations
- Real-time latency (~5-15s acceptable)
- Fits on frank-gpu (14GB now, ~24GB after upgrade)
- Commercial licence (Apache 2.0 or equivalent)

### Job 2 — Background Processing / Ingest Engine
Replaces Anthropic Haiku API ($0.80/MTok input) across all background tasks currently burning API credits.

**Current Haiku workloads:**
| Process | Volume | Current cost |
|---|---|---|
| Proposition extraction | ~10k docs | ~$5-10/run |
| ATF cross-reference tagging | ~750k chunks | ~$15-20/run |
| Fine-tune pair generation (SFT/DPO) | 5,000 pairs | ~$26/run |
| DREAM memory consolidation (FrankLife) | Daily per user | ongoing |
| FrankScout / MarketScout analysis | Weekly runs | ongoing |
| NCG bot responses | Per query | ongoing |
| FrankBot client responses | Per query | ongoing |
| BuildAI trade extraction | Per project | ongoing |

**Requirements:**
- Good structured output (JSON extraction, classification)
- Fast batch throughput (not real-time — can queue)
- Low VRAM — can run on CPU worker (32GB RAM) or cheap GPU
- Commercial licence
- Cost: needs to be effectively $0 (self-hosted replaces API spend)

---

## Models Assessed — Dual-Job Fit

| Model | Job 1 (Chat) | Job 2 (Background) | Why |
|---|---|---|---|
| IBM Granite 4.1 8B | ★★★★★ | ★★★★★ | Fits both — RAG-explicit, tool calling, 5GB VRAM, Apache 2.0 |
| NVIDIA Nemotron 30B NVFP4 | ★★★★★ | ★★★ | Great for chat, overkill + slow for background batch |
| Poolside Laguna XS.2 | ★★★ | ★★★ | Coding-focused, Apache 2.0, 20GB — not ideal for either job |
| AEON Qwen3.6-27B Ultimate | ★★★★ | ★★★ | Compliance risk for enterprise; good quality otherwise |
| Mistral Medium 3.5 128B | ★★★★★ | ★★ | Too large for local; API cost defeats the point for Job 2 |
| Xiaomi MiMo-V2.5 | ★★ | ★★ | Multi-GPU only; no local path for either job |

---

## The Missing Models — What to Add for Job 2

The 6 models evaluated are mostly large flagship models — not ideal for cheap background processing. These are the models to add to the KB for Job 2:

### Tier 1 — Best local background processors

| Model | Params | VRAM | Context | Why |
|---|---|---|---|---|
| **Qwen 2.5 7B Instruct** | 7B | ~5 GB | 128K | Best-in-class structured output at 7B. Already used in pipeline. |
| **Qwen 2.5 3B Instruct** | 3B | ~2 GB | 128K | Ultra-cheap batch processing. JSON extraction tasks. |
| **Granite 4.1 3B** | 3B | ~2 GB | Long-ctx | IBM's tiny version — same RAG/tool calling capability as 8B |
| **Llama 3.2 3B Instruct** | 3B | ~2 GB | 128K | Meta's fastest small model. Function calling. Apache 2.0. |
| **SmolLM2 1.7B** | 1.7B | ~1 GB | 8K | Hugging Face's tiny instruction model. Classification tasks. |
| **Phi-3.5 Mini 3.8B** | 3.8B | ~2.5 GB | 128K | Microsoft MIT — strong instruction following at tiny size |

### Tier 2 — Medium background (proposition extraction, pair generation)

| Model | Params | VRAM | Context | Why |
|---|---|---|---|---|
| **Qwen 2.5 14B Instruct** | 14B | ~9 GB | 128K | Best quality for complex extraction. Fits in 14GB easily. |
| **Granite 4.1 8B** | 8B | ~5 GB | Long-ctx | Current recommendation — can serve both jobs on same GPU |
| **Gemma 3 4B Instruct** | 4B | ~3 GB | 128K | Google quality at tiny size. Strong structured output. |

---

## Recommended Architecture

### Current State (14GB frank-gpu)

```
frank-gpu (14GB)
├── ChromaDB           ~6GB
├── frank-chat         ~3GB  (Llama 3.1 8B — frank:v3)
└── headroom           ~5GB  ← WASTED

cpu-worker (32GB RAM, no GPU)
└── EBA consumers (embedding via sentence-transformers)
```

### Proposed State — Two Models, One GPU

```
frank-gpu (14GB)
├── ChromaDB            ~6GB
├── Frank Lite model    ~5GB  (Granite 4.1 8B Q4 — chat + RAG)
└── Background model    ~2GB  (Qwen 2.5 3B or Granite 4.1 3B — batch tasks)
                        ═══
                        13GB total — fits

cpu-worker (32GB RAM)
└── EBA consumers (unchanged — sentence-transformers embedding)
    + Background batch queue (pulls from local Ollama on frank-gpu)
```

**Net result:**
- Haiku API calls replaced by local Qwen/Granite 3B → $0 ongoing
- Frank Lite quality improves (Granite 4.1 8B vs Llama 3.1 8B)
- Zero additional hardware cost
- Annual saving vs current Haiku usage: **~$500-2,000/year** depending on volume

### After GPU Upgrade (A40 ~24GB)

```
frank-gpu-upgraded (24GB)
├── ChromaDB              ~6GB
├── Frank Lite model      ~9GB  (Granite 4.1 14B or Nemotron NVFP4)  
└── Background model      ~5GB  (Granite 4.1 8B — better quality batch)
                          ════
                          20GB total — comfortable
```

---

## Job-Specific Model Recommendations

### Job 1 — Frank Lite Chat/RAG

**Now (14GB GPU):**
> **IBM Granite 4.1 8B** — 5GB Q4, Apache 2.0, RAG-explicit, MT-Bench 8.61, tool calling

**After GPU upgrade:**
> **NVIDIA Nemotron-3 Nano Omni 30B NVFP4** — document intelligence primary use case, 256K context, Mamba2 hybrid for long-context recall, 3B active params = fast inference

---

### Job 2 — Background Processing

**Immediate replacement for Haiku (all tasks):**
> **Qwen 2.5 7B Instruct** — already partially in the stack, best structured JSON output at 7B, 128K context handles long documents for proposition extraction, Apache 2.0

**For ultra-cheap classification/short tasks:**
> **Granite 4.1 3B** or **Qwen 2.5 3B** — 2GB VRAM, runs alongside everything else, handles FrankScout/DREAM/classification tasks

**Haiku replacement mapping:**

| Current Haiku task | Replace with | Savings |
|---|---|---|
| Proposition extraction | Qwen 2.5 7B local | ~$5-10/run |
| ATF cross-references | Qwen 2.5 7B local | ~$15-20/run |
| SFT pair generation | Qwen 2.5 14B local | ~$26/run |
| DREAM consolidation | Granite 4.1 3B local | ongoing |
| FrankScout analysis | Qwen 2.5 3B local | ongoing |
| NCG/FrankBot responses | Granite 4.1 8B local | ongoing |
| BuildAI extraction | Qwen 2.5 7B local | per project |

---

## Immediate Action Plan

### Phase 1 — This week (zero hardware cost)
1. Pull Granite 4.1 8B on frank-gpu: `ollama pull granite4.1:8b`
2. Pull Qwen 2.5 7B: `ollama pull qwen2.5:7b`
3. Run Frank gold benchmark: Granite 4.1 8B vs frank:v3
4. Run proposition extraction test: Qwen 2.5 7B vs Haiku — compare quality + speed
5. If Granite wins on chat → swap frank:v3; if Qwen wins on extraction → swap Haiku

### Phase 2 — After benchmark results
6. Update proposition_backfill.py, dpo_generator.py, DREAM to call local Ollama instead of Anthropic
7. Add `llm_router.py` logic: route background tasks to local; route real-time chat to local too
8. Decommission Haiku for background tasks → stop API bleeding

### Phase 3 — After GPU upgrade
9. Test Nemotron-3 Nano Omni NVFP4 for Frank Lite
10. Upgrade background model to Granite 4.1 8B (freed up by Nemotron taking the chat slot)

---

## Models Still to Add to KB

Add HuggingFace links for these to complete the picture:
- Qwen 2.5 7B Instruct — https://huggingface.co/Qwen/Qwen2.5-7B-Instruct
- Qwen 2.5 3B Instruct — https://huggingface.co/Qwen/Qwen2.5-3B-Instruct
- Granite 4.1 3B — https://huggingface.co/ibm-granite/granite-4.1-3b
- Gemma 3 4B Instruct — https://huggingface.co/google/gemma-3-4b-it
- Llama 3.2 3B Instruct — https://huggingface.co/meta-llama/Llama-3.2-3B-Instruct
- Phi-3.5 Mini 3.8B — https://huggingface.co/microsoft/Phi-3.5-mini-instruct
