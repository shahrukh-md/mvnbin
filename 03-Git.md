# Git & GitHub (Weeks 2 & 3)

**Git** = distributed version-control system (tracks changes locally). **GitHub** = hosting service for remote repositories.

**Areas:** Working directory → (`git add`) → Staging area → (`git commit`) → Local repo → (`git push`) → Remote repo.

---

## A. Setup (do once)

```bash
git --version
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --list
```

---

## B. Task 1 — Push a multi-folder project to a **private** GitHub repo

1. GitHub → **New repository** → name it → tick **Private** → **Create repository**.
2. Open Git Bash in the project's root folder:

```bash
cd /path/to/your/project
git init
git remote add origin https://github.com/<username>/<private-repo>.git
git add .
git commit -m "Initial commit"
git branch -M main
git push -u origin main
```
(If your branch is `master`: `git push -u origin master`. When asked to log in, use your username + **personal access token** as password.)

Verify: `git remote -v`, `git status`, then refresh the GitHub page.

---

## C. Task 2 — Command reference (copy/paste)

### Repository & remotes
```bash
git init                                   # new repo
git clone https://github.com/user/repo.git # copy remote repo
git remote -v                              # list remotes
git remote add origin <url>                # add remote
git remote rename origin upstream          # rename
git remote set-url origin <new-url>        # change URL
git remote show origin                     # details
git remote remove origin                   # remove
git fetch origin                           # download, don't merge
git pull origin main                       # fetch + merge
git push origin main                       # upload
git push --set-upstream origin feature-branch
git remote prune origin                    # clean stale remote-tracking branches
git fetch origin feature-branch            # fetch one branch
```

### Stage & commit
```bash
git status
git add filename.txt        # one file
git add .                   # everything
git commit -m "message"
```

### Branching & merging
```bash
git branch                  # list local
git branch -a               # local + remote
git branch -r               # remote only
git branch new-branch       # create
git checkout new-branch     # switch   (or: git switch new-branch)
git checkout -b new-branch  # create + switch
git branch -d new-branch    # delete (safe, only if merged)
git branch -D new-branch    # force delete
git checkout main
git merge new-branch        # merge into current branch
git fetch origin
git rebase origin/main      # linear history
```

### Undo
```bash
git restore filename                # discard unstaged changes
git restore --staged filename       # unstage (keeps changes)
git reset <file>                    # unstage
git reset --hard                    # DANGER: discard all changes
git revert <commit>                 # new commit that undoes one
git commit --amend -m "new msg"     # fix last commit message
```
> In `git revert`, an editor (vim) opens: press **Esc**, type **`:wq`**, press **Enter** to return.

### History
```bash
git log
git log --oneline
git log --oneline --graph --decorate --all
git diff                    # unstaged changes
git diff --staged           # staged changes
git show <commit>
git blame script.py         # who changed each line
git blame -L 25,25 script.py
git reflog                  # history of HEAD (recover lost commits/branches)
```

### Stash
```bash
git stash          # save work in progress
git stash list
git stash pop      # restore + drop
git stash apply    # restore + keep
```

### .gitignore
```bash
touch .gitignore
```
```
*.log
node_modules/
temp/
*.bak
folder/*
!folder/important_file.txt
```
(If a file is already tracked: `git rm --cached filename`, then commit.)

---

## D. Task 3 — Scenario answers (basic commands)

| # | Answer |
|---|---|
| 1 | Discard unstaged changes: `git restore file.txt` (older: `git checkout -- file.txt`) |
| 2 | Unstage but keep edits: `git restore --staged file1.txt` (or `git reset file1.txt`) |
| 3 | Wrong commit message, not pushed: `git commit --amend -m "Corrected message"` |
| 4 | Readable history: `git log --oneline` (add `--graph --decorate`) |
| 5 | `git config --global user.name "Name"` and `git config --global user.email "mail@x.com"` |
| 6 | See unstaged changes: `git diff` |
| 7 | `git checkout feature/login` (or `git switch feature/login`) |
| 8 | Deleted branch by mistake: `git reflog` → find commit hash → `git checkout -b feature-ui <commit_hash>` |
| 9 | `git push origin <branch>` (e.g. `git push origin main`) |
| 10 | `git fetch origin` |
| 11 | `git checkout -b search-filter` |
| 12 | API key committed: `git filter-branch --force --index-filter "git rm --cached --ignore-unmatch secrets.txt" --prune-empty --tag-name-filter cat -- --all`, then `git push --force --all`. **Also revoke/rotate the key** — it was exposed. (Modern alternatives: `git filter-repo`, BFG.) |
| 13 | `git branch -a` |
| 14 | `git checkout main` → `git merge feature/signup` |
| 15 | Conflict: `git status` → open file → edit between `<<<<<<<`, `=======`, `>>>>>>>` and remove markers → `git add app.js` → `git commit` |
| 16 | Create `.gitignore` containing `*.log` and `node_modules/` |
| 17 | `git blame -L 25,25 script.py` |
| 18 | `git stash` (then switch branch) |
| 19 | `git stash pop` (or `git stash apply`) |
| 20 | `git branch -d feature/test` |
| 21 | `git branch -d feature-ui` (safe: refuses if not merged) |
| 22 | `git branch -D feature-experiment` (force delete) |
| 23 | Make sure you are **not on** the branch you're deleting — switch first (`git checkout main`) |
| 24 | Check: `git branch --merged` / `git branch --no-merged`; then `git branch -d bugfix-footer` (refuses if unmerged) |
| 25 | `git branch -d feature-a feature-b feature-c` |

