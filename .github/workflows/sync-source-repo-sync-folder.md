---
title: Sync /sync folder from jon-source-repo
description: Keeps the /sync folder in this repo in sync with the /sync folder from jonorgdev/jon-source-repo using a GitHub App token.
agent: copilot
triggers:
  - workflow_dispatch
  - schedule: '0 3 * * *' # Daily at 3am UTC
  - repository_dispatch # Allows triggering from source repo webhook
permissions:
  contents: write
  id-token: write
  actions: read
secrets:
  - SOURCE_REPO_SYNC_APP_PRIVATE_KEY
vars:
  - SOURCE_REPO_SYNC_APP_ID
---

## Usage
This workflow syncs the `/sync` folder from `jonorgdev/jon-source-repo` into the `/sync` folder of this repository. It uses a GitHub App token for authentication.

## Steps
1. **Generate GitHub App Token**
   - Uses `actions/create-github-app-token@v2` to generate a token with code read access.
2. **Clone the source repo's /sync folder**
   - Uses the generated token to fetch only the `/sync` folder from `jonorgdev/jon-source-repo`.
3. **Replace local /sync folder**
   - Overwrites the local `/sync` folder with the fetched contents.
4. **Commit and push if changes**
   - If there are changes, commits and pushes them to the current branch.

---

# Workflow Implementation

```yaml
on:
  workflow_dispatch:
  schedule:
    - cron: '0 3 * * *'

jobs:
  sync-folder:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      id-token: write
      actions: read
    steps:
      - name: Checkout target repo
        uses: actions/checkout@v4

      - name: Generate GitHub App token
        id: generate-token
        uses: actions/create-github-app-token@v2
        with:
          app-id: ${{ vars.SOURCE_REPO_SYNC_APP_ID }}
          private-key: ${{ secrets.SOURCE_REPO_SYNC_APP_PRIVATE_KEY }}

      - name: Remove existing /sync folder
        run: rm -rf sync

      - name: Download /sync folder from source repo
        env:
          GH_TOKEN: ${{ steps.generate-token.outputs.token }}
        run: |
          gh repo clone jonorgdev/jon-source-repo source-tmp -- -q --depth=1
          cp -r source-tmp/sync ./sync
          rm -rf source-tmp

      - name: Create branch, commit, and open PR if changes
        env:
          GH_TOKEN: ${{ steps.generate-token.outputs.token }}
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git checkout -b sync-source-update-$(date +%Y%m%d%H%M%S)
          git add sync
          if ! git diff --cached --quiet; then
            git commit -m "Sync /sync folder from jonorgdev/jon-source-repo"
            git push --set-upstream origin $(git branch --show-current)
            gh pr create --title "Sync /sync folder from jonorgdev/jon-source-repo" --body "Automated PR to sync /sync folder from source repo." --base ${{ github.ref_name }} --head $(git branch --show-current) --label "sync-bot"
          else
            echo "No changes to commit."
          fi
```

---

## Notes
- The workflow will only commit if there are changes in the `/sync` folder.
- Requires the GitHub App to have code read access on the source repo and write access on the target repo.
Value: Paste your GitHub App’s App ID.
