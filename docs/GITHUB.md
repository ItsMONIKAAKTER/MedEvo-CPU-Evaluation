# Publishing the project

Repository name: `MedEvo-CPU-Evaluation`.

Suggested description: "Reproducible CPU evaluation of a MedEvo-inspired medical VQA adaptation on VQA-RAD and SLAKE, with versioned data, predictions and experiment records."

The current release covers feasibility and evaluation infrastructure. Keep that status in the README until repair, adaptation and final experiments are complete. Do not describe development checks as published medical benchmark results.

Use your existing project repository when available. Review filename conflicts before copying this package into it; preserve its `.git` history and merge useful ignore rules. GitHub publication is a manual action and is not performed by the scripts.

```powershell
git status
git remote -v
git diff
```

For a new repository only:

```powershell
git init -b main
git add .
git diff --cached --stat
git diff --cached
git commit -m "Add benchmark preparation and reproducible evaluation"
```

Use your own Git identity. Review the staged content before creating a remote or pushing. If using GitHub CLI:

```powershell
gh auth status
gh repo create MedEvo-CPU-Evaluation --private --source=. --remote=origin
git push -u origin main
```

Only run `gh repo create` if that repository does not already exist. If login is needed, complete `gh auth login` yourself. Do not put tokens in code, shared run configurations or screenshots.

`.gitignore` excludes virtual environments, local data, raw output directories, model files and environment secrets. Check that no cache or untracked large file is staged. Cite MedEvo, the primary model and the dataset sources. Before making the repository public, review data/model rights and select an appropriate licence for original code; none is assigned automatically here.

Later, export selected result tables and sanitized experiment records deliberately. The complete local run folders should remain available as evidence, even when omitted from Git. Keep failed and partial runs labelled accurately.

CLI reference: https://cli.github.com/manual/gh_repo_create