---

## E. Week 3 — Collaborative coding

### E1. Via an **Organization**
1. GitHub → profile → **Your organizations** → **New organization** → **Create a free organization**.
2. Enter name, contact email, account type → **Next**.
3. **Add members** (invitations are sent; they must accept) → **Complete setup**.
4. Organization **Settings → Member privileges → Base permissions → Write**. Also **Projects base permissions → Write**.
5. Create a repository inside the organization (make it **Private** so only members can see it).
6. All members can now clone, branch, push and see all versions/history.

### E2. Via a **shared repository** (Collaborator 1 owns repo, Collaborator 2 contributes)

**Collaborator 1:** repo → **Settings → Collaborators → Add people** → enter username → invitation sent.

**Collaborator 2:**
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git clone https://github.com/username/repository-name.git
cd repository-name
git checkout -b feature/your-feature-name
# edit files
git add .
git commit -m "Add a clear and descriptive commit message"
git push origin feature/your-feature-name
```
Keep the branch updated:
```bash
git checkout main
git pull origin main
git checkout feature/your-feature-name
git merge main
```

### E3. Fork + Pull Request (contributing to someone else's public repo)
1. Open Person 1's repo → click **Fork** → copy goes to your account.
2. Clone **your fork**, edit, commit, push:
```bash
git clone https://github.com/Person2/awesome-project.git
cd awesome-project
# edit
git add .
git commit -m "Describe change"
git push origin main
```
3. On GitHub: **Contribute → Open pull request** → add title/description → **Create pull request** ("No conflicts with base branch").
4. **Person 1:** Pull requests tab → open it → **Files changed** → **Review changes → Submit review** → **Merge pull request → Confirm merge** ("Pull request successfully merged and closed").

### E4. Merge-conflict exercise
Conflict happens when two people edit the **same line** of the same file on different branches.

1. Collaborator 2: clone, `git checkout feature-abc`, edit README.md line, `git add`, `git commit`, `git push origin feature-abc`.
2. Collaborator 1: edits the **same line** of README.md, commits, pushes → rejected / merge conflict.
3. Resolve:
```bash
git pull                       # conflict reported
git status                     # shows: both modified: README.md
```
4. Open file; it looks like:
```
<<<<<<< HEAD
Hello from First collaborator
=======
Hello from second collaborator
>>>>>>> feature-second
```
5. Keep one or both lines and **delete the three marker lines**.
6. Finish:
```bash
git add README.md
git commit -m "Resolve merge conflict"     # (rebase: git rebase --continue)
git push origin feature-abc
```
Abort if messy: `git merge --abort`.

### E5. Patch create & apply
**Creator**
```bash
git clone <public-project-url>
cd <project>
# edit a file
git add .
git commit -m "commit from cloned person"
git log                                   # copy the commit hash
git format-patch -1 <commit-hash>         # creates 0001-commit-from-cloned-person.patch
```
(`git format-patch -3` = last 3 commits; `git format-patch <base>..HEAD` = a range.)
Send the `.patch` file by e-mail/WhatsApp.

**Owner (receiver)**
```bash
git apply 0001-your-message.patch    # applies changes only (you then commit)
# OR preserve author + message:
git am 0001-your-message.patch
```

---

## F. Scenario answers — remote repository

| # | Scenario | Answer |
|---|---|---|
| 1 | Push rejected "non-fast-forward" | Remote has commits you lack. `git pull --rebase origin <branch>` (or `git pull origin <branch>`), resolve conflicts, then `git push origin <branch>` |
| 2 | Push feature branch without touching main | `git push -u origin feature/x` — pushes only that branch |
| 3 | Stay updated with remote changes/new branches | `git fetch --all --prune` then `git pull`; check `git branch -a` |
| 4 | Pull colleague's changes w/o conflicts when you have local changes | `git stash` → `git pull origin main` → `git stash pop` (or commit first, then `git pull --rebase origin main`) |
| 5 | Pushed API key | Remove from history (`git filter-branch ...` / `git filter-repo` / BFG) → `git push --force --all`; **revoke and rotate the key**; add to `.gitignore` |
| 6 | Integrate latest main into feature | `git checkout main` → `git pull origin main` → `git checkout feature` → `git merge main` (or `git rebase main`) |
| 7 | Push to a different remote | `git remote add newremote <url>` (or `git remote set-url origin <url>`) → `git push newremote <branch>` |
| 8 | Local branch behind remote | Commit/stash local work → `git pull origin <branch>` (fetch + merge) |
| 9 | Conflicts with others' changes | `git pull` → `git status` → edit conflict markers → `git add` → `git commit` → `git push` |
| 10 | Delete remote branch | `git push origin --delete feature-branch` (then `git remote prune origin` on others' clones) |

**Collaborative project scenario (feature/ui-update + hotfix on main + CSS patch by e-mail)**

| # | Answer |
|---|---|
| 11 | `git checkout main` → `git pull origin main` |
| 12 | `git checkout feature/ui-update` → `git merge main` (or `git rebase main`) |
| 13 | `git push origin feature/ui-update`. If rejected: `git pull --rebase origin feature/ui-update` (or merge), resolve conflicts (`git add`, `git rebase --continue`), push again (`git push --force-with-lease` only if you rebased) |
| 14 | `git am fix.patch` — applies the patch **and** records it as a commit with the author's name. (Or `git apply fix.patch` then `git add . && git commit -m "Apply CSS patch"`.) Test, then continue |
| 15 | Test everything, then: `git checkout main` → `git pull origin main` → `git merge feature/ui-update` → `git push origin main` |
