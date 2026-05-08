# Frank Model KB — Analysis & Recommendations
**Date:** 2026-05-08  
**Purpose:** Evaluate self-hosted LLM candidates for Frank Lite deployment

---

## Models Captured

| # | Model | Params (active) | Context | VRAM (Q4) | License |
|---|---|---|---|---|---|
| 1 | Poolside Laguna XS.2 | 33B (3B) | 131K | ~20 GB | Apache 2.0 |
| 2 | Xiaomi MiMo-V2.5 | 310B (15B) | 1M | N/A (multi-GPU) | Check HF |
| 3 | AEON Qwen3.6-27B Ultimate | 27B dense | 128K | ~16 GB (Text-XS) | Check HF |
| 4 | IBM Granite 4.1 8B | 8B dense | Long-ctx | ~5 GB | Apache 2.0 |
| 5 | Mistral Medium 3.5 | 128B dense | 256K | ~70 GB | Modified MIT |
| 6 | NVIDIA Nemotron-3 Nano Omni 30B | 31B (3B) | 256K | ~21 GB (NVFP4) | NVIDIA OMA |

---

## Frank Lite Hardware Constraints

**Current frank-gpu:** 14GB VRAM (Vultr A100 equivalent)  
**Upgraded GPU (planned):** A40 8GB plan = ~24GB VRAM  
**CPU Worker:** 32GB RAM (no GPU — embedding only)

---

## Fit Assessment

### 🔴 Doesn't fit current frank-gpu (14GB)

| Model | Why | Workaround |
|---|---|---|
| Mistral Medium 3.5 128B | 70GB Q4 — needs 4× A100 | API only (Mistral/OpenRouter) |
| Xiaomi MiMo-V2.5 310B | Multi-GPU cluster required | API only — no local option |
| NVIDIA Nemotron NVFP4 | 21GB — marginal over limit | Fits on A40 upgrade / DGX Spark |
| Laguna XS.2 Q4 | ~20GB — over current limit | Fits on A40 upgrade |
| AEON Qwen3.6-27B Text-XS | ~20GB — over current limit | Fits on A40 upgrade |

### 🟡 Fits with GPU upgrade (A40 ~24GB)

| Model | VRAM needed | Notes |
|---|---|---|
| AEON Qwen3.6-27B Text-NVFP4-MTP-XS | ~20 GB | ⚠️ Compliance risk (uncensored) |
| Laguna XS.2 | ~20 GB | Apache 2.0, coding-focused |
| NVIDIA Nemotron NVFP4 | ~21 GB | Document intelligence focus |

### 🟢 Fits current frank-gpu (14GB) right now

| Model | VRAM needed | Frank Fit |
|---|---|---|
| **IBM Granite 4.1 8B Q4** | ~5 GB | ★★★★★ |

---

## Head-to-Head on Frank Criteria

Frank's requirements ranked by importance:
1. **Long context** — EBAs are 50-200K tokens
2. **Instruction following** — structured responses, citations
3. **RAG quality** — retrieves and grounds answers in source text
4. **Tool/function calling** — for future Frank agents
5. **Licence** — Apache 2.0 or equivalent for commercial use
6. **Local deployment** — fits 14-24GB VRAM

### Scoring Matrix (1-5)

| Model | Context | Instruct | RAG | Tool Call | Licence | Local | **Total** |
|---|---|---|---|---|---|---|---|
| IBM Granite 4.1 8B | 4 | 5 | 5 | 4 | 5 | 5 | **28/30** |
| Laguna XS.2 | 4 | 4 | 3 | 4 | 5 | 3 | **23/30** |
| AEON Qwen3.6-27B | 4 | 4 | 4 | 4 | 2 | 3 | **21/30** |
| NVIDIA Nemotron 30B | 5 | 4 | 5 | 4 | 3 | 3 | **24/30** |
| Mistral Medium 3.5 | 5 | 5 | 4 | 5 | 3 | 1 | **23/30** |
| MiMo-V2.5 | 5 | 4 | 3 | 4 | 2 | 1 | **19/30** |

---

## Key Insights

