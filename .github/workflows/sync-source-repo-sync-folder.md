---
name: Sync /sync and /target-folder
description: Sync /sync and /target-folder from jonorgdev/jon-source-repo@main into this repository.
on:
  schedule: daily
permissions: read-all
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

## Steps


1) Use bash to clone jonorgdev/jon-source-repo@main with sparse-checkout for the /sync folder using the GH app token (in `GH_TOKEN`).
2) Merge the remote /sync folder into ./sync in this repository (do not delete local-only files).
3) Check `git status` for changes. If there are changes, use the `create_pull_request` safe output tool to open a PR. Do **NOT** try to `git push` yourself — the safe-outputs job handles pushing and PR creation automatically.

**Important**: The `GH_TOKEN` env var is ONLY for cloning the private source repo. Do NOT use it to push or set the remote URL. Do NOT run `git push` at all. Just commit your changes locally and call the `create_pull_request` tool.

Use this clone command (requires `GH_TOKEN`):

  git clone --depth 1 --filter=blob:none --sparse "https://x-access-token:${GH_TOKEN}@github.com/jonorgdev/jon-source-repo.git" <tmp>

Then:

  cd <tmp>
  git sparse-checkout set sync
  rsync -a "<tmp>/sync/" "$GITHUB_WORKSPACE/sync/"
