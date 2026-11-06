# MedEvo Evaluation Study

A reproducible evaluation project for a CPU-based adaptation of **MedEvo: Medical Reasoning Evolution through Knowledge Navigation**. The study will compare a pretrained vision-language model with an adapted model on VQA-RAD and SLAKE, retaining the data versions, predictions, settings and resource records needed to explain the results.

This is an independent implementation. It does not contain the authors' MedEvo weights or reproduce their published scores.

## Project status

Phase 2 provides data preparation, split checks, baseline inference and answer scoring. The original [CPU feasibility checks](docs/feasibility.md) remain available. Data preparation has been checked against both pinned dataset releases. Development inference has been checked separately from final evaluation; see [verification](docs/verification.md).

A three-question training-only repair pilot is now available. It records entropy evidence, gold-blind auditing, same-checkpoint continuation and preliminary checks. Medical adaptation, full training admission and final benchmark comparison remain pending. The separate synthetic auditor probe remains a connectivity/schema check.

The project includes a bounded twelve-question training collection: six questions from each dataset, reusing three saved non-repair cases and processing nine new ones. It records question echoes separately and indexes potential candidates without training an adapter. See [the collection guide](docs/training_collection.md).

The reported Windows collection finished with zero potential candidates. Inspection identified answer-readout failures and auditor-quality concerns. A separate approved [saved-reasoning readout experiment](docs/readout_experiment.md) now compares twelve text-only readouts with the saved originals. It makes no new reasoning or auditor calls and admits no training examples.

The reported readout comparison produced two lexical reference matches; inspection found one faithful extraction and one unsupported conclusion. It was not adopted automatically. The next approved diagnostic is an [auditor-only instruction comparison](docs/audit_experiment.md): twelve saved cases, identical request controls and a shorter system instruction. It performs no primary-model generation, continuation or training.

The shorter-instruction trial subsequently passed existing audit validation on only 2/12 cases, versus 11/12 originals, with zero repairs passing hint checks. It is not adopted. Simple image controls demonstrated correct colors/shapes and valid constrained JSON; they did not establish medical-auditing quality. The approved next step is a [single-case schema-grounding check](docs/single_audit_check.md), preserving the original instruction and adding the schema explicitly. Core method validation and final benchmark results remain incomplete.

The schema-grounded 3B and one-case 7B replies both failed the existing repair validator. The [one-case model comparison](docs/auditor_model_comparison.md) is retained as a negative result. A subsequent independent reference-blind review of eleven saved training cases proposed five repairs; three passed the existing hint guard, including one with a nonempty supported-prefix proposal. These annotations remain unverified and no medical examples have been admitted or trained.

A [single saved-audit continuation test](docs/saved_continuation.md) is available. It reuses the selected question, image, original reasoning and sealed independent audit. It generates at most one 128-token continuation and one 32-token answer readout, with no auditor call or training. Run it from the existing Windows project:

```powershell
& '.\.venv312\Scripts\python.exe' '.\continue_case.py' --collection-dir '.\outputs\20261108T061937Z_training_collection_9120a6'
```

The reported Windows continuation completed in 14.078 seconds with a measured Python-worker peak of 2.401 GiB. It preserved the prefix but rejected the suffix as empty or truncated, before any answer readout. This is not a successful repair and no training example was admitted. The output folder is `outputs/continuation_case_v1/`; an existing attempt is never overwritten.

The next approved experiment is a [larger answering-model check](docs/larger_primary.md), using the already installed `qwen2.5vl:7b` through Ollama on the same saved training image/question. It sends one CPU generation request with the original instruction and 128-token limit, recording the output and runtime metadata. It does not perform a new audit, repair continuation or adapter training:

```powershell
& '.\.venv312\Scripts\python.exe' '.\check_primary.py' --collection-dir '.\outputs\20261108T061937Z_training_collection_9120a6'
```

The reported Windows 7B answering run completed on CPU in 32.478 seconds. Its 128-token response stopped at the token limit, so no finished answer was obtained. The server reported 5.278 GiB allocation and zero VRAM; allocation is not a peak RAM measurement. The backend and quantization differ from the small primary model, so this is an exploratory feasibility check rather than a controlled comparison of model size alone.

The approved follow-up increases only the output budget to 512 tokens and preserves the completed 128-token baseline:

```powershell
& '.\.venv312\Scripts\python.exe' '.\check_primary.py' --collection-dir '.\outputs\20261108T061937Z_training_collection_9120a6' --max-new-tokens 512 --baseline-dir '.\outputs\larger_primary_v1'
```

The reported 512-token run finished in 67.771 seconds on CPU, preserved the baseline and again stopped at the output limit. It did not reach a finished answer or produce a training example. Both raw outcomes are retained.

