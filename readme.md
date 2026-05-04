1. Initialize a git repository locally
  1.1 How? → `git init`
  1.2 Its Action? → Creates a `.git folder`
2. Content of `.git folder`
  It has 4 folders: `hooks`, `info`, `objects`, `refs`
  2.1 `hooks` folder
    This contains scripts that run on Git events.
    Examples:
      pre-commit → runs before committing
      post-commit → runs after commit
      pre-push → before pushing
  2.2 `info` folder
    It contains `exclude` file that acts as `.gitignore` but local only not shared
  2.3 `objects` folder
    Stores all content in Git (commits, tags, trees, blobs)
    Everything is saved as compressed objects identified by SHA-1 hashes
    Types:
      blob → file contents
      tree → directory structure
      commit → snapshot + metadata (author, message, parent)
      tag → annotated tags
    Each object (commit, tree, blob, tag), its format is: `<type><size>\0<content>`, this format is the one used by git to set SHA-1 (Naming) for each object
  2.4 `refs` folder
    This holds references to objects (not the objects themselves).
    Structure:
      refs/heads/ → branches (e.g. main, dev) (Heads to branches)
      refs/tags/ → tags
      refs/remotes/ → remote-tracking branches
    Each file contains an object hash.
3. About Git
  Git follow 3 tier architecture, where we have 3 different stages where our files can exist:
    1- Working Directory -> Git has no view over you working yet
    2- Staging Area (Index) -> Your working are staged to the index so git can check which files are modified, added, deleted
    3- Committed Area -> Git takes your new working and create a commit object of it
  To stage files -> `git add <file>` or `git add .` (to stage all files) or `git add <pattern>` (add all files that matches a regex)
  To stage part of the file: `git add -p <file>` (Git shows each change (hunk) and asks stage or not) or `git add -e <file>` (Vim)
  To commit your updates -> `git commit -m "<commit_msg>"` commits all changes in the stage area
  commit single file `git commit <file> -m "<commit_msg>"` not need to do `git add <file>`

  How does git stores commits?
    -> On every commit, git creates a commit object, which includes a tree object, which includes blobs(files)
    -> Each file object (blob) is stored with name = SHA-1 equivalent 
    -> If there are 2 files with the same content git won't create different blobs for them

  To view an object?
    -> `git cat-file <object_id>` -> gets all data type, size, content
    -> To get specific part:
      - `git cat-file -t <object_id>` -> gets type
      - `git cat-file -s <object_id>` -> gets size
      - `git cat-file -p <object_id>` -> gets content

4. Staging Area (Index)
  It connects your working directory (files) with the Git database (objects).
  It stores a list of files (with their blob hashes) plus some metadata
  To list files in staged ares (All files that git is tracking):
    - `git ls-files` -> list file names
    - `git ls-files --stage` -> list (mode blob_id file_name)
    - `git ls-files --debug` -> list (file_name file_metadata)
  Common Modes:
    - 100644  Normal file
    - 100755  Executable file
    - 120000  Symlink
    - 40000   Directory (subtree)
  Ignoring staged/tracked file `git rm --cached <file>` -> totally removes the file's blob from staging area, file become untracked but content is preserved in working directory
  If you didnt stage changes yet and want to restore file content (discard working changes) -> `git restore <file>`
  If you staged changes and need to unstage them but still have the modifications in working directory -> `git restore --staged <file>`
  If you staged changes and need to unstage and also removechanges from working directory -> `git restore --staged --worktree <file>` = doing `git restore --staged <file>` then `git restore <file>`
  Note that `git rm --cached <file>` will do the same work as `git restore --staged <file>`, if the file is newly created
5. Tree Object
  Each tree entry includes:
    - File mode (permissions)
    - File name
    - Reference to a blob (or subtree)

6. Commit Object
  Each commit object includes:
    - Tree hash
    - Parent commit hash (optional)
    - Author
    - Committer
    - Date
    - Message
7.Tags
  There are 2 types of tags lightweight tag and annotated tag
  Lieghtweight Tag is just a quick label — no extra info (no message, date, tagger info, etc.)
  Annotated Tag it is stored as a full Git object (like a commit). Contains tagger name, date, and message
  Tags must be unique per branch
  Tags are not pushed automatically
  - Create lightweight tag:
      git tag <tag_name>

  - Create annotated tag:
      git tag -a <version> -m "<msg>"

  - List all local tags:
      git tag

  - Delete local tag:
      git tag -d <tag-name>

  - Delete remote tag:
      git push origin --delete tag <tag-name>

  - Push local tag:
      git push origin <tag-name>

  - Push all local tags:
      git push origin --tags

  - List tags in remote repository:
      git ls-remote --tags <remote-name>

  - Fetch all tags:
      git fetch --tags

  - Fetch specific tag:
      git fetch <remote-name> tag <tag-name>
8. HEAD pointer
  It is a pointer used to reference the latest commit on the current branch by default
  But it can be moved to another commits by updating it using `git update-ref HEAD <commit_hash>`
  If you have only one commit in your repo and you want to reset it: `git update-ref -d HEAD`
