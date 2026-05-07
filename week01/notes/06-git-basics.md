# Git Basics

## Core Concepts

- **Repository (repo)** — a directory tracked by git, containing your files + full history
- **Commit** — a snapshot of all tracked files at a point in time
- **Branch** — a named pointer to a commit; lets you work in parallel without breaking `main`
- **Remote** — a copy of the repo on a server (GitHub). `origin` is the conventional name for your main remote
- **Working tree** — your actual files on disk
- **Staging area (index)** — files you've marked to include in the next commit (`git add`)

---

## Daily Workflow

```bash
# Check what's changed
git status

# Stage specific files
git add src/main.c
git add week01/amogh/     # stage a whole folder

# Stage everything (use with care)
git add .

# Commit with a message
git commit -m "week01: add bit manipulation library"

# Push to GitHub
git push

# Pull latest changes from teammates
git pull
```

---

## Setting Up

```bash
# First time on a new machine
git config --global user.name "Amogh"
git config --global user.email "amogh@example.com"

# Clone the shared repo
git clone git@github.com:Majorwhiskey/embedded-engineering-roadmap.git
cd embedded-engineering-roadmap
```

---

## Branching

Never commit directly to `main` for anything beyond trivial fixes. Use a branch.

```bash
# Create and switch to a new branch
git checkout -b week01-amogh-memory-playground

# Or with newer git:
git switch -c week01-amogh-memory-playground

# List all branches
git branch -a

# Switch between branches
git switch main
git switch week01-amogh-memory-playground

# Push branch to GitHub (first time)
git push -u origin week01-amogh-memory-playground

# Subsequent pushes
git push
```

---

## Pull Requests (PR Workflow)

1. Create a branch locally
2. Commit your work to that branch
3. Push the branch to GitHub
4. Open a PR on GitHub: your branch → `main`
5. Teammates review the code
6. Merge the PR into `main`
7. Delete the branch

```bash
# After PR is merged, clean up locally
git switch main
git pull                                         # get the merged changes
git branch -d week01-amogh-memory-playground    # delete local branch
```

---

## Merging & Conflicts

```bash
# Merge another branch into your current branch
git merge main    # bring main's changes into your branch

# If there's a conflict, git marks the file:
<<<<<<< HEAD
your version of the code
=======
teammate's version of the code
>>>>>>> main

# Edit the file to resolve, then:
git add conflicted-file.c
git commit    # completes the merge
```

---

## Useful Commands

```bash
# See commit history
git log
git log --oneline --graph    # compact visual tree

# See what changed in a file
git diff src/main.c

# See what's staged
git diff --staged

# Undo changes to a file (discard edits, go back to last commit)
git restore src/main.c

# Unstage a file (keep the edits, just don't commit it)
git restore --staged src/main.c

# See a previous version of a file
git show HEAD~2:src/main.c    # 2 commits ago

# Stash work in progress (temporarily set aside changes)
git stash
git stash pop      # bring them back
```

---

## Commit Message Convention (This Repo)

Format: `weekNN: short description`

```
week01: add bit manipulation library
week01: fix memory leak in linked list
week03: bare-metal PWM servo controller
week06: Wireshark capture annotations
```

Keep it under 72 characters. Use present tense ("add", "fix", "implement" — not "added", "fixed").

---

## .gitignore

Already set up in this repo. Prevents build artifacts from being committed:

```
*.o         ← compiled object files
*.elf       ← linked executable
*.bin       ← binary firmware
*.hex       ← Intel hex firmware
*.map       ← linker map file
build/      ← build output directory
```

Never commit binaries — only source code.

---

## SSH vs HTTPS

This repo uses SSH. Verify your key is set up:

```bash
ssh -T git@github.com
# Should print: Hi Majorwhiskey! You've successfully authenticated...
```

If it fails:
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
cat ~/.ssh/id_ed25519.pub    # copy this
# Paste into GitHub → Settings → SSH and GPG keys → New SSH key
```
