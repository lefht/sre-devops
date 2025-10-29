# Git — Beginner → Advanced / Master

Comprehensive guide to Git: concepts, commands, workflows, internals, recovery, and best practices with examples.

---

## Table of contents
- Overview & concepts
- Install & configure
- Basic daily workflow
- Inspecting history & diffs
- Branching, merging, and rebasing
- Collaboration: remotes, fetch, pull, push, PRs
- Undoing, recovery & history edits (reset/revert/reflog)
- Stashing, worktrees, sparse-checkout, LFS
- Submodules & subtrees
- Hooks, automation & CI integration
- Advanced commands & plumbing
- Internals: objects, refs, index, packs
- Performance, maintenance & safety
- Security: signing commits & tags
- Workflows & best practices
- Examples appendix (copy-paste snippets)

---

## Overview & concepts
- Git is a content-addressable snapshot-based version control system.
- Key concepts:
  - Commit: a snapshot of the project tree, metadata and parent(s).
  - Branch: a movable reference (ref) pointing to a commit.
  - HEAD: the currently checked-out ref (or detached commit).
  - Index (staging area): prepared files for the next commit.
  - Remote: named URL targets (origin, upstream).
- Basic model: edit files → git add (stage) → git commit (snapshot).

---

## Install & configure
- Install:
  - Debian/Ubuntu: `sudo apt install git`
  - macOS: `brew install git` or Xcode CLI tools
  - Windows: Git for Windows installer
- Set identity and helpful defaults:
  ```bash
  git config --global user.name "Your Name"
  git config --global user.email you@example.com
  git config --global core.editor "nano"
  git config --global pull.rebase false   # or true to prefer rebase
  git config --global credential.helper cache
  ```
- Useful alias examples:
  ```bash
  git config --global alias.st status
  git config --global alias.co checkout
  git config --global alias.br branch
  git config --global alias.lg "log --oneline --graph --all --decorate"
  ```

---

## Basic daily workflow
- Start a repo:
  ```bash
  git init            # create an empty repo
  git clone url.git   # clone an existing repo
  ```
- Status, stage, commit:
  ```bash
  git status
  git add README.md
  git commit -m "docs: add README"
  ```
- Amend last commit (if not pushed):
  ```bash
  git add changed.file
  git commit --amend --no-edit   # replace last commit with new staged content
  ```
- Good commit message pattern: `type(scope): short summary` and optional longer body.

---

## Inspecting history & diffs
- Show commit history:
  ```bash
  git log --oneline --graph --decorate --all
  git lg   # if alias set
  ```
- View diff:
  ```bash
  git diff              # unstaged changes
  git diff --staged     # staged changes for next commit
  git show HEAD~1       # diff of a commit
  ```
- Blame / annotate:
  ```bash
  git blame --line-porcelain file | less
  ```

---

## Branching, merging, and rebasing
- Create and switch:
  ```bash
  git branch feature/xyz
  git switch feature/xyz      # modern command (or git checkout)
  ```
- Merge:
  ```bash
  git switch main
  git merge --no-ff feature/xyz
  ```
  - Use `--no-ff` for explicit merge commits; omit for linear history.
- Rebase (rewrite commit base):
  ```bash
  git switch feature/xyz
  git rebase main
  ```
  - Interactive rebase to reorder/squash/edit commits:
    ```bash
    git rebase -i main
    # change pick → squash/fixup/edit/reword as needed
    ```
  - Rebase vs merge:
    - Rebase: linear history, rewrites commits (use on private branches).
    - Merge: preserves true topology, safe for public branches.

---

## Collaboration: remotes, fetch, pull, push, PRs
- Add remote, fetch, push:
  ```bash
  git remote add origin git@github.com:org/repo.git
  git fetch origin
  git push -u origin feature/xyz
  ```
- Pull vs fetch:
  - `git fetch` updates remote-tracking branches.
  - `git pull` is `fetch` + `merge` (or `fetch` + `rebase` if configured).
- Common safety: prefer `git pull --rebase` for feature branches to avoid merge commits locally.
- Creating PRs:
  - Push branch: `git push origin feature/xyz`
  - Open PR on hosting platform (GitHub/GitLab) and add reviewers.
- Upstream syncing (if forked):
  ```bash
  git remote add upstream git@github.com:original/repo.git
  git fetch upstream
  git switch main
  git merge upstream/main
  git push origin main
  ```

---

## Undoing, recovery & history edits
- Revert a commit (create a new commit that undoes changes):
  ```bash
  git revert <commit>
  ```
- Reset (move branch pointer):
  - Soft: keep changes staged
    ```bash
    git reset --soft HEAD~1
    ```
  - Mixed (default): keep changes unstaged
    ```bash
    git reset HEAD~1
    ```
  - Hard: discard working tree changes (dangerous)
    ```bash
    git reset --hard HEAD~1
    ```
- Recover lost commits with reflog:
  ```bash
  git reflog
  git switch -c recover <reflog-ref>
  ```
- Cherry-pick a commit onto current branch:
  ```bash
  git cherry-pick <commit>
  ```
- Interactive rebase to edit history (only on branches not shared/pushed, or coordinate with others).

---

## Stashing, worktrees, sparse-checkout, LFS
- Stash local changes:
  ```bash
  git stash push -m "WIP: fix tests"
  git stash list
  git stash pop  # or apply then drop
  ```