9. Committed Area
  When you have already committed changes and want to rollback:
    1- To create a new commit that undo the changes `git revert <commit_hash>` or `git revert HEAD`
    2- To remove commits by taking the HEAD pointer backwards `git reset <commit_hash>` or `git reset HEAD~<steps>`:
        - It has options `--soft` & `--mixed` (default) & `--hard`
        - `--soft` option: move the pointer only (changes still staged and still exist in working directory)
        - `--mixed` option: move the pointer and unstage the changes (changes still exist in working directory)
        - `--hard` option: move the pointer, unstage changes, and remove changes from the working directory
      -> Note: `git reset --soft HEAD~` then `git restore --staged --worktree .` equivalent to `git reset --hard HEAD~`
    3- To modify last commit:
        - Modify commit msg: `git commit --amend -m "new commit message"`
        - Modify commit content: stage the new changes you want to add then `git commit amend --no-edit`
        - Modify both commit msg and content: stage the new changes you wan tto add then `git commit --amend -m "new commit message"`
  When you use `git reset` the 
10.  Stashing
  When youʼre in the middle of some work but need to switch tasks or branches without committing unfinished changes:
    1- use `git stash` (to stash tracked files only), `git stash -u` (to stash both tracked and untracked)
    2- push the stashed changes to the stash list using `git stash push -m <stash_msg>`
   -> you can merge the above 2 commands in one like this `git stash push -um <stash_msg>`
  To view the stash list -> `git stash list`
  To apply stashed changes:
    1- apply last stashed changes `git stash apply`
    2- apply specific stashed changes `git stash apply <stash_id>` get stash id by copying it from the stash list first
  To apply and delete autoatically `git stash pop` / `git stash pop <stash_id>`
  Delete stashed changes `git stash drop` / `git stash drop <stash_id>`
  To clear the whole list `git stash clear`

11. Fetch:
  It downloads changes from remote (Updates refs/remotes/origin/*), it doesn't change the working directory
  Fetch all: `git fetch --all`
  Fetch tags only: `git fetch --tags`
  Fetch specific branch: `git fetch origin main`
12. Working with Remotes:
  - To clone a repo:
      git clone <repo_link>

  - To connect local repo with remote repo:
      git remote add <shortname(origin)> <repo_link>

  - To list remotes:
      git remote

  - To list more detailed info (URLs) about remotes:
      git remote -v

  - To fetch from remote:
      git fetch <remote-name>

  - To pull from remote:
      git pull <remote-name>

  - To push changes to remote branch:
      git push <remote-name> <branch-name>

  - To rename remote:
      git remote rename <old-name> <new-name>
  
13. Branches:
  - To create a new branch and switch to it (if branch exists will error):
    - use `git checkout -b <branch_name>`
    - or more modern `git switch -c <branch_name>`
  - To create a new branch but without switching to it `git branch <branch_name>`
  - Switch to another existing branch:
    - use `git checkout <branch_name>`
    - or more modern `git switch <branch_name>`
  - To list local and remote branches:
      `git branch -a`
  - To delete a remote branch:
      `git push <remote-name> --delete <branch-name>`
  - To delete a local branch, only if merged:
      `git branch -d <branch-name>`
  - To delete a local branch, force in case of unmerged:
      `git branch -D <branch-name>`
14. Merging:
  It is the process of merging branches
  Merging create a new merge commit, if you want merge branch x(feature) to y(main):
    - `git switch y`
    - `git merge x`
15. Rebase Vs Merging:
  MERGE
    `git merge branch`
    keeps history
    creates merge commit
    non-linear history
  REBASE
    `git rebase branch`
    rewrites history
    linear history
    cleaner log
16. Pull = fetch + merge
17. Helpers:
       - view all logs `git log` use option `--oneline` to print compacted version
    - view status (which files modified, updated, deleted) `git status`, use option `-s` to print file names with ??(untracked) M(green -> staged but not commited) M(red -> modified and not staged)
    - To get difference between working dir and staging area:
        - `git diff` -> get difference of all tracked files
        - `git diff <file>` -> get difference of a specific file
    - To get the difference between stagin area and committed ares:
        - `git diff --staged` -> get difference of all tracked files
        - `git diff --staged <file>` -> get difference of a specific file
    - git difference of tracked files between working dir and stagin area 
    - Retrieve the current commit that the HEAD is pointing to `git show HEAD`
    - Retrieve any object details(commit, tree, blob, tag) `git show <commit_hash>`
18. To copy a repo to another including(refs, branches, tags):
  - `git clone --bare https://source-repo-url.git`
  - `git push --mirror https://destination-repo-url.git`

19. Submodules:

20. Theoritical:
  - Semantic Versioning: Major.Minor.Patch:
    - Major -> breaking changes
    - Minor -> added feature
    - Patch -> fixed bug
  - Branching Strategies:
    - Git Flow: more merging steps, more branches
    - Trunk based: fewer branches, merge fast
    - Feature based: Pull request for every change
  - Common branches:
    - Main/Master: The stable production-ready code.
    - Develop: The integration branch for features.
    - Release branches: For preparing a release.
    - Hotfix branches: For quick fixes to production
    


