# Day 6 — Git Basics
**Date: May 12, 2026**

Topics: init, clone, add, commit, branch, merge, push, pull, PR workflow, conflict resolution

---

## What Git Does

Git takes **snapshots** of your project at points you choose (commits). Every commit stores:
- The full state of every tracked file
- Who made the change (name + email)
- When it happened
- A message describing why
- A pointer to the previous commit

This gives you a complete, permanent history. You can jump back to any past state, branch off in a different direction, and merge changes from teammates — all without fear of losing work.

---

## Key Concepts

| Concept | What it is |
|---------|-----------|
| **Repository** | The `.git/` folder — stores entire history and metadata |
| **Working tree** | Your actual files on disk |
| **Staging area (index)** | Files queued for the next commit (`git add`) |
| **Commit** | A saved snapshot with author, timestamp, message |
| **Branch** | A named pointer that moves forward with each commit |
| **HEAD** | Points to the commit you're currently on |
| **Remote** | A copy of the repo on another machine (GitHub = `origin`) |
| **Clone** | Download a remote repo to your machine |
| **Push** | Upload your local commits to the remote |
| **Pull** | Download remote commits and merge into your local branch |

---

## Initial Setup (one-time per machine)

```bash
# Tell git who you are — embedded in every commit you make
git config --global user.name  "Vaishnavi"
git config --global user.email "vaishnavi@example.com"

# Set default branch name to main
git config --global init.defaultBranch main

# Set VS Code as the default editor (for commit messages)
git config --global core.editor "code --wait"

# Verify
git config --list
```

---

## SSH Key Setup (one-time per machine)

```bash
# Generate a key pair
ssh-keygen -t ed25519 -C "vaishnavi@example.com"
# Accept default location (~/.ssh/id_ed25519)
# Set a passphrase (optional but recommended)

# Print the public key — copy this entire line
cat ~/.ssh/id_ed25519.pub

# Add to GitHub: Settings → SSH and GPG keys → New SSH key → paste

# Test
ssh -T git@github.com
# Expected: Hi Vaishnavi! You've successfully authenticated...
```

Why SSH instead of HTTPS: with SSH you don't type a password on every push. The private key authenticates you automatically.

---

## Clone the Shared Repo

```bash
git clone git@github.com:Majorwhiskey/embedded-engineering-roadmap.git
cd embedded-engineering-roadmap

# Confirm it worked
git log --oneline    # see existing commits
git branch -a        # see all branches
```

---

## The Three-Stage Model

Every change in git passes through three stages:

```
Working Tree  →  Staging Area  →  Repository (committed)
  (edit)          (git add)         (git commit)
```

```bash
# Edit a file
echo "# My notes" > week01/vaishnavi/notes.md

# Check what's changed
git status            # shows modified/untracked files
git diff              # shows line-by-line changes (unstaged)
git diff --staged     # shows what's staged (ready to commit)

# Stage the file
git add week01/vaishnavi/notes.md

# Or stage a whole folder
git add week01/vaishnavi/

# Commit with a message
git commit -m "week01: add initial notes"

# Push to GitHub
git push
```

---

## Commit Messages — This Repo's Convention

Format: `weekNN: short description`

```bash
# Good
git commit -m "week01: add bit manipulation library"
git commit -m "week01: fix null pointer bug in linked list"
git commit -m "week03: bare-metal PWM servo controller"

# Bad — too vague
git commit -m "update"
git commit -m "fix"
git commit -m "asdf"
```

Rules:
- Present tense: "add" not "added"
- Under 72 characters
- Describe WHY if not obvious (use multi-line commits for this)

Multi-line commit:
```bash
git commit -m "week01: fix memory leak in stack implementation

The pop() function was not freeing the node after returning the value.
Valgrind reported 24 bytes leaked per pop() call.
Fixed by calling free(node) before return."
```

---

## Branching

Never commit directly to `main` for feature work. Create a branch, work on it, then merge via PR.

```bash
# Create and switch to a new branch
git checkout -b week01-vaishnavi-bitlib
# (equivalent: git switch -c week01-vaishnavi-bitlib)

# Verify you're on the new branch
git branch         # * marks current branch

# Do your work, then stage and commit as normal
git add .
git commit -m "week01: add bit manipulation library"

# Push branch to GitHub
git push -u origin week01-vaishnavi-bitlib
# -u sets the upstream tracking — subsequent pushes just need: git push
```

