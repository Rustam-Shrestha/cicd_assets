# cicd_assets
## 1. Branch Management
- Goal: Reset `pr-workflow` branch to match `main` without merging.
- Solution:
  - Checkout `pr-workflow` → `git reset --hard main` → `git push origin pr-workflow --force`.
  - Alternative: Delete and recreate `pr-workflow` from `main`.
- Result: Both branches exist independently; `pr-workflow` starts identical to `main` but can diverge.

---

## 2. Workflow File Location
- GitHub Actions only detects workflow files inside `.github/workflows/`.
- Practice files outside this directory (e.g., `demo.yaml`) are ignored.
- Fix: Create `.github/workflows/ci.yaml` and commit it.

---

## 3. Workflow Triggers
- Example starter workflow (`blank.yaml`) included both `push` and `pull_request` triggers:
  ```yaml
  on:
    push:
      branches: [main]
    pull_request:
      branches: [main]
    workflow_dispatch:
Recommendation: Keep both triggers for flexibility during learning.

Key distinction:

Push → runs when commits are pushed to main.

Pull Request → runs when a PR targets main.

4. Pull Request Workflows
To trigger pull_request workflows:

Work on pr-workflow.

Push changes.

Open a PR from pr-workflow → main.

Logs are visible under the Actions tab, not directly on the PR page.

5. Path Issues in Runner
Error encountered:

Code
python: can't open file '/home/runner/work/cicd_assets/cicd_assets/cicd_assets/test.py'
Cause: GitHub runner already places you inside /home/runner/work/<repo>/<repo>/.

Fix: Run scripts relative to repo root:

yaml
run: python test.py
6. Environment Variables and Contexts
Contexts provide dynamic data: github, env, secrets, job, runner.

Access syntax: ${{ github.actor }}, ${{ secrets.MY_SECRET }}, ${{ env.MY_VAR }}.

Variables (env) → non-sensitive config.

Secrets (secrets) → encrypted values, masked in logs.

7. Secrets Management
Secrets are added via Settings → Secrets and Variables → Actions → New repository secret.

Once saved, values cannot be viewed again; only names are visible.

Testing secrets:

yaml
run: echo "Secret length: ${#RUSTAM_SECRET_VAULT}"
env:
  RUSTAM_SECRET_VAULT: ${{ secrets.RUSTAM_SECRET_VAULT }}
If secret is missing, workflow fails with “Environment variable not found”.

8. GITHUB_TOKEN
Built-in secret automatically available in workflows.

Enables:

Cloning repos

Opening/closing PRs

Commenting on PRs/issues

Permissions can be elevated:

yaml
permissions:
  pull-requests: write
9. Automated PR Comments
Example workflow to comment on PR:

yaml
jobs:
  comment-on-pr:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - name: Comment on PR
        uses: thollander/actions-comment-pull-request@v2
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          message: |
            Hello world! This is an automated comment.
Custom secret (RUSTAM_SECRET_VAULT) can be used if defined.

10. Consolidated Workflow Example
yaml
name: PR

on:
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'
      - name: Run test.py
        run: |
          echo "Starting test.py execution"
          python test.py
          echo "Finished test.py execution"

  comment-on-pr:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - name: Check secret length
        run: echo "Secret length: ${#RUSTAM_SECRET_VAULT}"
        env:
          RUSTAM_SECRET_VAULT: ${{ secrets.RUSTAM_SECRET_VAULT }}
      - name: Comment on PR
        uses: thollander/actions-comment-pull-request@v2
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          message: |
            Hello world! This is an automated comment.












november 11 higlights

# Traininng model in github actions 

we need to have the data  as 
processed_dataset/
rsw_dataset/
  weather.csv
metrics_and_plots.py
model.py
preprocess_dataset.py
train.py
utils_and_constants.py

these are supposed to be there nad thing is we need to have our stuffs above 

the github will execute them as i will have bloank.yaml in .github/preprocess/
that will be having script to execute teh preprocess and train 
the model we will see the confusion mtrix in png 
and another preprocessed csv in preprocessedd_dataset and finally 
.json file will shot the accuracy metric calculated
that is already given in train.py file

##  Workflow Goals

- Automate model training and evaluation on pull requests to `main`
- Generate and comment evaluation metrics using CML
- Track datasets using DVC for reproducibility
- Ensure all dependencies and outputs are correctly handled in GitHub Actions

---

##  GitHub Actions Setup

### Workflow file: `.github/workflows/model_training.yaml`

Key components:
- Trigger: `on: pull_request` to `main`
- Python setup: `python-version: 3.9`
- Dependency install via `pip install -r requirements.txt`
- CML setup for PR commenting
- Execution of `preprocess_dataset.py` and `train.py`
- CML report generation from `metrics.json` and `confusion_matrix.png`

---

##  Issues and Fixes

### 1. **Workflow Not Triggering**
- Cause: `blank.yml` interfered with GitHub indexing
- Fix: Renamed `blank.yml` to `.txt` and confirmed `model_training.yaml` was correctly placed

### 2. **Empty Folder Not Tracked**
- Cause: Git ignores empty folders
- Fix: Added `.gitkeep` to preserve folder structure

### 3. **Missing Python Packages**
- Error: `ModuleNotFoundError: No module named 'pandas'`
- Fix: Added `pip install` step using `python3 -m pip install -r requirements.txt`

### 4. **Windows-only Package Error**
- Error: `pywin32==310` not found on Ubuntu runner
- Fix: Removed or OS-conditioned the package in `requirements.txt`

### 5. **Matplotlib Version Conflict**
- Error: `matplotlib==3.10.1` requires Python ≥3.10
- Fix: Downgraded to `matplotlib==3.9.0` for Python 3.9 compatibility

### 6. **Files Not Persisting in GitHub Actions**
- Issue: `metrics.json` and `confusion_matrix.png` not visible
- Cause: GitHub Actions uses a temporary workspace
- Fix: Verified file creation in code and optionally added `actions/upload-artifact` to expose outputs

### 7. **ImportError in `train.py`**
- Error: `cannot import name 'evaluate_model' from 'model'`
- Fix: Ensured `evaluate_model()` was defined in `model.py` or removed the import if unused

---

## DVC Integration

### Commands Used

```bash
dvc init
dvc add data.csv
git add data.csv.dvc
git commit -m "Track data.csv with DVC"
dvc remote add -d localremote /mnt/data/dvc-storage
dvc push
Key Concepts
.dvc files store metadata (hash, size, path)

Actual data stored in .dvc/cache/ or remote

Git tracks metadata; DVC handles data sync

Use dvc push and dvc pull to sync with remote

 Files Created During Workflow
metrics.json → model evaluation metrics

confusion_matrix.png → visual output of classification performance

model_eval_report.md → markdown summary for CML comment

.dvc/config → remote storage configuration

data.csv.dvc → metadata for tracked dataset

 Best Practices Learned
Always validate Python and pip alignment using which python3 and pip --version

Use python3 -m pip install to avoid environment mismatch

Pin versions in requirements.txt for reproducibility

Use .gitkeep to preserve empty folders

Upload artifacts explicitly if needed for review

Use CML only after confirming files exist in workspace



