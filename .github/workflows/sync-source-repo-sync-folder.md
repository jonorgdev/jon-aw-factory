---
name: Sync /sync folder from source repo
description: Sync /sync from jonorgdev/jon-source-repo@main into this repository.
on:
  schedule: daily
permissions: read-all
engine: copilot
steps:
  - name: Generate a token
    id: generate-token
    uses: actions/create-github-app-token@v2
    with:
      app-id: ${{ vars.SOURCE_REPO_SYNC_APP_ID }}
      private-key: ${{ secrets.SOURCE_REPO_SYNC_APP_PRIVATE_KEY }}
      owner: jonorgdev
      repositories: jon-source-repo
  - name: Export GH_TOKEN
    env:
      GH_TOKEN: ${{ steps.generate-token.outputs.token }}
    run: echo "GH_TOKEN=$GH_TOKEN" >> $GITHUB_ENV
  - name: Verify app access to source repo
    env:
      GH_TOKEN: ${{ steps.generate-token.outputs.token }}
    run: |
      gh api /repos/jonorgdev/jon-source-repo --jq .full_name >/dev/null
network:
  allowed:
    - "github.com"
safe-outputs:
  create-pull-request:
    title-prefix: "[source-repo-sync] "
    labels: [automation]
    draft: false
---



# Sync /sync folder

Sync the entire /sync folder (including all files and subfolders) from jonorgdev/jon-source-repo@main into the /sync folder of the repository where this workflow resides. This merges upstream files into the local /sync folder (no deletions of local-only files) and opens a pull request with the changes.

## Required setup

1) Create or use a GitHub App in the jonorgdev organization.
2) Generate at least one private key for that App.
3) Install the App on jonorgdev/jon-source-repo.
4) Grant repository permission `Contents: Read` to the App.
5) In each target repository (where this workflow runs), set:
  - Repository variable `SOURCE_REPO_SYNC_APP_ID` = your GitHub App ID.
  - Repository secret `SOURCE_REPO_SYNC_APP_PRIVATE_KEY` = full PEM private key content (including BEGIN/END lines).
  - Repository secret `COPILOT_GITHUB_TOKEN` = fine-grained PAT with `Copilot Requests` permission.

If any setup item is missing, runs can fail with errors like `None of the following secrets are set: COPILOT_GITHUB_TOKEN`, `Not Found`, `Integration must generate a public key`, or `Invalid keyData`.

## Steps


1) Use bash to clone jonorgdev/jon-source-repo@main with sparse-checkout for the /sync folder using the GitHub App token (in `GH_TOKEN`).
2) Merge the remote /sync folder into ./sync in this repository (do not delete local-only files).
3) Check `git status` for changes. If there are changes, use the `create_pull_request` safe output tool to open a PR. Do **NOT** try to `git push` yourself — the safe-outputs job handles pushing and PR creation automatically.

**Important**: The `GH_TOKEN` env var is ONLY for cloning the private source repo. Do NOT use it to push or set the remote URL. Do NOT run `git push` at all. Just commit your changes locally and call the `create_pull_request` tool.

Use this clone command (requires `GH_TOKEN`):

  git clone --depth 1 --filter=blob:none --sparse "https://x-access-token:${GH_TOKEN}@github.com/jonorgdev/jon-source-repo.git" <tmp>

Then:

  cd <tmp>
  git sparse-checkout set sync
  rsync -a "<tmp>/sync/" "$GITHUB_WORKSPACE/sync/"