### What branches look like

```
main:          A ── B ── C ── ── ── ── G
                           \          /
your-branch:                D ── E ── F
```

`main` doesn't know about D, E, F until you merge.

---

## Pull Requests (PR Workflow)

A pull request is a request to merge your branch into `main`. It creates a space for code review before merging.

**Step by step:**

1. Push your branch to GitHub
2. Go to https://github.com/Majorwhiskey/embedded-engineering-roadmap
3. GitHub shows a banner: **"Compare & pull request"** — click it
4. Set base: `main`, compare: your branch
5. Write a title and description
6. Submit — teammates get notified
7. Reviewer reads the diff, leaves comments
8. Make any requested changes, push again to the same branch
9. Reviewer approves → **Merge pull request**
10. Delete the branch (GitHub offers this after merge)

**Clean up locally after merge:**
```bash
git switch main
git pull                                           # get the merged commit
git branch -d week01-vaishnavi-bitlib              # delete local branch
```

---

## Pulling and Syncing

```bash
# Get latest changes from GitHub (fetch + merge)
git pull

# Safer: fetch first, review, then merge
git fetch origin
git log main..origin/main --oneline   # see what's new on remote
git merge origin/main                 # merge when ready

# Always pull before starting work each session
git pull
# Then create your branch
git checkout -b weekNN-yourname-feature
```

---

## Resolving Merge Conflicts

A conflict occurs when two people edited the same lines. Git can't decide which version to keep — it asks you.

```bash
git pull    # triggers conflict

# Git marks the conflicted file:
<<<<<<< HEAD
your version of the code
=======
teammate's version of the code
>>>>>>> origin/main

# Resolve: edit the file to keep the correct version
# Remove ALL the conflict markers (<<<, ===, >>>)

git add conflicted-file.c
git commit    # completes the merge (message is pre-filled)
git push
```

**Tip:** conflicts are rare if everyone sticks to their own subfolder. They only happen when two people edit the same file (like `README.md`).

---

## Useful Daily Commands

```bash
# See history
git log                        # full log
git log --oneline --graph      # compact tree view
git log --author="Vaishnavi"   # only your commits

# Undo changes
git restore file.c             # discard edits to a file (irreversible)
git restore --staged file.c    # unstage (keep edits, just remove from staging)
git reset HEAD~1               # undo last commit (keep changes in working tree)

# Save work in progress without committing
git stash                      # stash all unstaged changes
git stash pop                  # restore them
git stash list                 # see all stashes

# See what changed between commits
git diff HEAD~2 HEAD           # diff last 2 commits
git show abc1234               # show a specific commit

# See who changed a line
git blame week01/README.md
```

---

## .gitignore — What Not to Commit

The repo's `.gitignore` already excludes build artifacts. Key rule: **only commit source code**, never compiled output.

```bash
# Check if a file is being ignored
git check-ignore -v build/firmware.elf

# See what's ignored in a directory
git status --ignored
```

---

## Day 7 — Projects Day Preview (May 13)

All three projects use everything from Days 1–6:

| Project | Key skills |
|---------|-----------|
| C Memory Playground | Pointers, structs, heap (malloc/free), Valgrind, GDB |
| Bit Manipulation Library | Bitwise ops, header-only library, `assert()` unit tests |
| Logic Gate Simulator | Enums, function pointers, truth tables, combinational logic |

Suggested order: finish Bit Lib first (2–3 hours), then Logic Simulator (2 hours), then Memory Playground (2–3 hours + Valgrind + GDB session).

Push each project to your folder as you complete it.

---

## YouTube Resources

| Topic | Video | Channel |
|-------|-------|---------|
| Git full beginner course | [Git and GitHub for Beginners](https://www.youtube.com/watch?v=RGOj5yH7evk) | freeCodeCamp |
| Git branching explained | [Git Branches Tutorial](https://www.youtube.com/watch?v=e2IbNHi4uCI) | freeCodeCamp |
| How git works internally | [Git Internals](https://www.youtube.com/watch?v=P6jD966jzlk) | GitButler |
| Merge conflicts | [Resolving Git Merge Conflicts](https://www.youtube.com/results?search_query=git+merge+conflict+resolution+tutorial) | Traversy Media |
| Git for teams | [Git Collaboration Workflow](https://www.youtube.com/results?search_query=git+pull+request+workflow+github+collaboration) | The Coding Train |
