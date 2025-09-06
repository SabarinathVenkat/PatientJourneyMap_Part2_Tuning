Prompted vs Fine-Tuned (LoRA/PEFT) Patient-Journey Mapping

This repo compares a prompt-only pipeline to a LoRA/PEFT fine-tuned pipeline (Mistral-7B-Instruct) for extracting structured clinical events from messy notes (referrals, progress notes, discharges). It includes robust JSON parsing, Excel logging, and an evaluation harness with Before vs After charts.

Freeze the base model, train tiny LoRA adapters to learn site-specific phrasing and JSON discipline. Measure JSON validity, event types/dates, provider overlap, latency (P50/P95 per 1k chars), and docs/min—like-for-like.

What It Does

Extraction: Converts free-text chunks into JSON events:

{"event_type": "...", "date": "YYYY-MM-DD|YYYY-MM|YYYY|Unknown", "provider": "...|N/A", "details": "..."}


Fine-tuning: LoRA/PEFT adapters on attention & MLP projections (q/k/v/o, gate/up/down); base model stays frozen (QLoRA).

Evaluation: Unsupervised (format/structure proxies) + optional supervised vs a gold JSONL.

Ops metrics: Latency P50/P95 (ms per 1k chars) and docs/min—normalized, apples-to-apples.

Setup
Requirements

Python 3.10+

CUDA GPU recommended (training/inference)

Install:

pip install -q pandas openpyxl unsloth peft transformers accelerate torch

Data

Place your XLSX outputs into data/ (e.g., Model_Output_BF_Tuned.xlsx, Model_Output_AF_Tuned.xlsx).

Optional gold for supervised evaluation (one JSON per line):

{"input": "<RAW TEXT>", "output": [ { "event_type": "...", "date": "...", "provider": "...", "details": "..." } ]}

How to Use
1) Run the Notebook

Open code/PJM_Part2_FineTuned_Mistral.ipynb (Colab/local). It includes cells to:

Load base model + LoRA config (r=16, lora_alpha=16, dropout=0, Unsloth QLoRA).

Generate baseline outputs (prompt-only) and log latency_ms.

Fine-tune adapters on your reviewed JSON pairs.

Generate tuned outputs on the same inputs/prompt/template (with latency_ms).

Evaluate and write data/Final_Evaluation_Report.xlsx + PNG/PDF charts.

Keep decode params identical (temperature/top-p/max_new_tokens) for like-for-like timing.

2) Evaluation (Unsupervised & Supervised)

The notebook computes:

Unsupervised: JSON Validity Rate, Avg Events/Doc, Allowed Event-Type Rate, Date-Format Valid Rate, Provider Overlap (strict/relaxed), Latency P50/P95 (ms per 1k chars), Docs/min.

Supervised (optional): If data/pjm_gold.jsonl is provided, it adds Event F1 (micro), Date exact/partial match (YYYY-MM tolerant), and Provider token overlap.

Outputs go to data/Final_Evaluation_Report.xlsx with sheets:

summary (Before vs After)

bf_details, af_details (per-row metrics + parse errors)

bf_supervised, af_supervised (if gold provided)

3) Charts

chart_quality.png — Validity, Events/Doc, Event-Type Allowed, Date Valid

chart_provider.png — Strict/Relaxed Provider Overlap

chart_latency_ms_per_1k.png — P50/P95

chart_docs_per_min.png — Throughput

Final_Eval_Charts.pdf — One chart per page

Like-for-Like Fairness

Keep everything identical across runs except weights (baseline vs baseline+LoRA): same inputs, chunking, prompts/templates, tokenizer, decode params, and timing method.

Privacy & Governance

Use de-identified data only; follow your org’s PHI policy.

Conservative extraction: only explicit events, prefer “Unknown,” keep page-local evidence.

Split train/dev/test by patient to avoid leakage.

Record model version, adapter hash, prompts, and evidence spans.