### 1. IBM Granite 4.1 8B — Surprise leader for Frank right now
- Only model that **fits in 5GB VRAM** leaving 9GB headroom for ChromaDB on the same GPU
- **RAG explicitly listed** as a primary capability — rare for a model to call this out
- **Safety scores highest** of any model tested (SALAD-Bench 95.80) — critical for enterprise
- **Apache 2.0** — fully commercial, no strings
- **MT-Bench 8.61** — punches well above its weight class
- **Action:** Benchmark Granite 4.1 8B against frank:v3 (current Llama 3.1 8B) immediately

### 2. NVIDIA Nemotron-3 Nano Omni 30B — Best fit after GPU upgrade
- **Document intelligence** is listed as the primary use case — perfectly aligned with Frank
- 3B active parameters means inference is fast despite 31B total
- **256K context** handles full EBA documents
- Mamba2 hybrid architecture gives better long-context recall than pure transformers
- **NVFP4 at 21GB** — fits on A40 upgrade
- OCR/chart/table understanding is a bonus for complex EBA formatting
- **Action:** Test on A40 when GPU upgraded

### 3. Poolside Laguna XS.2 — Wrong primary use for Frank
- Built for **agentic coding** — not legal/HR document Q&A
- Strong benchmarks but on coding tasks (SWE-bench, Terminal-Bench)
- 131K context adequate but not exceptional
- **Apache 2.0** is great
- **Action:** Defer — revisit for future Frank coding agent features, not core Q&A

### 4. AEON Qwen3.6-27B Ultimate — Technical excellence, compliance risk
- "Uncensored/abliterated" — **not suitable for enterprise Frank** without validation
- Risk of generating inappropriate content for HR/IR scenarios (misconduct cases, sensitive dismissals)
- Base Qwen3.6-27B architecture is strong — monitor for properly-aligned variants
- **Action:** Skip this variant. Track base Qwen3.6-27B for enterprise-aligned fine-tunes

### 5. Mistral Medium 3.5 128B — API-only play
- Best overall model in the list — SWE-bench 77.6%, 256K context, configurable reasoning
- Simply too large for any reasonable local deployment
- **Modified MIT** has carve-outs for large-revenue companies — check compliance
- **Action:** Evaluate via Mistral API for Frank Cloud/enterprise tier. Not Frank Lite.

### 6. Xiaomi MiMo-V2.5 310B — Monitor only
- Remarkable architecture (1M context, omnimodal, 15B active)
- Multi-GPU cluster requirement puts it completely out of scope for Frank Lite
- **Action:** Watch for distilled smaller variants. No near-term action.

---

## Recommended Action Plan

### Immediate (this week)
1. **Benchmark IBM Granite 4.1 8B** vs frank:v3 on Frank's gold test set
   - Pull: `ollama pull granite4.1:8b`
   - Run: `python3 /opt/frank-rag/tools/benchmark_runner.py --model granite4.1:8b`
   - Compare: accuracy, citation quality, latency

2. **Fine-tune Granite 4.1 8B** on frank_unified_v3.jsonl (once generated)
   - Apache 2.0 allows fine-tuning with no restrictions
   - 8B size = cheaper RunPod fine-tune (~$4 vs ~$12 for 70B)

### After GPU upgrade (A40 ~24GB)
3. **Test NVIDIA Nemotron NVFP4** on document intelligence tasks
   - Focus on complex EBA table/clause extraction
   - Benchmark OCR capabilities on scanned EBA PDFs

4. **Consider Laguna XS.2** for a Frank Coding Agent (separate from Frank Q&A)

### API tier (no GPU constraint)
5. **Mistral Medium 3.5** via Mistral API for enterprise Frank deployments
   - High-reasoning mode for complex legal analysis
   - 256K context for full corpus queries

---

## Verdict

**Replace frank:v3 (Llama 3.1 8B) with IBM Granite 4.1 8B as the immediate priority.**  
Same size class, Apache 2.0, explicitly RAG-optimised, higher instruction following scores, stronger safety profile. No hardware changes needed. If benchmarks confirm quality improvement, ship it.

**After GPU upgrade: NVIDIA Nemotron-3 Nano Omni 30B NVFP4** as the production Frank Lite model — document intelligence as primary use case, 256K context, 3B active params for fast inference, Mamba2 hybrid for superior long-context recall.
