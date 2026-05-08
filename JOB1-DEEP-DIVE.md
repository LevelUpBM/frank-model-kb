# Job 1 Deep Dive — Frank Lite Chat / RAG Inference

---

## What frank:v3 Actually Is

`frank:v3` = **Llama 3.1 8B** (base) with a custom Modelfile containing Frank's system prompt.  
It is NOT fine-tuned. It is a prompted Llama model.

The fine-tune (SFT on 3,149 pairs + 200 DPO pairs) was scoped but never run against the new corpus.  
So the current bar is: **stock Llama 3.1 8B + system prompt**.

---

## Frank's Actual Failure Modes (from diagnostics)

From the v1-v4 diagnostic runs (`docs/diagnostics/`):

| Failure | Root Cause | How often |
|---|---|---|
| **Wrong jurisdiction** | Retrieval returning WA/state law for federal queries | Frequent |
| **Hallucinated section numbers** | Model paraphrases from training memory when retrieval is weak | Common |
| **Marketing flyer drift** | Model fills gaps with generic HR advice instead of sourced content | Occasional |
| **Case name confusion** | Retrieves Barker v PMD (electrician) when user means CBA v Barker (trust law) | Confirmed in v1 |
| **Vague answers when specific needed** | Model too conservative, hedges instead of citing exact clause | Common |
| **Legislation missing from final context** | Reranker scores decisions over statutes for legal queries | Systematic |

**The core problem is not the model — it's the retrieval pipeline.**  
Granite 4.1 8B on the same retrieval stack will still get statute-less context for "small business dismissal" until the reranker quota fix is properly deployed.

**BUT** — on top of retrieval quality, model quality matters for:
1. Staying grounded in retrieved text (not hallucinating)
2. Following Frank's strict output rules (cite your source, admit when unknown)
3. Extracting the right clause from a long, messy EBA
4. Recognising when context is insufficient vs when it's adequate

---

## Head-to-Head: Llama 3.1 8B vs Candidates

### Benchmark comparison

| Benchmark | Llama 3.1 8B (frank:v3) | Granite 4.1 8B | Qwen 2.5 14B | Mistral Nemo 12B |
|---|---|---|---|---|
| MMLU | 73.0 | **73.84** | 79.0 | 68.0 |
| MT-Bench | 8.1 | **8.61** | 8.7 | 8.0 |
| IFEval | ~80 | **87.06** | ~84 | ~81 |
| ArenaHard | ~60 | **68.98** | ~72 | ~62 |
| GPQA | ~38 | **41.96** | ~42 | ~36 |
| SALAD-Bench (safety) | ~88 | **95.80** | ~90 | ~89 |
| HumanEval | ~72 | 85.37 | ~79 | ~73 |
| BFCL v3 (tool calling) | ~52 | **68.27** | ~66 | ~58 |
| Context window | 128K | Long (exact TBC) | 128K | 128K |
| VRAM (Q4) | ~5 GB | **~5 GB** | ~9 GB | ~7 GB |
| Licence | Meta LLAMA 3 | **Apache 2.0** | Apache 2.0 | Apache 2.0 |

**Key takeaway:** Granite 4.1 8B beats Llama 3.1 8B on every Frank-relevant benchmark — instruction following (+7%), safety (+7%), tool calling (+16%), reasoning (+4%) — at the same 5GB VRAM cost.

---

## Why Each Benchmark Matters for Frank

### IFEval — Instruction Following (87.06% vs ~80%)
Frank's system prompt has 25+ rules: "RULE 1: Always cite the source clause", "RULE 6: Ask one focused question", "Never fabricate a section number." IFEval measures exactly this — does the model follow explicit instructions. Granite's +7% here is the most important difference.

### ArenaHard — Real-world preference (68.98 vs ~60)
Human evaluators prefer Granite's responses in head-to-head comparisons. For Frank, this translates to answer quality and structure that users find useful, not just technically correct.

### SALAD-Bench — Safety (95.80% vs ~88%)
Frank handles sensitive HR content — misconduct investigations, terminations, mental health disclosures. High safety scores mean the model won't generate inappropriate content when processing difficult case facts.

### BFCL v3 — Tool/Function Calling (68.27% vs ~52%)
Matters now for Frank's structured output (JSON citations, VAULT_SAVE tags) and critically important once we build Frank agents (legal research, document comparison tools).

### GPQA — Graduate-level reasoning (41.96% vs ~38%)
Complex multi-clause EBA analysis, multi-step legal reasoning, understanding when provisions interact. Higher GPQA = better at hard questions.

---

## Frank-Specific Considerations

### 1. Context Window
- **Llama 3.1 8B:** 128K — adequate for current Frank context (8K assembled context + system prompt ~2K)
- **Granite 4.1 8B:** "Long context" — IBM hasn't published exact number but Granite 4.0 was 128K; 4.1 is at minimum equal
- **Not a differentiator at current usage** — Frank's assembled context is ~32K chars (~8K tokens), well within both

### 2. RAG-Explicit Design
IBM explicitly lists RAG as a primary capability for Granite 4.1. This means the training data and post-training alignment specifically addressed retrieval-augmented tasks — staying grounded in provided context, citing sources, handling conflicting information. Llama 3.1 8B is a general-purpose model that was never explicitly optimised for RAG.

### 3. Australian Legal Text
Neither model has been trained on Australian IR law specifically. This is why fine-tuning matters — the base model choice affects HOW WELL the fine-tune sticks. Better instruction following (Granite) = fine-tune has better foundation to work from.

