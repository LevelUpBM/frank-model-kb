# AEON-7 Qwen3.6-27B AEON Ultimate Uncensored

**Source:** https://huggingface.co/AEON-7/Qwen3.6-27B-AEON-Ultimate-Uncensored-BF16  
**Type:** Dense (GatedDeltaNet hybrid attention)  
**Base:** Qwen3.6-27B  
**License:** Check HuggingFace — abliterated/uncensored fine-tune  
**Developer:** AEON-7  
**Ops repo:** https://github.com/AEON-7/Qwen3.6-27B-AEON-Ultimate-Uncensored-DFlash

---

## Key Facts

- Abliterated (uncensored) fine-tune of Qwen3.6-27B
- MTP head grafted from Qwen/Qwen3.6-27B base — speculative decoding enabled
- GatedDeltaNet linear attention hybrid architecture (SSM + standard attention)
- ⚠️ **Compliance note:** "Ultimate Uncensored" — validate before enterprise use

---

## Variants

| Variant | Disk | VRAM | Spec Decode | Target Hardware |
|---|---|---|---|---|
| **BF16 (this repo)** | 52 GB | 80 GB | qwen3_5_mtp n=3 | A100/H100 80GB |
| NVFP4 | 26 GB | ~26 GB | DFlash k=15 | DGX Spark (GB10) |
| Multimodal-NVFP4-MTP | 27 GB | ~27 GB | qwen3_5_mtp n=3 | RTX PRO 6000 Blackwell |
| Text-NVFP4-MTP | 26 GB | ~26 GB | qwen3_5_mtp n=3 | RTX PRO 6000 (text-only) |
| Multimodal-NVFP4-MTP-XS | 21 GB | ~21 GB | qwen3_5_mtp n=3 | RTX 5090 (32 GB) |
| **Text-NVFP4-MTP-XS** | 20 GB | ~20 GB | qwen3_5_mtp n=3 | RTX 5090 / 24 GB cards |

---

## Hardware Routing (measured)

| Hardware | Use | Why |
|---|---|---|
| DGX Spark / GB10 (unified memory) | NVFP4-DFlash | DFlash beats MTP +26% median, +52% peak on Spark |
| RTX PRO 6000 / RTX 5090 / B100 (dedicated VRAM) | NVFP4-MTP or XS | MTP wins on dedicated VRAM — 111.4 tok/s, 69% acceptance |
| A100 / H100 (no native FP4) | BF16 (this repo) | NVFP4 dequants to BF16 on Ampere/Hopper anyway |

---

## MTP Performance (DGX Spark)

- Mean accepted length: **3.3/3**
- P0 acceptance: **~90%**
- Avg draft acceptance: **~78%**
- Confirms abliteration doesn't damage MTP top-K distribution

---

## Architecture Notes

- **model_type:** qwen3_5
- **GatedDeltaNet linear attention:** SSM recurrence-critical conv1d preserved BF16 in XS variants
- **XS quantization strategy:** Quantize heavy projection matmuls to NVFP4 (~11 GB saved), keep conv1d at BF16 to prevent long-context recurrence drift
- **Multimodal:** Vision tower preserved in Multimodal variants

---

## Deployment

### vLLM with speculative decoding
```bash
vllm serve AEON-7/Qwen3.6-27B-AEON-Ultimate-Uncensored-BF16 \
  --speculative-config '{"method":"qwen3_5_mtp","num_speculative_tokens":3}'
```

### SGLang (DGX Spark NVFP4 DFlash)
See production docker-compose in GitHub ops repo.

---

## All Variant Links

| Variant | HuggingFace |
|---|---|
| BF16 | https://huggingface.co/AEON-7/Qwen3.6-27B-AEON-Ultimate-Uncensored-BF16 |
| NVFP4 | https://huggingface.co/AEON-7/Qwen3.6-27B-AEON-Ultimate-Uncensored-NVFP4 |
| Multimodal-NVFP4-MTP | https://huggingface.co/AEON-7/Qwen3.6-27B-AEON-Ultimate-Uncensored-Multimodal-NVFP4-MTP |
| Text-NVFP4-MTP | https://huggingface.co/AEON-7/Qwen3.6-27B-AEON-Ultimate-Uncensored-Text-NVFP4-MTP |
| Multimodal-NVFP4-MTP-XS | https://huggingface.co/AEON-7/Qwen3.6-27B-AEON-Ultimate-Uncensored-Multimodal-NVFP4-MTP-XS |
| Text-NVFP4-MTP-XS | https://huggingface.co/AEON-7/Qwen3.6-27B-AEON-Ultimate-Uncensored-Text-NVFP4-MTP-XS |

---

## Frank Evaluation Notes

- **Fit for Frank Lite:** ⚠️ Text-NVFP4-MTP-XS at 20GB is close — just over 14GB frank-gpu limit. Would fit on A40 8GB plan.
- **Compliance risk:** "Uncensored" — not suitable for enterprise Frank deployments without validation. Risk of generating inappropriate content without safety guardrails.
- **Technical quality:** Strong — 27B dense with hybrid SSM+attention, native MTP, measured benchmarks
- **Verdict:** Technically interesting, compliance risk too high for enterprise Frank. Monitor base Qwen3.6-27B instead.
