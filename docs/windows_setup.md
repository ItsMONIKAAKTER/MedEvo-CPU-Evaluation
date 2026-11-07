# Windows and PyCharm setup

This package extends the working Phase 1 code. Keep `D:\MedEvo_Phase1`, its virtual environments and its output folders intact. Extract `MedEvo_Evaluation.zip` separately, for example so the new entry point is `D:\MedEvo_Evaluation\evaluate.py`.

## First checkpoint: interpreter and tests

1. In PyCharm, open the extracted folder containing `evaluate.py`, `run_pilot.py`, `settings.json` and `requirements.txt`. Windows extraction may add an extra folder level; use the folder that contains the scripts.
2. Open **Settings > Python > Interpreter** (or **Project > Python Interpreter**, depending on PyCharm version).
3. Add an existing local interpreter and select `D:\MedEvo_Phase1\.venv312\Scripts\python.exe`.
4. Set the project working directory to the extracted project root.

The old terminal prompt may still display `.venv`. Use the executable path explicitly so it cannot select the old Python 3.14 environment.

In PyCharm's PowerShell terminal, change to the extracted project folder. For example, if it is directly at `D:\MedEvo_Evaluation`:

```powershell
Set-Location 'D:\MedEvo_Evaluation'
$projectPython = 'D:\MedEvo_Phase1\.venv312\Scripts\python.exe'
& $projectPython --version
& $projectPython -m pip install -r requirements.txt
& $projectPython -m pip check
& $projectPython -m unittest discover -s tests -v
```

Expected: Python 3.12.10, no broken requirements, and all tests passing. PyArrow is the only added direct dependency. The pinned requirements also verify that the previously installed supporting packages are present. CPU PyTorch remains in the existing environment. A Python 3.12 Windows x64 wheel for this pinned PyArrow release was verified during package preparation.

**Stop at this checkpoint and share the test summary.** Do not start a dataset download or a benchmark run yet. Installation adds a package to the reused environment; a new dedicated Python 3.12 environment is an alternative if the current one also serves other projects.

For a fresh environment, install the CPU builds first, then the requirements:

```powershell
py -V:3.12 -m venv .venv312
$projectPython = '.\.venv312\Scripts\python.exe'
& $projectPython -m pip install torch==2.15.0 torchvision==0.29.2 --index-url https://download.pytorch.org/whl/cpu
& $projectPython -m pip install -r requirements.txt
& $projectPython -m pip check
```

## Second checkpoint: prepare data

After the tests have been reviewed:

```powershell
& $projectPython evaluate.py prepare
& $projectPython evaluate.py inspect
```

Preparation downloads six pinned files, totalling 251,079,630 bytes, then exports the images and manifests. It may take several minutes or longer depending on the connection and disk. Progress messages identify the source being checked. A Hugging Face symlink warning does not by itself mean the run failed. No token or administrator session is required for these public sources.

The expected final question counts are:

| Dataset | Train | Feedback | Test |
|---|---:|---:|---:|
| VQA-RAD | 655 | 79 | 451 |
| SLAKE, English | 4,911 | 1,053 | 1,061 |

Keep `data/benchmarks/dataset.json` and `exclusions.jsonl`. They record the actual preparation outcome. If a fingerprint or count differs, retain the error and stop rather than substituting another source.

If the data directory already exists, preparation stops without overwriting it. Inspect the existing bundle. To prepare an additional copy, choose another destination with the global option before the command:

```powershell
& $projectPython evaluate.py --data-dir 'D:\MedEvo_Data_Second_Copy' prepare
```

Pass the same `--data-dir` to subsequent commands using that copy. The default path is relative to the script location, not the terminal directory.

## Third checkpoint: small feedback runs

Run one dataset at a time, after reviewing preparation:

```powershell
& $projectPython evaluate.py run --benchmark vqa_rad --limit 5
```

Inspect the printed output folder, then `metrics.json`, `predictions.jsonl` and `resources.json`. The role must say `feedback` and the scope `subset`. Review the result before starting the SLAKE check:

```powershell
& $projectPython evaluate.py run --benchmark slake --limit 5
```

The pretrained primary model is loaded from the existing Hugging Face cache when available. Changing Windows user accounts or the cache location may cause another download. No Phase 1 synthetic adapter is loaded. The evaluator does not use Ollama; its auditor service can remain idle during Phase 2.

Leave the PC awake. Windows sleep caused an earlier pilot timeout and invalidates timing comparisons. The evaluation worker has a 10 GiB sampled memory ceiling and an eight-hour attempt limit. These are safeguards, not runtime forecasts. Generation is limited to 128 new tokens per question. Measure the small runs before planning a full experiment.

Send the run's `metrics.json`, `resources.json` and any failure message. Do not share tokens, virtual environments or cached weights.

## Run configurations

For a Python configuration, choose the existing Python 3.12 interpreter and set **Script path** to `evaluate.py` in this new folder. Use separate configurations:

| Name | Script parameters |
|---|---|
| Evaluation help | Leave empty |
| Prepare benchmarks | `prepare` |
| Inspect benchmarks | `inspect` |
| VQA-RAD feedback | `run --benchmark vqa_rad --limit 5` |
| SLAKE feedback | `run --benchmark slake --limit 5` |

A Run configuration does not inherit environment variables from a different PowerShell session. This matters for the separate auditor probe; Phase 2 evaluation needs no auditor variables.

## Recovering a stopped run

Use the printed run folder and the same benchmark, role, limit, code and settings:

```powershell
& $projectPython evaluate.py run --benchmark vqa_rad --limit 5 --resume 'outputs\YOUR_RUN_FOLDER'
```

Completed journal records are retained. A partial final record is backed up before recovery. A corrupt interior record, changed configuration, or duplicate prediction stops the run.

A force-stopped worker may leave `run.lock`. First confirm that its evaluation process has ended in Task Manager. Only then remove that run's lock file and resume. Do not remove a lock belonging to an active process. Each attempt keeps its own console log and resource record. Failed question records are preserved; the same run does not automatically retry them.

For final test evaluation, wait until repair, adaptation and the protocol review are complete. The README documents the later explicit test commands. Small feedback runs are development checks, not the report's final benchmark results.

Official interpreter documentation: https://www.jetbrains.com/help/pycharm/configuring-python-interpreter.html

## Updating an existing Phase 2 installation

The reported Windows installation has already passed the 38-check suite, prepared and inspected both datasets, and completed five feedback questions from each dataset. An answer-format diagnostic is now available without new dependencies or data downloads. Keep the existing virtual environment, data and output folders when merging this update. The expanded suite contains 45 tests. Follow `prompt_experiment.md` for the exact update and diagnostic command; do not repeat the earlier installation and preparation stages.

## Scoring the completed Windows prompt comparison

The researcher has completed the paired prompt experiment. The next approved update adds a read-only whole-answer diagnostic with no new dependencies or model generation. Keep the existing comparison and trial folders. The expanded suite contains 56 tests. Follow `answer_diagnostic.md`; do not repeat data preparation or prompt generation for this step.

## Phase 3 repair-pilot checkpoint

The update adds a three-question training-only pilot and no new dependencies. Merge the code and documentation while preserving existing data, outputs, settings and environments. First run the expanded 79-test suite from the project root. Then follow `repair_pilot.md` for the live local-auditor command. The Qwen model was downloaded earlier; keep the Ollama desktop service running. The pilot records preliminary checks and performs no medical adapter training.

## Correcting the first live audit interpretation

The reported 79-test suite passed on Windows. The live pilot then returned three non-repair actions with unused hints, which exposed an overstrict validation condition. The current 86-test update provides a read-only `inspect-repair` command; follow `audit_review.md`. Preserve the original pilot and inspect its saved responses before repeating model generation. No new package installation is needed on the existing Windows setup.
