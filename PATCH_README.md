# Phase 1 auditor format fix

This patch addresses the observed reply containing `prefix_end: "color"` and a hint that supplies a color while asserting an incorrect shape. The original checker correctly rejected the reply. The patch improves the request contract and preserves strict rejection; it does not substitute a fabricated successful response.

## Apply on Windows

1. If PowerShell is showing `>>` after you typed a JSON line such as `"action": "repair",`, press **Ctrl+C** to return to the normal prompt. Model output is data to inspect, not a PowerShell command.
2. Extract this ZIP into a temporary folder.
3. Back up these three existing project files, then copy the matching files from the patch folder into `D:\MedEvo_Phase1`, replacing only:
   - `src\auditor.py`
   - `run_pilot.py`
   - `tests\test_phase1.py`
4. Keep your `.venv312`, `settings.json`, downloaded models, `.gitignore`, and `outputs` folder.
5. From the project root, run the tests with the confirmed interpreter:

```powershell
& 'D:\MedEvo_Phase1\.venv312\Scripts\python.exe' -m unittest discover -s tests -v
```

The updated suite contains 16 tests. Send the summary before the live auditor retry.

## Live retry after reviewing the tests

Run these together in the same PowerShell window:

```powershell
$env:AUDITOR_BASE_URL = 'http://127.0.0.1:11434/v1'
$env:AUDITOR_MODEL = 'qwen2.5vl:3b'
$env:AUDITOR_API_KEY = 'ollama'
& 'D:\MedEvo_Phase1\.venv312\Scripts\python.exe' 'D:\MedEvo_Phase1\run_pilot.py' auditor
```

Environment variables set this way belong to the current terminal session. The key is a local placeholder. No cloud service is configured by these commands.

Every configured probe now saves `auditor_request.json` and, after receiving a response, `auditor_response.json` in its own output folder. The latter contains the HTTP response text before JSON-content validation. A rejected response therefore remains available for inspection. Request headers and credentials are not saved.

If the retry fails, share `result.json` and `auditor_response.json` when present. A timeout or connection failure may have no response file. Do not repeatedly retry or weaken validation to produce a pass.

## Changes

- Requests schema-constrained JSON with exactly three fields, including an integer `prefix_end` in the observable character range.
- Defines a zero-based, end-exclusive character offset and an empty retained prefix.
- Clarifies that a hint must be an inspection instruction, not replacement answer text or a claim about a specific color or shape.
- Uses temperature 0 and seed 7 for this development probe; repeated execution can still vary across systems and backend versions.
- Retains strict type checks. Strings are not converted to integers, and out-of-range values are not clamped.
- Adds fixture-specific rejection of stated color/shape values and preservation of the known wrong fixture findings.
- Saves requests and replies to make failures traceable.

The request does not include the reference answer. The schema's numeric range comes from the length of the visible failed response. The known color/shape checks remain in the local synthetic-fixture verifier, not in the auditor request.

## Verification and limits

All 16 tests passed in the Linux preparation environment. Tests cover the reported malformed boundary, an incorrect shape claim in a hint, an unsupported retained fixture finding, the HTTP request contract, and retention of rejected responses. HTTP tests use a local mock server; they do not establish that Qwen follows the instructions.

The primary-model code, dependencies, model revision, and pilot settings are unchanged. Their earlier checks were not repeated for this narrow auditor patch. The original package's preparation record describes the original 13-test version; this patch's verification record describes the updated suite.

A real Ollama retry and its memory measurements remain pending. A schema controls output structure, not medical correctness, valid reasoning boundaries, or general answer leakage. This is still a synthetic Phase 1 check, not a completed medical evaluation.

Ollama documentation:
- https://docs.ollama.com/capabilities/structured-outputs
- https://docs.ollama.com/api/openai-compatibility
