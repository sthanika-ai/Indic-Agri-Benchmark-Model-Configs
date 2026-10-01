# Indic Agri Benchmark: Model Configs & Leaderboard

Exact run configuration and the full leaderboard for every candidate model evaluated on Indic-KCC-Agri-Advisory-Benchmark, open-ended agricultural-advisory QA in 11 Indian languages.

[![License: MIT](https://img.shields.io/badge/license-MIT-56BF4F?style=flat-square&labelColor=1E281F)](LICENSE)
[![Hugging Face](https://img.shields.io/badge/%F0%9F%A4%97%20dataset-gated-FFD21E?style=flat-square&labelColor=1E281F)](https://huggingface.co/datasets/sthanika-ai/Indic-KCC-Agri-Advisory-Benchmark)
[![Benchmark](https://img.shields.io/badge/harness-Indic--KCC--Agri-56BF4F?style=flat-square&labelColor=1E281F&logo=github&logoColor=white)](https://github.com/sthanika-ai/Indic-KCC-Agri-Advisory-Benchmark)
[![Report](https://img.shields.io/badge/report-sthanika.ai-56BF4F?style=flat-square&labelColor=1E281F&logo=firefox&logoColor=white)](https://sthanika.ai/research/indic-agri-advisory-2026)

> ⚠️ **Not agronomic advice.** Scores here measure language-model output quality against a QA benchmark. Nothing in this repo should be used as real farming guidance.

## What it measures

The benchmark has 500 real Kisan Call Centre (KCC) questions, sampled once in English and translated into 10 other languages, so every language scores the same underlying questions (5,500 rows). This repo is the configuration-and-leaderboard companion: it answers "what was actually run, with what settings" for 21 baseline models, and gives the headline leaderboard. It does not include raw per-row generations or other eval results.

Scoring is two-stage. Stage 1 has the candidate answer 0-shot with greedy decoding. Stage 2 has a separate judge model (never the candidate) score each answer against the reference on four 1–5 axes: correctness, naturalness, groundedness and safety. The two stages are separate `lm-evaluation-harness` passes so the candidate and judge never need to be loaded together. Report: [sthanika.ai](https://sthanika.ai/research/indic-agri-advisory-2026)

```
configs/
  01_*.yaml ... 21_*.yaml   Stage 1 (candidate) run config per model: repo id, backend, runtime
                             args (gpu_memory_utilization, max_model_len, batch_size, gen_kwargs)
  stage2.yaml               Stage 2 (judge) run settings for every model, keyed by model_key
```

## Quickstart

This repo holds configuration and the leaderboard, not the harness or the dataset. You need the gated corpus, an `lm-evaluation-harness` checkout and this repo's configs together.

```bash
# 1. Dataset (gated; request access first)
pip install -U huggingface_hub
huggingface-cli login
python -c "from huggingface_hub import snapshot_download; snapshot_download(
    repo_id='sthanika-ai/Indic-KCC-Agri-Advisory-Benchmark', repo_type='dataset',
    local_dir='data/Indic-KCC-Agri-Advisory-Benchmark')"

# 2. Eval harness
git clone --depth 1 https://github.com/EleutherAI/lm-evaluation-harness.git
cd lm-evaluation-harness && pip install -e ".[vllm]" && cd ..

# link the dataset where the Stage 1 tasks expect it
mkdir -p lm_eval_data
for f in data/Indic-KCC-Agri-Advisory-Benchmark/*/finalised_*_test.jsonl; do
  ln -s "$(pwd)/$f" lm_eval_data/
done

# 3. Stage 1: generation (take repo, backend and extra args from the model's file in configs/)
lm_eval --model vllm \
  --model_args pretrained=google/gemma-3-12b-it,dtype=bfloat16,gpu_memory_utilization=0.85,max_model_len=4096 \
  --tasks indic_agri_advisory_finalised_hi \
  --batch_size auto --apply_chat_template --log_samples \
  --output_path outputs/gemma-3-12b-it/hi

# 4. Stage 2: judging (after converting Stage 1 --log_samples output to the judge's input JSONL)
lm_eval --model vllm \
  --model_args pretrained=Qwen/Qwen3.6-35B-A3B-FP8,dtype=bfloat16,gpu_memory_utilization=0.85,max_model_len=8192 \
  --tasks indic_agri_judge \
  --batch_size auto \
  --gen_kwargs temperature=0.7,top_p=0.8,top_k=20,presence_penalty=1.5 \
  --log_samples \
  --output_path outputs/judged/gemma-3-12b-it/hi
```

Repeat steps 3–4 for each model in `configs/`, then fold every model's Stage 2 `results_*.json` into one table keyed by model and averaged across languages.

Notes:

- **Task configs are not checked in.** See `configs/` for per-model run settings and wire them into your own task definitions. Convert Stage 1 output to a flat JSONL with `question`, `reference_answer`, `candidate_answer`, plus `language`, `crop` and `query_type` carried through.
- **Per-language loading.** `load_dataset("sthanika-ai/Indic-KCC-Agri-Advisory-Benchmark", "Hindi", split="test")` works too. Config names are full language names, not two-letter codes.
- **Per-model Stage 2 settings** (`gpu_memory_utilization`, `max_model_len`, `gen_kwargs`) are in `configs/stage2.yaml`. Some models used greedy decoding, some the reasoning-safe override above.
- **Judge sampling is deliberate.** The judge (`Qwen/Qwen3.6-35B-A3B-FP8`) always emits a long "thinking" preamble. The harness default (greedy, `--max_gen_toks=400`) truncates it before the JSON appears, collapsing `parse_ok` to about 0 and every score to the floor. Every Stage 2 run here used the override with `--max_gen_toks 4096`.
- **Never pass `--num_fewshot > 0`** against `indic_agri_advisory*` tasks. The corpus has no fewshot split without leaking scored rows into the prompt.
- **Backends.** Every self-hosted model was served via vLLM, except `krutrim-1-7b` and `param-1-2.9b` (HF backend on an older `transformers`, since their custom modeling code vLLM rejects) and `kisanslm-gguf` (GGUF-only, so a llama.cpp server with `lm_eval` talking HTTP to it). `deepseek-v4-flash` is the one hosted API, via `local-chat-completions` with the provider's key in `OPENAI_API_KEY` for that invocation. Hosted APIs use concurrency and capped retries in place of a batch size.
- **Stage 1 decoding** is greedy: `temperature=0.0`, `top_p=1.0`, `do_sample=false`.

## Results

LLM-judged, 1 (worst) to 5 (best), averaged across all 11 languages of the finalised split. Parse OK is the share of judge replies that parsed as valid JSON, a data-quality signal and not a model-quality metric. Ranked by correctness:

| # | model | params | correctness | naturalness | groundedness | safety | parse OK |
|---|---|---|---|---|---|---|---|
| 1 | deepseek-v4-flash | undisclosed (hosted) | 4.29 | 4.38 | 4.67 | 4.60 | 0.98 |
| 2 | gemma-3-27b-it | 27B | 3.26 | 3.78 | 4.16 | 4.76 | 0.98 |
| 3 | qwen3.6-27b | 27B | 3.26 | 3.75 | 4.01 | 4.86 | 0.99 |
| 4 | mistral-small-3.1-24b (anomaly flagged) | 24B | 2.54 | 2.66 | 3.33 | 4.63 | 0.99 |
| 5 | gemma3-12b-int4 (QAT) | 12B | 2.40 | 3.10 | 3.43 | 4.74 | 0.98 |
| 6 | google/gemma-3-12b-it | 12B | 2.30 | 3.22 | 3.44 | 4.84 | 0.99 |
| 7 | microsoft/phi-4 | 14B | 2.21 | 2.83 | 2.80 | 4.52 | 0.98 |
| 8 | gpt-oss-20b | 21B total / 3.6B active (MoE) | 1.99 | 2.44 | 2.32 | 4.54 | 0.98 |
| 9 | llama3.1-8b-instruct | 8B | 1.75 | 2.56 | 2.38 | 4.77 | 0.99 |
| 10 | Qwen/Qwen3-VL-8B-Instruct | 8B | 1.73 | 2.20 | 2.15 | 4.80 | 0.98 |
| 11 | google/gemma-3-4b-it | 4B | 1.68 | 1.98 | 2.20 | 4.76 | 0.98 |
| 12 | krutrim-1-7b | 7B | 1.56 | 2.04 | 1.77 | 4.82 | 0.99 |
| 13 | sarvam-30b | 30B | 1.51 | 1.39 | 2.00 | 4.64 | 0.96 |
| 14 | kisanslm-gguf (Qwen3.5-2B base, Q4_K_M) | 2B | 1.47 | 1.76 | 1.58 | 4.29 | 0.98 |
| 15 | navarasa-2.0-7b (Gemma-7B base) | 7B | 1.38 | 2.23 | 1.90 | 4.79 | 0.99 |
| 16 | Qwen/Qwen2.5-7B-Instruct | 7.6B | 1.32 | 1.73 | 1.56 | 4.73 | 0.98 |
| 17 | qwen25vl-7b-base | 7B | 1.28 | 1.79 | 1.67 | 4.84 | 0.98 |
| 18 | param-1-2.9b | 2.9B | 1.24 | 1.58 | 1.61 | 4.82 | 0.98 |
| 19 | sarvam-1-2b | 2B | 1.23 | 1.33 | 1.63 | 4.72 | 0.95 |
| 20 | openhathi-7b | 7B | 1.16 | 1.34 | 1.37 | 4.81 | 0.98 |
| 21 | airavata-7b (Airavata-8bit, OpenHathi-7B base) | 7B | 1.08 | 1.76 | 1.34 | 4.75 | 0.97 |

mistral-small-3.1-24b is flagged in the source report as an anomaly worth re-checking (unexpectedly high relative to its correctness-vs-safety profile). It is kept for transparency, not as a verified data point.

deepseek-v4-flash leads by a wide margin (0.7+ points over the next model). Every model scores worse in the 10 non-English languages than in English. The one apparent exception, gemma-3-4b-it, is simply weak everywhere (1.19 in English, 1.73 Indic average).

English vs Indic-language correctness (positive gap means worse in Indic languages), sorted by gap:

| model | English | Indic avg | gap |
|---|---|---|---|
| google/gemma-3-4b-it | 1.19 | 1.73 | −0.54 |
| sarvam-30b | 1.54 | 1.51 | +0.03 |
| krutrim-1-7b | 1.69 | 1.54 | +0.15 |
| deepseek-v4-flash | 4.43 | 4.28 | +0.15 |
| sarvam-1-2b | 1.47 | 1.21 | +0.26 |
| gpt-oss-20b | 2.25 | 1.96 | +0.29 |
| gemma-3-27b-it | 3.66 | 3.22 | +0.44 |
| google/gemma-3-12b-it | 2.76 | 2.25 | +0.51 |
| navarasa-2.0-7b | 1.87 | 1.33 | +0.54 |
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

Average gap from English per language (correctness, across all models):

| language | gap |
|---|---|
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

Scope: this repo does not include raw per-row generations (there is no `results/` folder), other benchmarks run against the same pool, the dataset itself, or our own fine-tuned checkpoints and their configs. Full report: [sthanika.ai](https://sthanika.ai/research/indic-agri-advisory-2026)

## Citation

If you use this leaderboard or these configurations, cite this repo and the benchmark.

```bibtex
@software{indic_agri_benchmark_model_configs2026,
  title  = {Indic Agri Benchmark: Model Configs and Leaderboard},
  author = {{sthanika-ai}},
  year   = {2026},
  url    = {https://github.com/sthanika-ai/Indic-Agri-Benchmark-Model-Configs}
}
```

## License

Code (configs, scripts) is MIT, see [LICENSE](LICENSE). This repo contains no benchmark data and no model weights. The dataset is separately licensed and gated, see the benchmark repo.

## Related

- [Indic-KCC-Agri-Advisory-Benchmark](https://github.com/sthanika-ai/Indic-KCC-Agri-Advisory-Benchmark): the evaluation harness and methodology
- [Indic-KCC-Agri-Advisory-Benchmark dataset](https://huggingface.co/datasets/sthanika-ai/Indic-KCC-Agri-Advisory-Benchmark): the corpus, on Hugging Face (gated)
- Site: [sthanika.ai](https://sthanika.ai)
