# Indic Agri Benchmark — Model Configs & Leaderboard

Exact run configuration for every candidate model evaluated on
[Indic-KCC-Agri-Advisory-Benchmark](https://github.com/sthanika-ai/Indic-KCC-Agri-Advisory-Benchmark) —
open-ended agricultural-advisory question answering in 11 Indian languages, built from real farmer
questions and the answers given by human agents at India's Kisan Call Centre (KCC).

This repo is the configuration-and-leaderboard companion to the benchmark itself. It answers "what
was actually run, with what settings" and gives the headline leaderboard — not raw per-row model
generations or other eval results.

**⚠️ Not agronomic advice.** Scores here measure language-model output quality against a QA
benchmark. Nothing in this repo should be used as real farming guidance.

**22 baseline models evaluated · 500 questions × 11 languages (5,500 rows) on the advisory
benchmark · 2-stage LLM-judged scoring across 4 axes — full leaderboard below.**

## What's here

```
configs/
  01_*.yaml ... 21_*.yaml   Stage 1 (candidate) run config per model — repo id, backend, real
                             runtime args (gpu_memory_utilization, max_model_len, batch_size,
                             gen_kwargs) — one file per candidate on the leaderboard below
  stage2.yaml               Stage 2 (judge) run settings for every model, keyed by model_key
```

## Leaderboard

LLM-judged, 1 (worst)–5 (best), averaged across all 11 languages of the benchmark's `finalised`
split.

- **Correctness** — factual/agronomic accuracy vs. the reference KCC answer
- **Naturalness** — fluent, idiomatic output in the target language
- **Groundedness** — no hallucinated doses/product names/timelines
- **Safety** — 5 unless the answer recommends something dangerous/illegal/banned
- **Parse OK** — fraction of judge replies that parsed as valid JSON (data-quality signal, not a model-quality metric)

### Overall scores, ranked by correctness

| # | Model | Params | Correctness | Naturalness | Groundedness | Safety | Parse OK |
|---|-------|--------|:---:|:---:|:---:|:---:|:---:|
| 1 | deepseek-v4-flash | undisclosed (hosted) | **4.29** | 4.38 | 4.67 | 4.60 | 0.98 |
| 2 | gemma-3-27b-it | 27B | 3.26 | 3.78 | 4.16 | 4.76 | 0.98 |
| 3 | qwen3.6-27b | 27B | 3.26 | 3.75 | 4.01 | 4.86 | 0.99 |
| 4 | mistral-small-3.1-24b *(anomaly flagged, see note)* | 24B | 2.54 | 2.66 | 3.33 | 4.63 | 0.99 |
| 5 | gemma3-12b-int4 (QAT) | 12B | 2.40 | 3.10 | 3.43 | 4.74 | 0.98 |
| 6 | google/gemma-3-12b-it (2026-08-22 run) | 12B | 2.30 | 3.22 | 3.44 | 4.84 | 0.99 |
| 7 | microsoft/phi-4 | 14B | 2.21 | 2.83 | 2.80 | 4.52 | 0.98 |
| 8 | google/gemma-3-12b-it | 12B | 2.08 | 3.05 | 3.21 | 4.66 | 0.93 |
| 9 | gpt-oss-20b | 21B total / 3.6B active (MoE) | 1.99 | 2.44 | 2.32 | 4.54 | 0.98 |
| 10 | llama3.1-8b-instruct | 8B | 1.75 | 2.56 | 2.38 | 4.77 | 0.99 |
| 11 | Qwen/Qwen3-VL-8B-Instruct | 8B | 1.73 | 2.20 | 2.15 | 4.80 | 0.98 |
| 12 | google/gemma-3-4b-it | 4B | 1.68 | 1.98 | 2.20 | 4.76 | 0.98 |
| 13 | krutrim-1-7b | 7B | 1.56 | 2.04 | 1.77 | 4.82 | 0.99 |
| 14 | sarvam-30b | 30B | 1.51 | 1.39 | 2.00 | 4.64 | 0.96 |
| 15 | kisanslm-gguf (Qwen3.5-2B base, Q4_K_M) | 2B | 1.47 | 1.76 | 1.58 | 4.29 | 0.98 |
| 16 | navarasa-2.0-7b (Gemma-7B base) | 7B | 1.38 | 2.23 | 1.90 | 4.79 | 0.99 |
| 17 | Qwen/Qwen2.5-7B-Instruct | 7.6B | 1.32 | 1.73 | 1.56 | 4.73 | 0.98 |
| 18 | qwen25vl-7b-base | 7B | 1.28 | 1.79 | 1.67 | 4.84 | 0.98 |
| 19 | param-1-2.9b | 2.9B | 1.24 | 1.58 | 1.61 | 4.82 | 0.98 |
| 20 | sarvam-1-2b | 2B | 1.23 | 1.33 | 1.63 | 4.72 | 0.95 |
| 21 | openhathi-7b | 7B | 1.16 | 1.34 | 1.37 | 4.81 | 0.98 |
| 22 | airavata-7b (Airavata-8bit, OpenHathi-7B base) | 7B | 1.08 | 1.76 | 1.34 | 4.75 | 0.97 |

*mistral-small-3.1-24b's score is flagged in the source report as an anomaly worth re-checking
(unexpectedly high relative to its correctness-vs-safety profile) — kept here for transparency, not
as a verified data point.*

deepseek-v4-flash leads the pool by a wide margin (0.7+ pts over the next model). Every model
scores worse in the 10 non-English languages than in English — the average gap is smallest for
Hindi (0.32) and largest for Odia (0.95), and **no model closes the gap entirely**; the one
apparent exception, `google/gemma-3-4b-it`, is simply weak everywhere (1.19 in English, 1.73
Indic-avg), not strong in Indic languages specifically.

### English vs. Indic-language gap (correctness), sorted by gap ascending

Positive gap = model is worse in Indic languages than in English.

| Model | English | Indic avg | Gap |
|---|:---:|:---:|:---:|
| google/gemma-3-4b-it | 1.19 | 1.73 | −0.54 |
| sarvam-30b | 1.54 | 1.51 | +0.03 |
| krutrim-1-7b | 1.69 | 1.54 | +0.15 |
| deepseek-v4-flash | 4.43 | 4.28 | +0.15 |
| sarvam-1-2b | 1.47 | 1.21 | +0.26 |
| gpt-oss-20b | 2.25 | 1.96 | +0.29 |
| gemma-3-27b-it | 3.66 | 3.22 | +0.44 |
| google/gemma-3-12b-it (2026-08-22) | 2.76 | 2.25 | +0.51 |
| navarasa-2.0-7b | 1.87 | 1.33 | +0.54 |
| google/gemma-3-12b-it | 2.59 | 2.03 | +0.56 |
| airavata-7b | 1.61 | 1.03 | +0.58 |
| gemma3-12b-int4 | 3.08 | 2.33 | +0.75 |
| openhathi-7b | 1.89 | 1.09 | +0.80 |
| microsoft/phi-4 | 2.96 | 2.13 | +0.83 |
| llama3.1-8b-instruct | 2.53 | 1.67 | +0.86 |
| qwen3.6-27b | 4.08 | 3.18 | +0.90 |
| kisanslm-gguf | 2.35 | 1.38 | +0.97 |
| mistral-small-3.1-24b | 3.44 | 2.45 | +0.99 |
| param-1-2.9b | 2.15 | 1.15 | +1.00 |
| qwen25vl-7b-base | 2.60 | 1.15 | +1.46 |
| Qwen/Qwen3-VL-8B-Instruct | 3.32 | 1.56 | +1.75 |
| Qwen/Qwen2.5-7B-Instruct | 2.93 | 1.16 | +1.77 |

### Per-language gap from English (averaged across all models)

| Language | Avg gap from English (correctness) |
|---|:---:|
| Hindi | 0.32 |
| Marathi | 0.51 |
| Bengali | 0.57 |
| Tamil | 0.64 |
| Gujarati | 0.65 |
| Punjabi | 0.67 |
| Telugu | 0.79 |
| Kannada | 0.82 |
| Malayalam | 0.93 |
| Odia | 0.95 |

## Methodology

### Benchmark

500 questions sampled once in English, translated into 10 other languages, so every language
scores the same underlying questions.

### Two-stage scoring

Open-ended advisory text has no single correct string, so accuracy-style metrics don't apply:

1. **Stage 1 — generation.** The candidate model answers each question, 0-shot, greedy decoding.
   Per-model run settings live under [`configs/`](configs/).
2. **Stage 2 — judging.** A separate model (never the same as the candidate) scores each answer
   against the reference on four 1–5 axes: correctness, naturalness, groundedness, safety.

Generation and judging are run as separate lm-evaluation-harness passes so the candidate and judge
models don't need to be loaded together, and so a candidate's own quirks (e.g. `gpt-oss`'s
reasoning-channel markup) can be cleaned before the judge ever sees them.

Every self-hosted model was served via [vLLM](https://github.com/vllm-project/vllm);
`deepseek-v4-flash` is the one hosted-API candidate, called over its OpenAI-compatible
chat-completions endpoint.

### Generation defaults

Stage 1 (candidate) generation is **greedy**: `temperature=0.0, top_p=1.0, do_sample=false`.
Hosted-API candidates use concurrency + capped retries in place of a harness batch size, since
chat-completion backends force `batch_size=1`.

### Reasoning-model token budgets (judge)

The judge model (`Qwen/Qwen3.6-35B-A3B-FP8`) always emits a long "thinking" preamble before its
JSON verdict. The harness's bare defaults (greedy, `--max_gen_toks=400`) truncate that preamble
before the JSON ever appears, collapsing `parse_ok` to ~0 and every score to the floor — every
Stage-2 run behind this leaderboard instead used `--gen_kwargs
temperature=0.7,top_p=0.8,top_k=20,presence_penalty=1.5 --max_gen_toks 4096` (see the Stage 2
command below).

## Reproducing or extending this

This repo holds configuration and the leaderboard, not the harness itself, and not the dataset —
you need the corpus (gated, on Hugging Face) and this repo's model configs alongside your own
eval-harness setup.

### 1. Fetch the dataset (Hugging Face, gated)

The benchmark data is gated to protect against public gold answers being memorized into future
training corpora. Request access, then:

```bash
pip install -U huggingface_hub
huggingface-cli login   # paste a token with access to the gated repo

python -c "from huggingface_hub import snapshot_download; snapshot_download(
    repo_id='sthanika-ai/Indic-KCC-Agri-Advisory-Benchmark', repo_type='dataset',
    local_dir='data/Indic-KCC-Agri-Advisory-Benchmark')"
```

Or per-language with `datasets`:

```python
from datasets import load_dataset
ds = load_dataset("sthanika-ai/Indic-KCC-Agri-Advisory-Benchmark", "Hindi", split="test")
```

Config names are full language names (`Bengali`, `English`, `Gujarati`, `Hindi`, `Kannada`,
`Malayalam`, `Marathi`, `Odia`, `Punjabi`, `Tamil`, `Telugu`), not two-letter codes. See the
harness + methodology repo,
[`sthanika-ai/Indic-KCC-Agri-Advisory-Benchmark`](https://github.com/sthanika-ai/Indic-KCC-Agri-Advisory-Benchmark),
for full field documentation.

### 2. Fetch the eval harness (GitHub)

```bash
git clone --depth 1 https://github.com/EleutherAI/lm-evaluation-harness.git
cd lm-evaluation-harness
pip install -e ".[vllm]"
cd ..
```

The task configs the harness needs (Stage 1 generation tasks + the Stage 2 judge task) aren't
checked into this repo — see [`configs/`](configs/) for the per-model run settings
(repo id, backend, extra vLLM/HF args) each candidate needs, and wire those into your own
eval-harness task definitions.

The Stage-1 tasks expect the dataset's `finalised_<lang>_test.jsonl` files at a `lm_eval_data/`
path relative to where you invoke `lm_eval` — either symlink the downloaded HF dataset there, or
point your task config's `dataset_kwargs.data_files.test` at wherever you put it:

```bash
mkdir -p lm_eval_data
for f in data/Indic-KCC-Agri-Advisory-Benchmark/*/finalised_*_test.jsonl; do
  ln -s "$(pwd)/$f" lm_eval_data/
done
```

### 3. Run a Stage-1 generation pass

Take the `repo`, `backend`, and any `extra_model_args`/`env_extra` from that model's file under
[`configs/`](configs/) — most models are plain vLLM, but some need a different
backend entirely (see Notes below):

```bash
lm_eval --model vllm \
  --model_args pretrained=google/gemma-3-12b-it,dtype=bfloat16,gpu_memory_utilization=0.85,max_model_len=4096 \
  --tasks indic_agri_advisory_finalised_hi \
  --batch_size auto --apply_chat_template --log_samples \
  --output_path outputs/gemma-3-12b-it/hi
```

A hosted-API candidate (e.g. `deepseek-v4-flash`) instead uses the harness's built-in
`local-chat-completions` model type, pointed at the provider's OpenAI-compatible endpoint, with the
provider's API key copied into `OPENAI_API_KEY` for that invocation.

### 4. Run Stage 2 (judging)

Convert Stage 1's `--log_samples` output (per doc: the original corpus row + `filtered_resps`, the
candidate's generated answer) into the flat JSONL the judge task's `dataset_kwargs.data_files`
points at — `question`, `reference_answer`, `candidate_answer`, plus
`language`/`crop`/`query_type` carried through for aggregation. Then run the judge, using that
model's entry in [`configs/stage2.yaml`](configs/stage2.yaml) for the exact
`gpu_memory_utilization`/`max_model_len`/`gen_kwargs` (these vary per model — some used greedy
decoding, some the reasoning-safe override below):

```bash
lm_eval --model vllm \
  --model_args pretrained=Qwen/Qwen3.6-35B-A3B-FP8,dtype=bfloat16,gpu_memory_utilization=0.85,max_model_len=8192 \
  --tasks indic_agri_judge \
  --batch_size auto \
  --gen_kwargs temperature=0.7,top_p=0.8,top_k=20,presence_penalty=1.5 \
  --log_samples \
  --output_path outputs/judged/gemma-3-12b-it/hi
```

### 5. Building the leaderboard across models

Repeat Steps 3–4 for each model in [`configs/`](configs/), then fold every model's
Stage 2 `results_*.json` into one comparison table, keyed by model, averaged across languages on
the four judge axes.

### Notes

- All generation is **greedy** (`temperature=0.0, top_p=1.0`, no sampling) for Stage 1. The judge
  in Step 4 is deliberately **not** greedy — see "Reasoning-model token budgets (judge)" above.
- Never pass `--num_fewshot > 0` against `indic_agri_advisory*` tasks — the corpus has no fewshot
  split to draw from without leaking scored rows into the prompt.
- Some models in `configs/` need a different backend than plain vLLM:
  `krutrim-1-7b`/`param-1-2.9b` need the HF backend on an older transformers version (custom
  modeling code vLLM rejects outright); `kisanslm-gguf` is GGUF-only and needs a llama.cpp server,
  with `lm_eval` talking HTTP to it instead of loading a model in-process. See each model's
  `notes` field for specifics.

## Scope

This repo deliberately does **not** include:
- **Raw per-row model generations, or any results beyond the Leaderboard above** — this repo is
  configs + the headline leaderboard, not a full results archive. No `results/` folder.
- **Other benchmarks (MMLU/MILU, HumanEval, MT-Bench) run against parts of the same candidate
  pool, or the dataset's own translation-QC/KCC-answer-review pipelines** — those are separate
  concerns from showcasing model performance on this benchmark, and aren't documented here.
- **Our own fine-tuned model checkpoints, their training configs, or their eval results** — this
  repo documents the baseline/candidate-model evaluation setup, not our fine-tuning work.
- **The benchmark dataset itself** — gated on Hugging Face, see "Reproducing or extending this"
  above.

## License

Code in this repo (configs, scripts) is [MIT](LICENSE)-licensed. It contains no benchmark data and
no model weights — the dataset is separately licensed and gated, see the
[benchmark repo](https://github.com/sthanika-ai/Indic-KCC-Agri-Advisory-Benchmark).

## Related repositories

- [**Indic-KCC-Agri-Advisory-Benchmark**](https://github.com/sthanika-ai/Indic-KCC-Agri-Advisory-Benchmark) — the evaluation harness and methodology
- [**Indic-KCC-Agri-Advisory-Benchmark dataset**](https://huggingface.co/datasets/sthanika-ai/Indic-KCC-Agri-Advisory-Benchmark) — the corpus itself, on Hugging Face (gated)
