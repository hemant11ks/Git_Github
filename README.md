# Git & GitHub — Practical Guide

One reference for version control, collaboration, code review, and repository automation. Includes beginner setup, daily commands, recovery techniques, and a runnable local practice lab.

**Audience:** developers, DevOps engineers, QA engineers, and anyone managing source code.  
**Requirements:** a modern Git installation; Git 2.28+ for `git init -b`. GitHub operations also require an account and appropriate repository access.  
**Examples:** use `main` as the default branch. Replace `YOUR_USERNAME`, `OWNER`, `REPOSITORY`, paths, and commit IDs with your values. Shell examples target Bash or Git Bash unless stated otherwise.

You can upload this single README to your repository. Code blocks are examples to run individually after reading their prerequisites—not one script to execute from top to bottom.

## Contents

1. [Git versus GitHub](#1-git-versus-github)
2. [Core concepts](#2-core-concepts)
3. [Install and configure Git](#3-install-and-configure-git)
4. [Authenticate with GitHub](#4-authenticate-with-github)
5. [Create or clone a repository](#5-create-or-clone-a-repository)
6. [Daily Git workflow](#6-daily-git-workflow)
7. [Inspect changes and history](#7-inspect-changes-and-history)
8. [Branches and remotes](#8-branches-and-remotes)
9. [Fetch and pull](#9-fetch-and-pull)
10. [Merge and rebase](#10-merge-and-rebase)
11. [Resolve conflicts](#11-resolve-conflicts)
12. [Undo and recover changes](#12-undo-and-recover-changes)
13. [Stash and cherry-pick](#13-stash-and-cherry-pick)
14. [Tags and GitHub releases](#14-tags-and-github-releases)
15. [Ignore files and normalize line endings](#15-ignore-files-and-normalize-line-endings)
16. [GitHub pull requests](#16-github-pull-requests)
17. [Forks and open-source contributions](#17-forks-and-open-source-contributions)
18. [Issues and repository documentation](#18-issues-and-repository-documentation)
19. [Rulesets and security](#19-rulesets-and-security)
20. [GitHub Actions](#20-github-actions)
21. [GitHub CLI](#21-github-cli)
22. [Advanced Git tools](#22-advanced-git-tools)
23. [Local practice lab](#23-local-practice-lab)
24. [Troubleshooting](#24-troubleshooting)
25. [Cheat sheet and interview revision](#25-cheat-sheet-and-interview-revision)
26. [Official references](#26-official-references)

## 1. Git versus GitHub

| Git | GitHub |
|---|---|
| Distributed version-control software | A platform that hosts Git repositories |
| Records commits and manages branches locally | Adds pull requests, issues, permissions, and automation |
| Many operations work offline | Collaboration features generally require network access |
| Installed on your computer or build agent | Accessed through the web, Git, APIs, and CLI |

A **commit** saves a snapshot in your local repository. A **push** updates a remote repository. A **pull request** proposes integrating a branch through GitHub's review workflow. Creating a commit does not automatically upload it.

GitHub is one hosting option. The same core Git concepts apply to GitLab, Bitbucket, Gerrit, and self-hosted Git servers; their review and permission models differ.

## 2. Core concepts

| Term | Meaning |
|---|---|
| Working tree | Checked-out files you edit |
| Index / staging area | Proposed contents of your next commit |
| Repository | Git objects, references, configuration, and history |
| Commit | Snapshot plus metadata and links to parent commits |
| Branch | Movable reference to a commit |
| `HEAD` | Usually a reference to your checked-out branch; sometimes directly to a commit |
| Remote | Named configuration for another repository, commonly `origin` |
| `origin/main` | Local remote-tracking reference updated by fetching |
| Upstream branch | Branch a local branch tracks for default comparisons and integration |
| Tag | Named reference commonly used to identify a release |

`git add` records the selected content in the index. If you edit the file afterward, those additional edits remain unstaged until added again. A commit can therefore contain only part of your working changes.

## 3. Install and configure Git

Ubuntu/Debian:

```bash
sudo apt update
sudo apt install git
```

On Windows, install [Git for Windows](https://gitforwindows.org/) and use Git Bash for these examples. On macOS, use Apple's command-line developer tools or the [Git installation options](https://git-scm.com/install/).

```bash
git --version
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
git config --global init.defaultBranch main
git config --list --show-origin
```

Use an email associated with your GitHub account for attribution, or the exact no-reply address shown in your GitHub email settings. These settings identify commit authors; they do not authenticate you.

Override identity only in the current repository when needed:

```bash
git config user.name "Your Work Name"
git config user.email "your-work-email@example.com"
```

Configuration output can contain sensitive settings. Review it before posting logs publicly.

## 4. Authenticate with GitHub

Choose HTTPS or SSH.

| Method | Example remote | Authentication |
|---|---|---|
| HTTPS | `https://github.com/OWNER/REPOSITORY.git` | Credential manager, GitHub CLI, or an appropriately scoped token |
| SSH | `git@github.com:OWNER/REPOSITORY.git` | Private key locally; corresponding public key registered with GitHub |

GitHub account passwords are not used for Git HTTPS authentication. Prefer a credential manager or interactive CLI login; do not put tokens directly into remote URLs or scripts.

### SSH setup

First inspect existing keys so you do not overwrite one:

```bash
ls -la ~/.ssh
```

For a new key, use a distinct filename if the default already exists. The following Bash example assumes the chosen filename is unused:

```bash
ssh-keygen -t ed25519 -C "your-email@example.com" -f ~/.ssh/id_ed25519_github
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_github
cat ~/.ssh/id_ed25519_github.pub
```

Choose a passphrase. Add the **public** `.pub` content under GitHub account settings → SSH and GPG keys. Keep the private file private.

Add this entry to `~/.ssh/config`, preserving other entries:

```sshconfig
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github
    IdentitiesOnly yes
```

```bash
ssh -T git@github.com
```

Verify a first-connection host fingerprint against [GitHub's published fingerprints](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints). Successful GitHub SSH authentication prints a message explaining that shell access is unavailable; it can return exit status `1` despite successful authentication.

For enterprise organizations, additional SSO authorization or credential policies may apply.

## 5. Create or clone a repository

### Start a new local project

Run in a new project directory:

```bash
mkdir git-learning
cd git-learning
git init -b main
printf '# Git Learning\n' > README.md
git add README.md
git commit -m "docs: add initial README"
```

Create an **empty** repository on GitHub. When uploading existing local history, leave GitHub's README, license, and `.gitignore` initialization options unchecked.

```bash
git remote add origin git@github.com:YOUR_USERNAME/git-learning.git
git push -u origin main
```

The `-u` option establishes upstream tracking. If the GitHub repository already has commits, clone it first and add your files there, or intentionally reconcile the histories. Do not solve this by blindly force-pushing.

### Clone an existing repository

Use this as an alternative to creating the new repository above:

```bash
git clone https://github.com/OWNER/REPOSITORY.git
cd REPOSITORY
git remote -v
git status
```

A clone creates a local repository. It does not grant write permission to the original repository.

## 6. Daily Git workflow

Start from a clean working tree and an up-to-date default branch:

```bash
git switch main
git pull --ff-only
git switch -c feature/update-docs
```

Edit `README.md`, then review and commit:

```bash
git status --short
git diff
git add README.md
git diff --staged
git commit -m "docs: explain local setup"
git push -u origin feature/update-docs
```

Open a pull request on GitHub. Keep each commit focused on one understandable change. Stage specific files or use `git add -p` to review individual hunks; `git add .` can stage unrelated content if you have not inspected the working tree.

After the PR is merged:

```bash
git switch main
git pull --ff-only
git branch -d feature/update-docs
git fetch --prune
```

A squash or rebase merge may make `branch -d` refuse deletion because the original commits are not ancestors of `main`. Verify the PR is merged and no unique work remains before considering forced local branch deletion.

## 7. Inspect changes and history

| Command | What it shows |
|---|---|
| `git status` | Branch and working/index state |
| `git diff` | Unstaged tracked-file changes |
| `git diff --staged` | Staged changes relative to `HEAD` |
| `git diff HEAD` | Tracked changes in the working tree relative to `HEAD` |
| `git log --oneline --graph --decorate --all` | Commit graph and references |
| `git show HEAD` | Latest commit metadata and patch |
| `git log --follow -- README.md` | History of a file, following renames where possible |
| `git blame README.md` | Commits last responsible for lines |
| `git diff main...HEAD` | Changes from the merge base with `main` to `HEAD` |

New untracked files do not appear in ordinary `git diff`; inspect them separately. `blame` is a navigation tool, not a judgment about who caused a defect.

## 8. Branches and remotes

```bash
git branch
git branch -a
git branch -vv
git switch -c fix/login-timeout
git remote -v
git remote show origin
```

A branch is lightweight: Git moves its reference as commits are added. It does not duplicate the whole repository.

If a remote URL changes:

```bash
git remote set-url origin git@github.com:OWNER/REPOSITORY.git
```

`origin` is conventional, not special. A fork workflow often uses `origin` for your fork and `upstream` for the original project.

Remote branch deletion is a shared change. Once a branch is confirmed obsolete and team policy permits deletion:

```bash
git push origin --delete feature/obsolete-branch
```

## 9. Fetch and pull

| Operation | Effect |
|---|---|
| `git fetch origin` | Downloads objects and updates remote-tracking references |
| `git pull --ff-only` | Fetches and advances the current branch only when a fast-forward is possible |
| `git pull --rebase` | Fetches and rebases local commits onto the integrated upstream |
| `git pull --no-rebase` | Fetches and merges according to the applicable merge options |

Fetch does not normally modify your working tree or move your local branches. `origin/main` is not a live network view; it reflects the last relevant fetch.

If fast-forward-only pull fails, inspect divergence first:

```bash
git fetch origin
git log --oneline --left-right HEAD...origin/main
```

Then select merge or rebase based on the branch's ownership and team policy. Commit or stash local edits before integration.

## 10. Merge and rebase

| Merge | Rebase |
|---|---|
| Combines histories; may create a merge commit | Replays commits onto a new base |
| Preserves existing commit identities | Usually creates new commit identities |
| Useful for shared history | Useful for cleaning up a branch you control |

On your feature branch, choose **one** approach after fetching:

```bash
git fetch origin
git merge origin/main
```

Or:

```bash
git fetch origin
git rebase origin/main
```

Do not casually rebase a branch others are building on. If an already-pushed personal branch must be rewritten, coordinate first and use a lease rather than plain `--force`:

```bash
git push --force-with-lease origin feature/update-docs
```

A lease is a conditional protection, not a guarantee that rewriting is appropriate. Its default expected value depends on local remote-tracking state, which background fetches can update. Protected branches may reject force updates entirely.

Interactive rebase, such as `git rebase -i HEAD~3`, can reorder or squash the last three commits when that history exists. Review the selected range carefully and create a backup branch before substantial history editing.

## 11. Resolve conflicts

A conflict means Git needs a decision about how changes should combine. Start with:

```bash
git status
git diff --name-only --diff-filter=U
```

A conflicting text file may contain:

```text
<<<<<<< HEAD
Current side
=======
Incoming side
>>>>>>> feature/example
```

Edit the file to the intended final content, remove the markers, run relevant checks, then stage it:

```bash
git add path/to/conflicted-file
```

Continue the operation shown by `git status`:

| Operation | Continue | Abort |
|---|---|---|
| Merge | `git merge --continue` | `git merge --abort` |
| Rebase | `git rebase --continue` | `git rebase --abort` |
| Cherry-pick | `git cherry-pick --continue` | `git cherry-pick --abort` |

Rebase may stop more than once. Do not use `--skip` unless you intend to omit the current commit. During rebase, the meaning of “ours” and “theirs” can be surprising; review the actual content instead of choosing a side mechanically. Starting with a clean working tree makes abort behavior easier to reason about.

## 12. Undo and recover changes

Choose a command according to what you want to preserve.

| Goal | Command | Consequence |
|---|---|---|
| Unstage a file | `git restore --staged README.md` | Keeps working-tree edits |
| Discard unstaged edits in a tracked file | `git restore README.md` | Replaces working content from the index; those edits are lost |
| Undo a published ordinary commit | `git revert COMMIT_ID` | Creates a new reversing commit |
| Amend the latest local commit | `git commit --amend` | Includes current staged changes and rewrites the commit |
| Undo latest local commit, keep staged content | `git reset --soft HEAD~1` | Moves branch back; preserves index and working tree |
| Undo latest local commit, keep unstaged content | `git reset --mixed HEAD~1` | Moves branch and resets index; preserves working tree |

`HEAD~1` requires a parent commit. Reverting a merge requires deliberate parent selection and is more complex than reverting an ordinary commit.

**Destructive operations:** `git reset --hard` discards tracked changes and can remove untracked content obstructing checkout. `git clean -fd` removes untracked files/directories; adding `-x` includes ignored files. Preview with `git clean -nd` and back up valuable files before any removal. A branch backup preserves committed history, not uncommitted files.

To investigate a lost commit:

```bash
git reflog
git show COMMIT_ID
git branch recovery/lost-work COMMIT_ID
```

Reflogs are local and expire. They can help find previously referenced commits; they are not a reliable recovery mechanism for content never stored in Git.

## 13. Stash and cherry-pick

Temporarily save tracked changes plus untracked files:

```bash
git stash push -u -m "WIP before switching tasks"
git stash list
git stash show --stat 'stash@{0}'
git stash apply 'stash@{0}'
```

After confirming the restored work is correct, remove that stash entry:

```bash
git stash drop 'stash@{0}'
```

`apply` retains the entry; `pop` attempts to apply and remove it after success. Ignored files are not included with `-u`. Stashes are local and are not pushed with your branches.

Apply one existing commit's changes onto the current branch:

```bash
git cherry-pick COMMIT_ID
```

Cherry-pick generally produces a new commit ID. Use it for a targeted backport when appropriate; repeatedly copying changes between long-lived branches complicates history.

## 14. Tags and GitHub releases

Create an annotated tag at the current commit after validation:

```bash
git tag -a v1.0.0 -m "Release 1.0.0"
git show v1.0.0
git push origin v1.0.0
```

A normal branch push does not necessarily publish all tags; push the intended tag explicitly. Tags identify source versions. A GitHub Release adds release notes and optional downloadable assets around a tag.

Treat published release tags as stable. Correct a faulty release through a documented new version rather than silently moving a tag consumers already use.

## 15. Ignore files and normalize line endings

Example `.gitignore` for a Python-oriented repository:

```gitignore
.venv/
__pycache__/
*.py[cod]
.env
.env.*
!.env.example
build/
dist/
*.log
```

Keep `.env.example` free of real secrets. Ignore rules apply to untracked files; they do not automatically stop tracking existing files. To stop tracking an already-committed local configuration while retaining your local copy:

```bash
git rm --cached .env
git add .gitignore
git commit -m "chore: stop tracking local environment file"
```

This does not remove old copies from history. Rotate exposed credentials first if a secret was committed.

Example `.gitattributes`:

```gitattributes
* text=auto
*.sh text eol=lf
*.bat text eol=crlf
*.png binary
```

When introducing a line-ending policy into an existing repository, use a clean branch, run `git add --renormalize .`, and review the staged diff before committing.

## 16. GitHub pull requests

1. Push a focused branch.
2. Open a PR with the correct **base** branch and **compare** branch.
3. Explain the problem, change, validation, and any rollout concerns.
4. Review the Files changed tab for accidental files or secrets.
5. Request reviewers and address feedback through additional commits.
6. Wait for the configured checks and approvals, then use the permitted merge method.
7. Update your local default branch after merging.

A draft PR communicates work in progress. Opening a PR does not automatically merge anything.

| GitHub merge method | Result |
|---|---|
| Merge commit | Preserves branch commits and records a merge |
| Squash and merge | Combines PR changes into one new commit on the base |
| Rebase and merge | Adds individual commits with rewritten identities onto the base |

Template for `.github/pull_request_template.md`:

```markdown
## Why
Describe the problem and its impact.

## Changes
Describe what changes for users or maintainers.

## Validation
List the checks performed and their results.

## Related work
Link relevant issues or documentation.
```

Use `Closes #123` only when the PR should close that issue under GitHub's linking rules.

## 17. Forks and open-source contributions

A fork is a repository hosted under another account or organization. A clone is a local copy. You often fork first when you lack write access to the original repository.

After creating a fork on GitHub:

```bash
git clone git@github.com:YOUR_USERNAME/REPOSITORY.git
cd REPOSITORY
git remote add upstream https://github.com/ORIGINAL_OWNER/REPOSITORY.git
git fetch upstream
git switch main
git merge --ff-only upstream/main
git push origin main
git switch -c fix/documentation
```

Commit and push your feature branch, then open a PR targeting the original project's intended branch. Read its contribution policy first. If your fork's `main` has diverged, inspect the commits before deciding how to synchronize it.

## 18. Issues and repository documentation

| Feature or file | Purpose |
|---|---|
| Issues | Track defects, proposals, and actionable work |
| Labels and milestones | Classify work and group delivery targets |
| Projects | Organize work across views and repositories |
| Discussions | Questions and longer conversations where enabled |
| `README.md` | Purpose, setup, usage, and navigation |
| `CONTRIBUTING.md` | Contribution and testing expectations |
| `LICENSE` | Explicit reuse terms chosen for the project |
| `SECURITY.md` | Private vulnerability-reporting instructions |
| `.github/CODEOWNERS` | Associate paths with responsible reviewers |

A public repository is not automatically licensed for unrestricted reuse. Choose a suitable license intentionally. Use issue forms or templates for reproducible bug reports: expected behavior, actual behavior, minimal reproduction, and environment.

## 19. Rulesets and security

GitHub rulesets or branch protection can enforce review and integration requirements. Availability and individual controls depend on plan, repository visibility, and organization configuration.

For important branches, evaluate requiring PRs, relevant status checks, approval, and restrictions on deletion or force pushes. Limit bypass permissions. Merely creating a workflow does not make its result a merge requirement; configure the corresponding required check.

Use narrowly scoped tokens and separate machine identities where appropriate. Enable suitable secret scanning and dependency security features available to the repository. If a credential leaks, revoke or rotate it immediately; deleting the newest file does not invalidate the credential or erase prior copies.

For CI, avoid exposing privileged credentials or trusted self-hosted runners to untrusted code. Fork PRs and external contributions require a deliberate trust boundary.

## 20. GitHub Actions

A workflow is a YAML file under `.github/workflows/`. Events trigger workflows, workflows contain jobs, jobs run on runners, and steps execute shell commands or reusable actions.

This example works with a README-only repository. Save as `.github/workflows/docs.yml`:

```yaml
name: Documentation check

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  docs:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    steps:
      - name: Check out source
        uses: actions/checkout@v7
        with:
          persist-credentials: false
      - name: Verify README exists and is nonempty
        shell: bash
        run: test -s README.md
```

This checks file presence, not Markdown quality or application correctness. Extend it with meaningful tests for your project. The checkout version shown was checked against the official action documentation when this guide was prepared; review runner compatibility when using self-hosted runners.

Major version tags are readable but mutable. For stricter supply-chain controls, pin actions to reviewed full commit SHAs and maintain an update process. Checkout defaults to limited history; configure `fetch-depth: 0` when a job actually requires full history or tags.

Store sensitive values using GitHub's supported secrets and environment mechanisms. Keep workflow token permissions minimal. Never execute untrusted PR code with privileged secrets through `pull_request_target` merely to bypass fork restrictions.

## 21. GitHub CLI

The optional [GitHub CLI](https://cli.github.com/) uses the `gh` command and has its own installation procedure.

```bash
gh auth login
gh auth status
gh repo view
gh pr list
gh pr create --base main --head feature/update-docs --title "Improve setup documentation" --body "Explains local setup and verification."
gh pr checks
gh run list
gh issue list
```

Run repository commands inside the relevant clone or specify the repository explicitly. `gh pr create` publishes a PR; review its title, body, and target before running it. The CLI complements Git; it does not replace local commits and branches.

## 22. Advanced Git tools

| Tool | Example | When useful |
|---|---|---|
| Worktree | `git worktree add -b fix/urgent ../urgent-fix main` | Check out a second branch without disturbing current work |
| Bisect | `git bisect start` | Find the commit that introduced a reproducible regression |
| Search | `git grep -n 'TODO'` | Search tracked working-tree content |
| Patch export | `git format-patch -1 HEAD` | Export a commit for a patch-based workflow |
| Patch import | `git am change.patch` | Apply a mail-format patch as a commit |
| Submodule | `git submodule update --init --recursive` | Populate dependencies recorded as nested repositories |
| Sparse checkout | `git sparse-checkout set docs scripts` | Limit populated directories in an initialized sparse checkout |
| Git LFS | `git lfs track '*.psd'` | Track selected large file types after installing Git LFS |

For bisect, mark a known bad revision and a known good revision, test each selected checkout, then run `git bisect reset` afterward. Avoid testing with unrelated working-tree changes.

For a sparse clone, start with `git clone --filter=blob:none --sparse URL`; partial-clone support depends on the server. Git LFS tracking creates attributes but does not automatically migrate all historical large files. Commit `.gitattributes` and plan any history migration separately.

Remove a finished worktree with `git worktree remove ../urgent-fix` once its changes are handled. Submodule commits must be accessible to collaborators; committing only the parent pointer does not publish the nested repository's commit.

## 23. Local practice lab

This complete Bash lab uses a temporary directory and a local bare remote. It needs Git, Bash, and `mktemp`; no GitHub account or network is required. It leaves its directory in place for inspection.

```bash
set -eu
lab_dir=$(mktemp -d)
printf 'Practice directory: %s\n' "$lab_dir"

git init --bare -b main "$lab_dir/remote.git"
git clone "$lab_dir/remote.git" "$lab_dir/work"
cd "$lab_dir/work"
git config user.name "Practice User"
git config user.email "practice@example.invalid"

printf '# Practice\n' > README.md
git add README.md
git commit -m "docs: initialize practice repository"
git push -u origin main

git switch -c feature/add-guide
printf '\nA short Git guide.\n' >> README.md
git add README.md
git commit -m "docs: add guide description"
git push -u origin feature/add-guide

git switch main
git merge --no-ff feature/add-guide -m "Merge documentation guide"
git push origin main

git tag -a v1.0.0 -m "Practice release"
git push origin v1.0.0

git clone "$lab_dir/remote.git" "$lab_dir/check"
test -s "$lab_dir/check/README.md"
git -C "$lab_dir/check" log --oneline --graph --all
git -C "$lab_dir/check" tag --list
printf 'Lab completed in %s\n' "$lab_dir"
```

Expected outcome: the remote has `main`, the feature branch, and `v1.0.0`; the second clone contains the merged README. Next, practice conflicting edits in two branches and use section 11 to resolve them. Run destructive experiments only in this disposable lab.

## 24. Troubleshooting

| Symptom | Likely explanation | Next step |
|---|---|---|
| `Permission denied (publickey)` | Wrong key, missing registration, or account restrictions | Check `ssh -T`, selected key, account access, and SSO requirements |
| HTTPS authentication failed | Invalid credential or insufficient token access | Reauthenticate using the credential manager or `gh auth login` |
| `remote origin already exists` | Remote already configured | Inspect `git remote -v`; use `remote set-url` if appropriate |
| Non-fast-forward push rejected | Remote contains work absent from your branch | Fetch, inspect, and integrate before pushing |
| `src refspec main does not match any` | No commit yet or wrong branch name | Check `git status` and `git branch` |
| Local changes would be overwritten | Checkout or integration conflicts with edits | Commit, stash, or deliberately discard the relevant changes |
| Detached HEAD | Checked out a commit/tag directly | Create a branch with `git switch -c recovery/work` to retain new work |
| `.gitignore` appears ineffective | File is already tracked | Inspect `git ls-files`; remove from index only if intended |
| `.git/index.lock` exists | Active Git process or stale lock after failure | Confirm no Git/editor process owns the operation before removing a stale lock |
| Certificate validation fails | Proxy CA or hostname/certificate problem | Fix the trust chain or endpoint; keep TLS verification enabled |
| Repository not found | Wrong URL or unavailable private repository | Verify spelling, account, and access |
| Ruleset rejects push | Required review/check or restricted action | Follow the repository's configured workflow |

Do not delete `.git` to fix a routine error; it contains repository history and configuration. Diagnose the specific operation first.

## 25. Cheat sheet and interview revision

| Need | Command |
|---|---|
| Current state | `git status` |
| Create branch | `git switch -c feature/name` |
| Stage selected hunks | `git add -p` |
| Commit | `git commit -m "Describe the change"` |
| Update remote knowledge | `git fetch --prune` |
| Integrate only a fast-forward | `git pull --ff-only` |
| First branch push | `git push -u origin feature/name` |
| Inspect graph | `git log --graph --oneline --decorate --all` |
| Unstage file | `git restore --staged FILE` |
| Reverse published change | `git revert COMMIT_ID` |
| Find previous branch positions | `git reflog` |
| List linked working trees | `git worktree list` |

**Common interview questions**

- **Git and GitHub?** Git manages version history; GitHub hosts repositories and collaboration services.
- **Fetch and pull?** Fetch updates remote-tracking data; pull also integrates into the current branch.
- **Merge and rebase?** Merge combines ancestry; rebase replays commits and changes their identities.
- **Reset and revert?** Reset can move local history or reset the index; revert records a new inverse change.
- **Fork and clone?** Fork creates a hosted repository relationship; clone creates a local repository.
- **What is staging?** It selects the exact content for the next commit.
- **Can Git recover every deleted file?** No. Recovery depends on whether content was stored and remains reachable or otherwise retained.
- **Why do checks pass but merging remain blocked?** Review requirements, branch freshness, rulesets, or a differently named required check may still be unsatisfied.

## 26. Official references

- [Git documentation](https://git-scm.com/docs) and [Pro Git book](https://git-scm.com/book/en/v2)
- [Restore](https://git-scm.com/docs/git-restore), [reset](https://git-scm.com/docs/git-reset), and [revert](https://git-scm.com/docs/git-revert)
- [Rebase](https://git-scm.com/docs/git-rebase) and [push](https://git-scm.com/docs/git-push)
- [GitHub documentation](https://docs.github.com/)
- [SSH authentication](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [GitHub rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)
- [GitHub Actions quickstart](https://docs.github.com/en/actions/get-started/quickstart)
- [Official checkout action](https://github.com/actions/checkout)
- [GitHub CLI manual](https://cli.github.com/manual/)
- [Git LFS](https://git-lfs.com/)

Maintain this guide alongside your team's contribution policy. Commands that rewrite history, discard files, or modify remote branches require an understanding of the repository's current state.
