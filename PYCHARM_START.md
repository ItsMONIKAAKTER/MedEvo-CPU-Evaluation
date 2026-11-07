# Run Phase 1 in PyCharm on Windows

Work through one stage at a time. Our first shared checkpoint is the environment report. Keep your existing MedEvo work; extract this ZIP into its own folder.

## 1. Extract and open

1. Right-click `MedEvo_Phase1.zip` in File Explorer and choose **Extract All**.
2. Choose a normal user folder, such as `Documents\MedEvo_Phase1`. Avoid `C:\Windows\System32`.
3. In PyCharm choose **File > Open**.
4. Open the folder containing `run_pilot.py`, `settings.json`, and `requirements.txt`. Select the inner folder if extraction created an extra directory level.

The scripts locate settings and outputs relative to their own project folder. A different working directory does not change where results are saved.

## 2. Select your Python interpreter

You already installed CPU PyTorch in a virtual environment. Reuse that interpreter if it is available.

1. Open **Settings > Python > Interpreter**. Some versions show **Settings > Project > Python Interpreter**.
2. Choose **Add Interpreter > Add Local Interpreter**, then select an existing environment.
3. Browse to the existing virtual environment's `Scripts\python.exe`. Choose the actual file on your computer; do not copy another person's username.
4. Apply the change. Copy the interpreter's full executable path for the terminal commands below.

If you need a fresh environment instead, create a Virtualenv in this project's `.venv` folder with Python 3.12. You will need to install CPU PyTorch there; environments do not share their installed packages.

Official interpreter instructions: https://www.jetbrains.com/help/pycharm/configuring-python-interpreter.html

## 3. Run the first check

Open `run_pilot.py`, right-click in the editor, and select **Run 'run_pilot'**.

Leave script parameters empty. The default stage is `environment`. This inventories packages and hardware, writes a report, and does not download a model or call an auditor.

Look for:

```text
Output folder: ...\outputs\..._environment_...
```

Open `result.json` inside that folder. `passed` means the required package distributions were found. `blocked` with `missing_packages` identifies what still needs installing. Either outcome is useful; it does not prove model inference or training.

**Pause here and share `result.json` before continuing with the later stages.** It contains your interpreter path and hardware information; remove identifying path details if you prefer. Never share API keys or tokens.

## 4. Install missing dependencies when we reach that step

Open PyCharm's Terminal in the project folder. In PowerShell, set a variable to the interpreter path you selected above:

```powershell
$projectPython = 'PASTE_YOUR_SELECTED_PYTHON_EXE_PATH_HERE'
& $projectPython --version
& $projectPython -m pip install -r requirements.txt
& $projectPython -m pip check
```

Replace the placeholder with the real executable path, retaining the quotes. If your interpreter is this project's `.venv`, you can use:

```powershell
$projectPython = '.\.venv\Scripts\python.exe'
```

If torch and torchvision are missing, install the CPU builds before the other requirements:

```powershell
& $projectPython -m pip install torch==2.15.0 torchvision==0.29.2 --index-url https://download.pytorch.org/whl/cpu
& $projectPython -m pip install -r requirements.txt
```

These match the tested CPU versions. If a wheel is unavailable, save the exact error and stop instead of choosing random alternatives. The supporting dependencies are pinned in `requirements.txt`; installing them can change versions in a reused environment. Use a dedicated environment if that environment also serves unrelated projects.

Run the environment check again after installation.

## 5. Run the software checks

After the environment is ready:

```powershell
& $projectPython -m unittest discover -s tests -v
```

The tests use synthetic data and a local mock HTTP server. They do not download weights, use paid services, or produce medical benchmark scores. Some tests intentionally stop child processes to check the limits.

## 6. Run the inference probe

After approval for the model download:

```powershell
& $projectPython run_pilot.py inference
```

The first run downloads the public 500M checkpoint and processor. Later runs reuse the Hugging Face cache when available. The package has no GPU requirement and does not install FlashAttention.

The input is a generated practice image and a short color/shape question. Inspect:

- `result.json`: raw response, input shapes, token count, and generation time.
- `resources.json`: total elapsed time and sampled memory.
- `console.log`: worker output and any error.

A response from this fixture is a feasibility observation, not a medical performance result. A warning about Windows symlinks can affect cache efficiency without blocking execution. Diagnose the final result rather than treating every warning as a failure.

If `operation_checks` is `passed` but the overall status is `blocked` / `resource_measurement_unverified`, the model operation worked but the memory reading was unavailable or implausible. Keep both report files and inspect this on your PC; do not record the memory limit as validated. That condition occurred in the Linux preparation environment and is documented in `docs/VERIFICATION.md`.

## 7. Run the training-update probe

After reviewing inference and receiving approval to continue:

```powershell
& $projectPython run_pilot.py train
```

This loads a fresh original primary model, attaches a small LoRA adapter, and performs one update using the synthetic fixture. It verifies prompt masking, finite loss/gradients, a changed adapter parameter, an unchanged frozen-weight sample, and matching logits after saving/reloading the adapter.

The stage succeeds only when its verification passes. It does not use or modify your earlier Qwen checkpoint. The result describes a single synthetic update, not a MedEvo-trained medical model.

The saved adapter depends on the original model revision recorded in `base_model.json`. Only use an adapter directory whose run ended with `adapter_reload_passed: true`.

## 8. Auditor connectivity — a separate checkpoint

This requires an approved, image-capable OpenAI-compatible endpoint. A subscription to a chat application does not by itself establish API access. A configured local service can also be used if it supports this request format.

Set these environment variables in **Run > Edit Configurations > Environment variables** for an auditor run:

- `AUDITOR_BASE_URL`: the API base ending in `/v1`, not the full `/chat/completions` path.
- `AUDITOR_MODEL`: the actual independent auditor model identifier.
- `AUDITOR_API_KEY`: only if the service requires a key.

Set script parameters to `auditor` and run that configuration. This command makes one external call when configured and may incur the service's charge. Select the provider and budget before using it. Credentials remain outside the source files. Do not store shared PyCharm run configurations containing them.

The probe includes an image, question, and deliberately wrong response. It excludes the reference answer. It checks JSON structure, boundary range, and an elementary answer-free-hint condition for this fixture. It cannot establish medical competence, full independent model identity, or general protection against answer leakage.

If the endpoint is unset, the correct outcome is `blocked` / `auditor_not_configured`. Provider-specific API incompatibility is a recorded failure, not a reason to mark the auditor as working.

## PyCharm run configurations

You can use these instead of terminal commands. Choose a Python configuration with script path `run_pilot.py`, your selected interpreter, and the following parameters:

| Configuration name | Script parameters |
|---|---|
| Phase 1 — Environment | Leave empty |
| Phase 1 — Inference | `inference` |
| Phase 1 — Training update | `train` |
| Phase 1 — Auditor | `auditor` |

Create separate configurations so clicking Run does not accidentally start a model download or service call. The program never runs all stages automatically.

## What to send back

For each approved stage, share `result.json` and `resources.json` if present. Keep the full output folder locally. If there is an error, send the error message and stage name. Do not upload your `.venv`, model weights, or credentials to GitHub.

Phase 1 remains open until the primary-model operations, resource measurements on your PC, auditor access, and repository setup have been reviewed. Later phases implement and evaluate the medical method.