The approved next experiment changes only the answering instruction to request at most three brief image-grounded reasoning steps and a short boxed final answer. It retains the same model, image, question and 512-token budget, validating the original-prompt 512-token baseline:

```powershell
& '.\.venv312\Scripts\python.exe' '.\check_primary.py' --collection-dir '.\outputs\20261108T061937Z_training_collection_9120a6' --max-new-tokens 512 --baseline-dir '.\outputs\larger_primary_512_v1' --concise-prompt
```

The reported concise-prompt run finished naturally at 104 tokens in 28.9 seconds, produced three steps and a boxed answer, and preserved the baseline. Its answer did not match the saved reference. The prompt adaptation solved completion/formatting for that case, not answer correctness. No training example was admitted.

The next approved step freezes that 7B configuration for a [ten-question feedback evaluation](docs/primary_feedback.md), reusing five fixed questions from each dataset. It records accuracy, formatting, failures and timing with the unchanged scorer. This exploratory baseline does not establish full benchmark performance or MedEvo improvement:

```powershell
& '.\.venv312\Scripts\python.exe' '.\evaluate_primary.py' --profile-dir '.\outputs\larger_primary_concise_v1' --vqa-rad-run '.\outputs\20261108T023429Z_vqa_rad_feedback_119aa2' --slake-run '.\outputs\20261108T023954Z_slake_feedback_a5e572'
```

The reported Windows batch completed all ten requests in 266.723 seconds, with no parsing failures or truncation. Strict exact match was 0/5 on VQA-RAD and 3/5 on SLAKE (3/10 overall). These are feedback-subset observations. Two additional responses had wording-related mismatches; the unchanged scorer still counts them as wrong. Other outputs included substantive or ambiguous answers, and an exact answer match does not validate its reasoning.

The [tiny CPU quantized-training backend test](docs/cpu_training_backend.md) passed on Windows: one synthetic NF4/LoRA update and packed-base/adapter reload, 5.141 seconds total and 0.377594 GiB sampled worker peak. Full-model training remains untested. This admits no medical examples and provides no medical adaptation result.

The next approved experiment is a [twenty-question Qwen3.5 feedback baseline](docs/qwen35_feedback.md), using the already installed `qwen3.5:9b`, ten feedback questions per dataset, the paper's Appendix D.1 instruction, thinking disabled and temperature zero. It records accuracy, boxed-answer rate, lengths, timing, truncations and failures without retries. There is no training, download or test-set generation:

```powershell
& '.\.venv312\Scripts\python.exe' '.\evaluate_feedback.py'
```

The model's quantized CPU backend, reduced budget and thinking setting are declared study adaptations. The reported Windows run completed all twenty questions in 739.952 seconds: VQA-RAD 3/10 exact matches, SLAKE 7/10, 16/20 valid boxes and four truncated responses. Inspection reproduced those scores and confirmed the four parsing failures were the four truncations. This is a feedback baseline, not MedEvo improvement.

The revised [expanded feedback experiment](docs/expanded_feedback.md) first audits all 79 VQA-RAD and forty SLAKE feedback IDs offline, including the earlier twenty. A separately approved run uses a 1024-token ceiling with otherwise identical generation controls. It retains the old scorer and adds a separately versioned formatting-wrapper scorer. Begin with the audit only:

```powershell
& '.\.venv312\Scripts\python.exe' '.\compare_feedback_budget.py' inspect
```

The full 119-question run awaits review and approval of the actual Windows selection audit. Medical repair/admission, adaptation, final test comparison and the technical report remain incomplete.

## Models

| Component | Configuration |
|---|---|
| Primary model | `HuggingFaceTB/SmolVLM2-500M-Video-Instruct` |
| Model revision | `7b375e1b73b11138ff12fe22c8f2822d8fe03467` |
| Inference | CPU, float32, greedy decoding, batch size 1, four threads |
| Image input | One view, longest edge 512 pixels, image splitting disabled |
| Evaluation budget | At most 128 new tokens per question |
| Auditor used in the local feasibility check | `qwen2.5vl:3b` through Ollama; a separate model, not called by the Phase 2 evaluator |

The smaller backbone and reduced image resolution are practical study choices. They limit comparison with the paper's Qwen3.5 experiments. A successful software check does not establish medical accuracy.

## Getting started

Use Python 3.12. The Windows instructions in [docs/windows_setup.md](docs/windows_setup.md) explain how to reuse the working interpreter without replacing the existing project.

```powershell
python -m unittest discover -s tests -v
python evaluate.py
```

The development suite includes an optional real CPU NF4 update/reload check when the pinned bitsandbytes package is installed. Its success checks software behavior, not medical reasoning quality or completion of the research assignment.

