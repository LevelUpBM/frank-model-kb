# Mistral Medium 3.5 128B

**Source:** https://huggingface.co/mistralai/Mistral-Medium-3.5-128B  
**Type:** Dense  
**License:** Modified MIT (commercial + non-commercial; exceptions for large-revenue companies)  
**Developer:** Mistral AI  
**Blog:** https://mistral.ai/news/vibe-remote-agents-mistral-medium-3-5

---

## Specs

| Field | Value |
|---|---|
| Parameters | 128B (dense) |
| Context Window | 256K tokens |
| Modalities | Text + Image input → Text output |
| Reasoning | Configurable per request (none / high) |
| Function Calling | Native |
| JSON Output | Native |
| Vision | ✅ Variable image sizes & aspect ratios |
| Languages | English, French, Spanish, German, Italian, Portuguese, Dutch, Chinese, Japanese, Korean, Arabic + more |

---

## Key Features

- **Unified model:** Replaces Mistral Medium 3.1 + Magistral + Devstral 2 in a single set of weights
- **Configurable reasoning effort:** `reasoning_effort="none"` for fast chat, `reasoning_effort="high"` for complex agentic tasks
- **Powers Le Chat** (Mistral's flagship product)
- **Replaces Devstral 2** in Vibe coding agent
- **EAGLE speculative decoding** available: https://huggingface.co/mistralai/Mistral-Medium-3.5-128B-EAGLE

---

## Benchmarks

### Agentic
| Benchmark | Score |
|---|---|
| SWE-Bench Verified | **77.6%** |
| τ³-Telecom | **91.4%** |

Supersedes all previous Mistral coding models (Devstral, Magistral) across all benchmarks.

Full instruction following, reasoning, and coding benchmark charts at source URL.

---

## Recommended Settings

| Use Case | reasoning_effort | Temperature | Top-p |
|---|---|---|---|
| Fast chat / simple tasks | `none` | 0.0–0.7 | 1.0 |
| Complex reasoning / agentic | `high` | 0.7 | 0.95 |

---

## VRAM Requirements

| Format | VRAM |
|---|---|
| BF16 full | ~256 GB |
| Q4_K_M (GGUF) | ~70 GB |
| Multi-GPU min | 4× A100 80GB |

---

## Deployment

### Ollama
```bash
ollama pull mistral-medium-3.5
```

### vLLM (recommended)
```bash
vllm serve mistralai/Mistral-Medium-3.5-128B \
  --reasoning-parser mistral \
  --enable-auto-tool-choice \
  --tool-call-parser mistral
```

### vLLM + EAGLE speculative decoding
```bash
vllm serve mistralai/Mistral-Medium-3.5-128B \
  --speculative-model mistralai/Mistral-Medium-3.5-128B-EAGLE \
  --num-speculative-tokens 5
```

### SGLang
See HuggingFace page for SGLang config.

### GGUF (llama.cpp / LM Studio)
Unsloth GGUFs: https://huggingface.co/unsloth/Mistral-Medium-3.5-128B-GGUF

⚠️ **Config fix required:** GGUFs generated before commit `c4be198` have long-context degradation bug. Use updated config.

---

## Mistral Vibe Integration (local vLLM)
```toml
# ~/.vibe/config.toml
[[models]]
name = "mistralai/Mistral-Medium-3.5-128B"
provider = "vllm"
alias = "mistral-medium-3.5"
thinking = "high"
temperature = 0.7
auto_compact_threshold = 168000
```

---

## Links

| Resource | URL |
|---|---|
| HuggingFace | https://huggingface.co/mistralai/Mistral-Medium-3.5-128B |
| EAGLE model | https://huggingface.co/mistralai/Mistral-Medium-3.5-128B-EAGLE |
| GGUF (Unsloth) | https://huggingface.co/unsloth/Mistral-Medium-3.5-128B-GGUF |
| Ollama | https://ollama.com/library/mistral-medium-3.5 |
| Blog | https://mistral.ai/news/vibe-remote-agents-mistral-medium-3-5 |
| Mistral Vibe | https://github.com/mistralai/mistral-vibe |

---

## Frank Evaluation Notes

- **Fit for Frank Lite:** ❌ 128B dense — needs 4× A100 80GB minimum. Not a single-GPU option.
- **API use:** ✅ Available via Mistral API and OpenRouter — strong candidate for Frank hosted/cloud tier
- **SWE-bench 77.6%** — exceptional coding/agentic performance
- **256K context** — covers full EBA documents + system prompt comfortably
- **Configurable reasoning** is ideal for Frank: fast mode for simple queries, high-effort for complex legal analysis
- **Verdict:** Too large for Frank Lite local deployment. Strong candidate for Frank Cloud / enterprise tier via API. Monitor for smaller distilled variants.
