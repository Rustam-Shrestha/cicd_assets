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