Running `evaluate.py` without a command prints help. It does not download data or run a model. The following commands are separate checkpoints:

```powershell
python evaluate.py prepare
python evaluate.py inspect
python evaluate.py run --benchmark vqa_rad --limit 5
python evaluate.py run --benchmark slake --limit 5
```

The two `run` commands use **feedback examples**, not the final test sets. Review each output before increasing the workload. Keep the PC awake while a command is running.

## Answer-format experiment

A separate diagnostic compares a concise instruction with the two completed five-question feedback runs. It reuses their exact selections, retains the original evidence, and keeps the model, image processing, token budget and scorer fixed. It produces new raw predictions and a comparison report. The original instruction remains the default. In the Linux preparation check, the concise instruction produced shorter responses but no valid answer boxes; the negative outcome is retained in [the diagnostic verification](docs/prompt_verification.md).

See [docs/prompt_experiment.md](docs/prompt_experiment.md) for the approved command, update instructions and interpretation limits. This diagnostic does not train a model or use final test questions.

## Whole-answer scoring diagnostic

The saved prompt comparison can also be inspected under a separate whole-answer exact-match rule. It reuses existing predictions, keeps all boxed scores unchanged, and reports both metrics side by side. There is no new model generation. See [docs/answer_diagnostic.md](docs/answer_diagnostic.md) for the approved Windows command and exact rules.

The new score is an exploratory diagnostic, not a model improvement or an adopted final-evaluation protocol. Local preparation gave 3/5 whole-answer matches per concise trial; boxed accuracy stayed at 0/5. The ten reused feedback questions do not establish benchmark performance.

## Training-only repair pilot

The next checkpoint runs two eligible VQA-RAD and one SLAKE training questions through reasoning, token-entropy measurement, a local auditor and supported-prefix continuation when justified. A separate short answer readout supports training-only lexical verification. The original model, data roles, evaluation defaults and boxed scorer remain unchanged.

See [docs/repair_pilot.md](docs/repair_pilot.md) for the test checkpoint, one-command run, request boundaries, budgets and interpretation. Preliminary candidates are recorded for review; the pilot exports no training targets and performs no adapter updates. Live medical auditing requires the local Qwen service on the researcher's PC.

## Saved-audit review

A correction to the pilot validator preserves skip/unresolved decisions even when their unused auxiliary fields contain text. Those hints remain recorded and never reach continuation. A read-only `inspect-repair` command can review earlier rejected audits in a separate report while preserving all original files. See [docs/audit_review.md](docs/audit_review.md). The first reported Windows pilot still contains zero repair decisions; the correction does not create successful repairs.

## Datasets

Source revisions and SHA-256 fingerprints are recorded in `configs/data_sources.json`. Preparation downloads approximately 251 MB of source files, verifies them, decodes the images and creates manifests in `data/benchmarks/`. The cache, extracted images and model weights need additional space; none are included in this repository.

| Dataset | Eligible training questions | Feedback questions | Final test questions |
|---|---:|---:|---:|
| VQA-RAD | 655 | 79 | 451 |
| SLAKE, English | 4,911 | 1,053 | 1,061 |

These are the counts produced by the pinned distributions and the declared split policy, not the counts of the original papers. VQA-RAD training questions that reuse a test image are excluded. The SLAKE validation set supplies feedback; VQA-RAD feedback uses a deterministic selection of eligible training image groups. Feedback images never enter training. Exact repeated adaptation questions are deduplicated, with every exclusion recorded. Patient-level and near-duplicate separation are not established.

Preparation refuses to overwrite an existing data folder. Use `inspect` to validate it. See [data sources](docs/data_sources.md) and [evaluation protocol](docs/protocol.md) for the provenance, exclusions and limitations.

## Outputs and scoring

Each inference command creates its own folder under `outputs/`. Predictions are appended and flushed after every question. Reference answers and answer-type annotations are used by the scorer, never supplied to the model.

| File | Contents |
|---|---|
| `run.json` | Selected IDs, role, source revisions, model settings, protocol and code hashes |
| `environment.json`, `dependencies.json` | Interpreter, hardware inventory and installed package versions |
| `predictions.jsonl` | Raw model answers, token counts, generation durations and failures |
| `scored.jsonl` | Parsed answer, reference and per-question scores |
| `metrics.json` | Coverage, strict and normalized exact match, token F1, format failures and open/closed breakdowns |
| `resources.json`, `resource_history.jsonl` | Latest worker measurements and retained attempt history |
| `console_*.log` | A separate console log for each attempt |