- Worktrees (multiple checkouts of same repo):
  ```bash
  git worktree add ../repo-feature feature/xyz
  ```
- Sparse-checkout (partial checkout large repos):
  ```bash
  git sparse-checkout init --cone
  git sparse-checkout set src/docs
  ```
- Git LFS for large binaries:
  ```bash
  git lfs install
  git lfs track "*.iso"
  git add .gitattributes && git commit -m "track ISO via LFS"
  ```

---

## Submodules & subtrees
- Submodule basics:
  ```bash
  git submodule add git@github.com:org/lib.git vendor/lib
  git submodule update --init --recursive
  ```
- Update submodule to a specific commit and commit the change in superproject.
- Subtrees (alternative) allow embedding history:
  ```bash
  git subtree add --prefix=vendor/lib git@github.com:org/lib.git main --squash
  ```
- Choose submodules when you need a separate repo; choose subtrees when simpler embedding is desired.

---

## Hooks, automation & CI integration
- Hooks live in `.git/hooks` (client-side) and can be enabled by making them executable.
- Examples:
  - pre-commit: linting tests before committing.
  - pre-push: run quick tests.
- Example `pre-commit` (shell):
  ```bash
  #!/usr/bin/env bash
  set -e
  black --check .
  flake8
  ```
- Use CI (GitHub Actions / GitLab CI) for server-side checks; keep hooks light-weight and fast.

---

## Advanced commands & plumbing
- Useful plumbing:
  - `git rev-parse`, `git cat-file -p`, `git ls-tree`, `git update-index`
- Show object content:
  ```bash
  git cat-file -p <sha1>
  ```
- Low-level commit creation (rarely needed):
  ```bash
  git hash-object -w file
  git update-index --add --cacheinfo <mode>,<sha>,path
  git write-tree
  git commit-tree <tree> -p <parent> -m "message"
  git update-ref refs/heads/branch <new-commit>
  ```

---

## Internals: objects, refs, index, packs
- Objects: blob (file), tree (directory), commit, tag — all stored by SHA-1/256 depending on Git version.
- Refs: refs/heads/* (branches), refs/tags/*, refs/remotes/*.
- Index: staging area storing file metadata and blob associations.
- Packs: Git repacks objects to packfiles for efficiency; `git gc` runs maintenance.
- Inspect:
  ```bash
  git show-ref
  git ls-files --stage
  git rev-parse HEAD
  ```

---

## Performance & maintenance
- Repack & garbage collect:
  ```bash
  git gc --aggressive --prune=now
  ```
- Check repo health:
  ```bash
  git fsck --full
  ```
- Shallow clone to reduce download:
  ```bash
  git clone --depth 1 git@github.com:org/repo.git
  ```
- Partial clone for large monorepos:
  ```bash
  git clone --filter=blob:none --no-checkout url
  ```

---

## Security: signing commits & tags
- Generate GPG key, then configure:
  ```bash
  git config --global user.signingkey <GPG_KEY_ID>
  git config --global commit.gpgsign true
  git config --global tag.gpgSign true
  ```
- Create signed tag:
  ```bash
  git tag -s v1.2.3 -m "release v1.2.3"
  git push origin v1.2.3
  ```

---

## Workflows & best practices
- Prefer small, focused commits with clear messages.
- Use feature branches for work; keep main branch stable.
- Use pull requests with CI checks and code review.
- Rebase private branches to keep history clean; avoid rewriting public history.
- Protect main branch (branch protection rules on hosting).
- Use semantic versioning and signed tags for releases.

---

## Troubleshooting scenarios (practical)
- Recover accidentally deleted branch:
  ```bash
  git reflog
  git branch recovered <reflog-sha>
  ```
- Undo a pushed commit safely:
  - Non-destructive: `git revert <bad-commit>`
  - Force push only when coordinated: `git reset --hard <good-commit> && git push --force-with-lease`
- Resolve merge conflicts:
  ```bash
  # edit files to resolve
  git add resolved.file
  git rebase --continue   # or git commit (for merge)
  ```
- Bisect to find a bad commit:
  ```bash
  git bisect start
  git bisect bad          # current is bad
  git bisect good v1.2.0  # known good
  # run tests, mark good/bad until commit found
  git bisect reset
  ```

---

## Examples appendix (compact copy-paste snippets)

- Initialize repo and make first commit:
  ```bash
  git init
  git add .
  git commit -m "chore: initial commit"
  ```
- Create feature, work, PR:
  ```bash
  git switch -c feature/auth
  # work...
  git add .
  git commit -m "feat(auth): add login handler"
  git push -u origin feature/auth
  # open PR on hosting site
  ```
- Interactive rebase to squash:
  ```bash
  git switch feature/auth
  git rebase -i main
  # change 'pick' to 's' for squash, save and exit
  git push --force-with-lease origin feature/auth
  ```
- Safe force-push pattern:
  ```bash
  git fetch origin
  git rebase origin/main
  git push --force-with-lease
  ```
- Create signed release tag:
  ```bash
  git tag -s v2.0.0 -m "release v2.0.0"
  git push origin v2.0.0
  ```

---

## Further reading & resources
- Official book: Pro Git (https://git-scm.com/book/en/v2)
- Git manpages: `man git`, `man git-rebase`, `man git-commit`
- Advanced internals: "Git from the bottom up" and `git internals` articles
- Hosting docs: GitHub, GitLab, Bitbucket guides for PRs, branching, and CI.

---

End of guide.
