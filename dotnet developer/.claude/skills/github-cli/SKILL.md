---
name: github-cli
description: "Use GitHub CLI (`gh`) and Git for safe work with remote GitHub repositories. USE FOR: pushing, pulling, fetching, cloning, forking, syncing forks, creating or checking pull requests, issues, releases, GitHub Actions runs, repository metadata, or any task involving a remote GitHub repo. Before doing remote GitHub work, verify `gh` is installed and authenticated; if `gh auth status` fails, stop and ask the user to run `gh auth login`. DO NOT USE FOR: local-only Git history inspection or edits that do not touch GitHub/remotes."
compatibility: "Requires `gh` CLI, Git, and a GitHub repository or GitHub remote. Assumes `gh` is already logged in; stop and request login when it is not."
---

# GitHub CLI

Use this skill for GitHub remote work. Keep actions boring and reversible.

## Safety gate

Before any remote GitHub operation:

1. Check tools and auth:
   ```bash
   command -v gh
   gh auth status
   ```
2. If either command fails, stop. Tell user: `Run gh auth login, then retry.`
3. Check repository context:
   ```bash
   git remote -v
   gh repo view --json nameWithOwner,url,defaultBranchRef
   ```
4. If no GitHub remote exists, ask for owner/repo or clone URL.

Do not push, force-push, delete branches/tags, close issues/PRs, rerun workflows, or publish releases unless user explicitly asked for that action.

## Command choices

Prefer existing tools:

- Local Git state: `git status --short --branch`, `git log`, `git diff`, `git branch`.
- Git transport: `git fetch`, `git pull`, `git push`.
- GitHub objects: `gh pr`, `gh issue`, `gh release`, `gh run`, `gh workflow`, `gh repo`.
- GitHub API gaps: `gh api` with narrow endpoint and fields.

Run `gh auth setup-git` only if Git needs GitHub credentials and current setup fails.

## Push workflow

1. Inspect state:
   ```bash
   git status --short --branch
   git remote -v
   ```
2. Confirm target branch and upstream:
   ```bash
   git branch --show-current
   git rev-parse --abbrev-ref --symbolic-full-name @{u}
   ```
   If upstream missing, use `git push -u origin <branch>` only after target remote/branch is clear.
3. Review outgoing commits when useful:
   ```bash
   git log --oneline @{u}..HEAD
   ```
4. Push with plain `git push`. Avoid `--force`; use `--force-with-lease` only when user explicitly requests rewrite and understands risk.

## Pull or fetch workflow

1. Inspect current branch and dirty worktree:
   ```bash
   git status --short --branch
   ```
2. If worktree has local changes, do not pull blindly. Ask whether to commit, stash, or stop.
3. Fetch safely:
   ```bash
   git fetch --prune
   ```
4. For pull, prefer configured policy. If policy unclear, use fast-forward only:
   ```bash
   git pull --ff-only
   ```
   If fast-forward fails, explain divergence and ask whether to merge or rebase.

## Pull requests

- Check current PR:
  ```bash
  gh pr status
  gh pr view --json number,title,state,url,headRefName,baseRefName,mergeable,reviewDecision,statusCheckRollup
  ```
- Create PR only after branch is pushed:
  ```bash
  gh pr create --fill
  ```
- Use explicit title/body when user provides wording.
- Never merge PR unless user explicitly asks.

## Issues

- Read/search with `gh issue list`, `gh issue view`, or `gh search issues`.
- Create/edit/close only when user asks.
- Quote issue numbers and URLs in final response.

## Releases and tags

- Inspect with `gh release list`, `gh release view`, and `git tag`.
- Creating releases publishes user-visible artifacts. Confirm tag, title, notes, and files unless prompt already states them.
- Never delete or overwrite a release without explicit confirmation.

## Actions

- Inspect workflows/runs with:
  ```bash
  gh workflow list
  gh run list
  gh run view <run-id> --log-failed
  ```
- Rerun/cancel/dispatch workflows only when user asks.

## Output

Report:

- command result summary;
- branch/repo affected;
- URLs for PRs, issues, releases, or runs;
- any stopped action and exact next command user must run.

Keep raw command output short. Include exact errors when relevant.