The primary metric is case-insensitive exact match on the last valid `\boxed{...}` answer. An absent or malformed final box counts as wrong. Normalized exact match and token F1 are secondary lexical measures. These rules are explicitly named `boxed-strict-em-v1`; they differ from parts of MedUniEval and do not yield directly comparable paper scores.

Incomplete generation leaves the primary accuracy unset and reports coverage. Scores for successfully generated answers remain clearly identified as partial. A token-limit warning is retained even when an answer can be parsed.

To score stored predictions or resume a stopped run:

```powershell
python evaluate.py score --run-dir 'outputs\YOUR_RUN_FOLDER'
python evaluate.py run --benchmark vqa_rad --limit 5 --resume 'outputs\YOUR_RUN_FOLDER'
```

Use the original command's benchmark, role and limit when resuming. Changed data, code, model settings or direct dependency versions are rejected. Existing generation-error records are retained, not silently retried. See the protocol for interrupted-write recovery and lock files.

## Final evaluation

Freeze the implementation and evaluation settings before using either final test set. At that point, a full baseline run can be requested explicitly:

```powershell
python evaluate.py run --benchmark vqa_rad --role test --full --final
python evaluate.py run --benchmark slake --role test --full --final
```

These commands are documented for the later evaluation stage. Phase 2 does not run them automatically. Test results must not guide prompt selection, repair settings or training. The baseline and adapted model will need the same selected questions and scoring protocol. An adapted-model evaluation option will be added with the training implementation.

## Source layout

| Path | Responsibility |
|---|---|
| `evaluate.py` | Data preparation, validation, evaluation and scoring commands |
| `run_pilot.py` | Environment, synthetic inference, LoRA update/reload and auditor checks |
| `check_auditor.py` | One saved medical audit with the original instruction and explicit schema grounding |
| `check_training_backend.py` | Synthetic CPU NF4/PEFT update and packed-checkpoint reload check |
| `evaluate_feedback.py` | Fixed twenty-question Qwen3.5 feedback baseline with timing and answer-length metrics |
| `compare_feedback_budget.py` | Offline 119-question selection audit and separate expanded budget experiment |
| `feedback_scoring.py` | Versioned formatting-wrapper companion scorer; historical scoring remains unchanged |
| `src/benchmarks.py` | Pinned downloads, image export, data roles and integrity checks |
| `src/evaluation.py` | Deterministic selection, run identity, prediction journal and scoring joins |
| `src/scoring.py` | Box parsing, exact match and token F1 |
| `src/answer_diagnostic.py` | Read-only whole-answer diagnostic and paired scoring report |
| `src/prompt_experiment.py` | Paired feedback prompt diagnostic, evidence copies and comparison report |
| `src/model.py` | Pretrained model, generation and the synthetic LoRA feasibility check |
| `src/auditor.py` | Synthetic auditor request and schema checks |
| `src/runtime.py` | Environment inventory, process limits and atomic JSON reports |
| `src/repair_model.py` | Entropy measurement, training-only readouts and verified assistant-prefill continuation |
| `src/repair_audit.py` | Local auditor/review requests, schemas and conservative hint checks |
| `src/repair_pilot.py` | Training-only selection, case orchestration, evidence and summary |
| `src/repair_review.py` | Read-only revalidation of saved audit decisions with original outcomes retained |
| `src/training_collection.py` | Fixed twelve-case collection, archived-case reuse, readout diagnostics and candidate evidence indexes |
| `src/readout_experiment.py` | Separate twelve-case text-only readout diagnostic with preserved evidence and paired reporting |
| `src/audit_experiment.py` | Separate twelve-case auditor instruction comparison with unchanged request controls and validation |
| `configs/` | Data-source fingerprints and evaluation protocol |
| `tests/` | Split, scoring, resume, model-interface and feasibility checks |
| `docs/` | Setup, provenance, protocol, verification and experiment log |

No website, API server or GitHub account is required. Publishing remains a separate manual action; [docs/github.md](docs/github.md) describes the repository checks.

## Research scope

The completed study needs baseline and adapted results on both datasets, an experimental record, error analysis and a technical report. This checkpoint supplies the measurement infrastructure. It does not complete that comparison or guarantee a particular research outcome.

Method: Wang et al., *MedEvo: Medical Reasoning Evolution through Knowledge Navigation*, supplied technical report, [arXiv:2610.17690](https://arxiv.org/abs/2610.17690). Project resources: [MedEvo](https://github.com/pork-pay/MedEvo). Evaluation reference: [MedUniEval](https://github.com/ZJUI-AI4H/Hulu-Med/tree/6d4d47aa014342b1beda70695ab7c799edc7ade6/MedUniEval). Primary model: [SmolVLM2](https://huggingface.co/HuggingFaceTB/SmolVLM2-500M-Video-Instruct).