### 4. System Prompt Adherence
Frank's Modelfile system prompt is ~2,000 tokens with very specific behavioural rules. IFEval +7% advantage compounds here — Granite will more reliably follow "always include a source citation" and "never give legal advice" than Llama.

### 5. Safety for HR Content
Frank regularly processes:
- Termination procedures
- Misconduct allegations  
- Sexual harassment complaints
- Mental health accommodations

SALAD-Bench 95.80% vs ~88% matters here. Llama 3.1 8B has known cases of generating inappropriate content in sensitive contexts. Granite's safety training is more robust.

---

## The Other Candidates

### Qwen 2.5 14B — Strong but 9GB VRAM
- MT-Bench 8.7 — marginally better than Granite 4.1 8B
- Structured output is exceptional (JSON extraction)
- **Problem:** 9GB VRAM leaves only 5GB for ChromaDB — tight
- Better as a background processor (Job 2) than Frank chat model
- **Verdict:** Use for Job 2, not Job 1

### Mistral Nemo 12B — Good but not enough gain
- 7GB VRAM — doable but eats into ChromaDB headroom
- Good on long context, multilingual
- MT-Bench 8.0 — worse than Granite 4.1 8B
- No explicit RAG optimisation
- **Verdict:** Skip — Granite is better and smaller

### DeepSeek R1 14B — Reasoning model, wrong use case
- Reasoning models (chain-of-thought) are designed for complex multi-step problems
- Frank queries rarely need 30-second CoT reasoning — they need fast grounded retrieval
- R1's "thinking" tokens add latency without clear benefit for RAG tasks
- **Verdict:** Interesting for complex legal analysis queries, not general Frank use

### Gemma 3 12B — Technically strong, licence concern
- 128K context, vision capability
- Google's Gemma Terms of Service has restrictions vs Apache 2.0
- 7GB VRAM
- **Verdict:** Monitor but Apache 2.0 alternatives (Granite) preferred for enterprise

---

## Fine-Tuning Path

Once we generate the 5,000 SFT pairs from the new corpus, we need to fine-tune. The model choice affects:

| Factor | Llama 3.1 8B | Granite 4.1 8B |
|---|---|---|
| Fine-tune cost (RunPod A40) | ~$8-12 | ~$8-12 (same size) |
| Base instruction following | ~80% IFEval | 87% IFEval |
| Alignment post fine-tune (est.) | Good | Better (stronger base) |
| LoRA adapter compatibility | ✅ | ✅ |
| GGUF export | ✅ | ✅ |
| Ollama deployment | ✅ | ✅ |
| Apache 2.0 (commercial use of fine-tune) | ❌ Meta LLAMA 3 licence | ✅ |

**Licence is critical:** Meta's LLAMA 3 licence has commercial use restrictions above certain user thresholds. Apache 2.0 (Granite) has no restrictions — fine-tune and deploy commercially without limitation.

---

## Recommendation

### Replace frank:v3 with Granite 4.1 8B in two steps:

**Step 1 — Immediate swap (this week, zero risk)**
```bash
# On frank-gpu
ollama pull granite4.1:8b

# Create new Modelfile with Frank's system prompt
# Copy frank:v3 Modelfile, change FROM to granite4.1:8b
ollama create frank:granite -f Modelfile-granite

# Run gold benchmark comparison
python3 /opt/frank-rag/tools/benchmark_runner.py --model frank:granite --baseline frank:v3
```

If Granite wins (expected) → swap. If Llama wins on any dimension → investigate before switching.

**Step 2 — Fine-tune on new corpus (after benchmark)**
- Generate 5,000 SFT pairs using the new pipeline
- Fine-tune Granite 4.1 8B on RunPod A40 (~$8-12)
- This produces `frank:granite-ft` — the first Frank model actually trained on Australian IR law
- Export GGUF Q4_K_M → deploy to Ollama

**Expected outcome:**
- Instruction following: +7% (IFEval advantage)
- Hallucination reduction: measurable (safety +8%, explicit RAG training)
- Citation accuracy: improved (tool calling +16%)
- Commercial licence: clean (Apache 2.0 vs Meta LLAMA 3)
- VRAM: identical (~5GB Q4)
- Cost: $0 model cost, ~$8-12 one-time fine-tune

---

## After GPU Upgrade — NVIDIA Nemotron-3 Nano Omni 30B NVFP4

Once frank-gpu is upgraded to ~24GB:

| Factor | Granite 4.1 8B | Nemotron-3 Nano 30B NVFP4 |
|---|---|---|
| Total params | 8B | 31B |
| Active params | 8B | 3B (MoE) |
| VRAM | ~5 GB | ~21 GB |
| Context | Long | 256K |
| Architecture | Dense transformer | Mamba2-Transformer Hybrid MoE |
| Long-context recall | Good | **Better** (Mamba2 SSM) |
| Document intelligence | Listed as capability | **Primary use case** |
| OCR/table parsing | No | Yes |
| Inference speed | Fast | Fast (3B active) |
| Cost | Free (local) | Free (local) |

**Nemotron's Mamba2 architecture** is specifically designed for long-context recall without attention bottlenecks. For Frank's use case — retrieving from 750K+ chunks across 6 collections, assembling 32K context windows — Mamba2 hybrid should outperform pure transformer architectures at the same inference cost.

The upgrade path: Granite 4.1 8B now → prove quality → GPU upgrade → Nemotron NVFP4 as the production Frank Lite model.
