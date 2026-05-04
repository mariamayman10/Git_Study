# Git Notes

## Table of Contents

1. [Initialize a Git Repository](#1-initialize-a-git-repository)
2. [The `.git` Folder](#2-the-git-folder)
3. [Three-Tier Architecture](#3-three-tier-architecture)
4. [Staging Area (Index)](#4-staging-area-index)
5. [Git Objects](#5-git-objects)
6. [Tags](#6-tags)
7. [HEAD Pointer](#7-head-pointer)
8. [Undoing Changes (Committed Area)](#8-undoing-changes-committed-area)
9. [Stashing](#9-stashing)
10. [Branches](#10-branches)
11. [Merging](#11-merging)
12. [Rebasing](#12-rebasing)
13. [Working with Remotes](#13-working-with-remotes)
14. [Fetch vs Pull](#14-fetch-vs-pull)
15. [Submodules](#15-submodules)
16. [Helpers & Inspection](#16-helpers--inspection)
17. [Copying a Repository](#17-copying-a-repository)
18. [Theory & Concepts](#18-theory--concepts)

---

## 1. Initialize a Git Repository

```bash
git init
```

Creates a `.git` folder in the current directory, which is the local Git database for your project.

---

## 2. The `.git` Folder

The `.git` folder has 4 main subdirectories: `hooks/`, `info/`, `objects/`, `refs/`.

### `hooks/`
Contains scripts that run automatically on Git events:
- `pre-commit` — runs before a commit is created
- `post-commit` — runs after a commit is created
- `pre-push` — runs before pushing to a remote

### `info/`
Contains an `exclude` file that works like `.gitignore` but is local-only and not shared with others.

### `objects/`
Stores all Git content as compressed objects identified by SHA-1 hashes. There are 4 object types:

| Type | Description |
|------|-------------|
| `blob` | File contents |
| `tree` | Directory structure (list of blobs and subtrees) |
| `commit` | Snapshot + metadata (author, message, parent hash) |
| `tag` | Annotated tag object |

Every object is stored in the format `<type> <size>\0<content>` before being hashed and compressed. This format is what Git uses to compute the SHA-1 name for each object.

> If two files have the same content, Git stores only **one** blob for both — deduplication is built-in.

### `refs/`
Holds references (pointers) to object hashes, not the objects themselves:
- `refs/heads/` — local branches (e.g. `main`, `dev`)
- `refs/tags/` — tags
- `refs/remotes/` — remote-tracking branches

Each file inside simply contains a commit hash.

---

## 3. Three-Tier Architecture

Git uses a three-stage model for tracking changes:

```
Working Directory  →  Staging Area (Index)  →  Committed Area
  (untracked /           (git add)              (git commit)
   modified)
```

| Stage | Description |
|-------|-------------|
| **Working Directory** | Your local files; Git monitors changes but doesn't manage them yet |
| **Staging Area (Index)** | Files queued for the next commit; Git tracks what's modified, added, or deleted |
| **Committed Area** | A permanent snapshot in the Git history |

### Staging files

```bash
git add <file>           # Stage a specific file
git add .                # Stage all changes
git add <pattern>        # Stage files matching a glob/regex
git add -p <file>        # Interactively stage individual hunks
git add -e <file>        # Open hunk editor in Vim
```

### Committing

```bash
git commit -m "<message>"           # Commit all staged changes
git commit <file> -m "<message>"    # Stage + commit a single file in one step
```

### How Git stores commits

On every commit, Git creates:
1. A **commit object** (contains tree hash, parent hash, author, message)
2. A **tree object** (directory snapshot)
3. **Blob objects** for each file (file contents, deduplicated by content hash)

---

## 4. Staging Area (Index)

The index connects your working directory to the Git object database. It stores a list of tracked files along with their blob hashes and metadata.

### Inspecting the index

```bash
git ls-files                # List tracked file names
git ls-files --stage        # List with mode, blob hash, and file name
git ls-files --debug        # List with full file metadata
```

### File modes

| Mode | Meaning |
|------|---------|
| `100644` | Normal file |
| `100755` | Executable file |
| `120000` | Symbolic link |
| `040000` | Directory (subtree) |

### Restoring and unstaging

| Goal | Command |
|------|---------|
| Discard working directory changes (not yet staged) | `git restore <file>` |
| Unstage changes (keep in working directory) | `git restore --staged <file>` |
| Unstage **and** discard working directory changes | `git restore --staged --worktree <file>` |
| Stop tracking a file (untrack but keep on disk) | `git rm --cached <file>` |

> **Note:** `git rm --cached <file>` and `git restore --staged <file>` behave the same way for **newly created** (never-committed) files.

---

## 5. Git Objects

### Inspecting objects

```bash
git cat-file <object_id>       # Show type, size, and content
git cat-file -t <object_id>    # Show type only
git cat-file -s <object_id>    # Show size only
git cat-file -p <object_id>    # Show content (pretty-printed)
```

### Tree object entries

Each entry in a tree object contains:
- File mode (permissions)
- File name
- Reference to a blob (for files) or another tree (for subdirectories)

### Commit object fields

Each commit object contains:
- Tree hash
- Parent commit hash (absent for the very first commit)
- Author name + email + timestamp
- Committer name + email + timestamp
- Commit message

---

## 6. Tags

There are two types of tags:

| Type | Description |
|------|-------------|
| **Lightweight** | Just a named pointer to a commit — no extra info |
| **Annotated** | A full Git object with tagger name, date, and message |

> Tags are **not pushed automatically** — you must push them explicitly.  
> Tag names must be **unique** within a repository.

### Tag commands

```bash
# Create
git tag <tag_name>                        # Lightweight tag at HEAD
git tag -a <version> -m "<message>"       # Annotated tag at HEAD

# List
git tag                                   # List all local tags
git ls-remote --tags <remote>             # List tags on a remote

# Delete
git tag -d <tag_name>                     # Delete local tag
git push origin --delete tag <tag_name>   # Delete remote tag

# Push
git push origin <tag_name>               # Push a single tag
git push origin --tags                   # Push all local tags

# Fetch
git fetch --tags                          # Fetch all remote tags
git fetch <remote> tag <tag_name>         # Fetch a specific tag
```

---

## 7. HEAD Pointer

`HEAD` is a special pointer that references the latest commit on the currently checked-out branch.

```bash
git update-ref HEAD <commit_hash>    # Move HEAD to a specific commit
git update-ref -d HEAD               # Delete HEAD (useful for resetting the first-ever commit)
```

---

## 8. Undoing Changes (Committed Area)

### Revert — safe undo (creates a new commit)

```bash
git revert <commit_hash>    # Creates a new commit that undoes the target commit
git revert HEAD             # Revert the latest commit
```

### Reset — move HEAD backwards (rewrites history)

```bash
git reset <commit_hash>
git reset HEAD~<n>          # Go back n commits
```

| Option | Pointer | Staging Area | Working Directory |
|--------|---------|--------------|-------------------|
| `--soft` | Moved | Unchanged (changes stay staged) | Unchanged |
| `--mixed` *(default)* | Moved | Cleared (changes unstaged) | Unchanged |
| `--hard` | Moved | Cleared | Cleared (changes lost) |

> **Tip:** `git reset --soft HEAD~1` followed by `git restore --staged --worktree .` is equivalent to `git reset --hard HEAD~1`.

### Amend — modify the last commit

```bash
git commit --amend -m "<new message>"         # Change commit message only
git commit --amend --no-edit                  # Add staged changes, keep message
git commit --amend -m "<new message>"         # Add staged changes + new message
```

> For the last two, stage your new changes with `git add` first.

---

## 9. Stashing

Use stashing when you need to switch context without committing unfinished work.

```bash
# Save work to the stash
git stash                          # Stash tracked files only
git stash -u                       # Stash tracked + untracked files
git stash push -m "<message>"      # Stash with a descriptive label
git stash push -um "<message>"     # Stash untracked + label (combined)

# View
git stash list                     # List all stashed entries

# Apply
git stash apply                    # Apply the most recent stash (keep it in list)
git stash apply <stash_id>         # Apply a specific stash by ID
git stash pop                      # Apply most recent stash and remove it
git stash pop <stash_id>           # Apply specific stash and remove it

# Delete
git stash drop                     # Remove the most recent stash
git stash drop <stash_id>          # Remove a specific stash
git stash clear                    # Remove all stashes
```

---

## 10. Branches

```bash
# Create and switch
git switch -c <branch>             # Create new branch and switch to it (modern)
git checkout -b <branch>           # Same (classic syntax)

# Create without switching
git branch <branch>

# Switch to existing branch
git switch <branch>                # Modern
git checkout <branch>              # Classic

# List
git branch -a                      # List all local and remote branches

# Delete
git branch -d <branch>            # Delete local branch (only if merged)
git branch -D <branch>            # Force-delete local branch (even if unmerged)
git push <remote> --delete <branch>  # Delete remote branch
```

---

## 11. Merging

Merging integrates one branch into another, preserving history and creating a merge commit.

```bash
# To merge feature branch into main:
git switch main
git merge feature
```

> Merge commits have **two parent commits**, capturing the full history of both branches.

---

## 12. Rebasing

Rebasing replays commits from one branch on top of another, producing a linear history.

```bash
# To rebase current branch onto main:
git rebase main
```

| | Merge | Rebase |
|-|-------|--------|
| History | Preserved, non-linear | Rewritten, linear |
| Merge commit | Yes | No |
| Best for | Shared/public branches | Local/feature branches |

> ⚠️ **Never rebase commits that have already been pushed to a shared remote.** It rewrites history and causes problems for other contributors.

---

## 13. Working with Remotes

```bash
# Clone
git clone <url>

# Manage remotes
git remote add <name> <url>        # Connect local repo to a remote
git remote                         # List remotes (names only)
git remote -v                      # List remotes with URLs
git remote rename <old> <new>      # Rename a remote

# Sync
git fetch <remote>                 # Download changes without merging
git pull <remote>                  # Fetch + merge (= git fetch + git merge)
git push <remote> <branch>         # Push local branch to remote
```

---

## 14. Fetch vs Pull

| Command | What it does |
|---------|-------------|
| `git fetch <remote>` | Downloads remote changes and updates `refs/remotes/origin/*`. Does **not** touch your working directory. |
| `git pull <remote>` | `git fetch` + `git merge` — updates your working directory. |

```bash
git fetch --all          # Fetch from all remotes
git fetch --tags         # Fetch tags only
git fetch origin main    # Fetch a specific branch
```

---

## 15. Submodules

A **submodule** lets you embed one Git repository inside another as a tracked dependency. The parent repo stores a reference to a specific commit of the submodule — not the submodule's files directly.

### Add a submodule

```bash
git submodule add <url> <path>
# Example:
git submodule add https://github.com/user/lib.git libs/lib
```

This creates a `.gitmodules` file and a new entry in the index pointing to the submodule's commit.

### Clone a repo that contains submodules

```bash
# Option 1: clone + initialize in one step
git clone --recurse-submodules <url>

# Option 2: clone first, then initialize
git clone <url>
git submodule init       # Register submodules from .gitmodules
git submodule update     # Check out the recorded commit for each submodule
```

### Update submodules

```bash
# Pull the latest commit from each submodule's remote
git submodule update --remote

# Update all submodules recursively
git submodule update --remote --recursive
```

### Work inside a submodule

A submodule is a full Git repo — you can `cd` into it and use regular Git commands:

```bash
cd libs/lib
git checkout main
git pull
cd ../..
git add libs/lib         # Stage the updated submodule reference
git commit -m "Update lib to latest"
```

### View submodule status

```bash
git submodule status         # Show each submodule's current commit hash
git submodule foreach 'git status'  # Run a command inside every submodule
```

### Remove a submodule

```bash
git submodule deinit -f <path>      # Unregister the submodule
git rm -f <path>                    # Remove the submodule directory from index
rm -rf .git/modules/<path>          # Remove the cached submodule repo
```

### Key things to know

- The parent repo stores a **commit pointer**, not the submodule's full history.
- After cloning, submodules start in a **detached HEAD** state — check out a branch before making changes.
- Always commit both the submodule update **and** the parent repo change together.
- Use `git clone --recurse-submodules` or people cloning your repo won't get the submodule contents.

---

## 16. Helpers & Inspection

```bash
# Log
git log                            # Full commit history
git log --oneline                  # Compact one-line-per-commit view

# Status
git status                         # Show modified, staged, and untracked files
git status -s                      # Short format:
                                   #   ?? = untracked
                                   #   M (green) = staged
                                   #   M (red) = modified but not staged

# Diff
git diff                           # Working directory vs staging area (all tracked files)
git diff <file>                    # Working directory vs staging area (specific file)
git diff --staged                  # Staging area vs last commit (all tracked files)
git diff --staged <file>           # Staging area vs last commit (specific file)

# Inspect objects
git show HEAD                      # Show the latest commit's details and diff
git show <hash>                    # Show any object (commit, tree, blob, tag)
```

---

## 17. Copying a Repository

To mirror a repo to a new remote (including all branches, tags, and refs):

```bash
git clone --bare <source_url>
cd <repo>.git
git push --mirror <destination_url>
```

---

## 18. Theory & Concepts

### Semantic Versioning

Format: `MAJOR.MINOR.PATCH`

| Part | When to increment |
|------|------------------|
| `MAJOR` | Breaking changes |
| `MINOR` | New backward-compatible features |
| `PATCH` | Bug fixes |

### Branching Strategies

| Strategy | Description |
|----------|-------------|
| **Git Flow** | Multiple long-lived branches (`main`, `develop`, `release/*`, `hotfix/*`); heavier process |
| **Trunk-Based Development** | Everyone works close to `main`; short-lived feature branches; merge fast |
| **Feature Branch** | A pull request for every change; good for code review workflows |

### Common Branch Types

| Branch | Purpose |
|--------|---------|
| `main` / `master` | Stable, production-ready code |
| `develop` | Integration branch for in-progress features |
| `release/*` | Preparing a new release (version bump, final fixes) |
| `hotfix/*` | Urgent fixes applied directly to production |
