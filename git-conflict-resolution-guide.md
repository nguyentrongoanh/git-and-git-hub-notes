# Git Conflict Resolution Guide: Divergent Branches and Merge Conflicts

## Table of Contents
1. [The Divergent Branches Error](#the-divergent-branches-error)
2. [Understanding the Problem](#understanding-the-problem)
3. [Three Resolution Methods](#three-resolution-methods)
4. [Step-by-Step Resolution](#step-by-step-resolution)
5. [Understanding Merge Conflicts](#understanding-merge-conflicts)
6. [Common Scenarios and Solutions](#common-scenarios-and-solutions)
7. [Prevention Strategies](#prevention-strategies)
8. [Advanced Scenarios](#advanced-scenarios)
9. [Visual Examples](#visual-examples)
10. [Best Practices](#best-practices)

---

## The Divergent Branches Error

### What Happened in Our Case

**The Error Messages**:
```bash
# First attempt to push
$ git push origin main
To https://github.com/nguyentrongoanh/dev-ops-jsmastery.git
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'https://github.com/nguyentrongoanh/dev-ops-jsmastery.git'
hint: Updates were rejected because the remote contains work that you do not
hint: have locally.

# Then attempted to pull
$ git pull
hint: You have divergent branches and need to specify how to reconcile them.
fatal: Need to specify how to reconcile divergent branches.
```

### What This Means in Plain English

Imagine you and a friend are both editing the same Google Doc:
- You started editing at version 5
- Your friend also started editing at version 5
- Your friend finished first and saved their changes (now version 6)
- You try to save your changes, but Google says "Wait! Someone already made version 6!"
- You need to decide: Do you want to combine both versions or replace one with the other?

**In Git terms**:
- You both started from commit `d960dd9`
- GitHub has commit `1ef3e46` (Create README.md)
- You have commit `f455814` (add comment to index.js)
- Git doesn't know which one should come first

---

## Understanding the Problem

### The Timeline of Events

#### What You Did Locally:
```bash
1. Started with commit: d960dd9 (feat: add CI pipeline)
2. Made changes to index.js
3. Committed: f455814 (add comment to index.js)
4. Tried to push → REJECTED
```

#### What Happened on GitHub:
```bash
1. Started with commit: d960dd9 (feat: add CI pipeline)
2. Someone (possibly you on GitHub website) created README.md
3. Committed: 1ef3e46 (Create README.md)
4. This was saved to GitHub first
```

### Visual Representation

**Before the conflict**:
```
Your Local Repository:
d960dd9 ← f455814 (main)
  ↓         ↓
 "CI"   "add comment"

GitHub Remote Repository:
d960dd9 ← 1ef3e46 (main)
  ↓         ↓
 "CI"   "Create README"
```

**The branches have diverged** - they both moved forward from the same point but in different directions.

```
        f455814 (local main)
       /
d960dd9
       \
        1ef3e46 (remote main)
```

---

## Three Resolution Methods

Git offers three ways to reconcile divergent branches:

### Method 1: Merge (--no-rebase)

**Command**: `git pull --no-rebase` or `git pull --merge`

**What it does**:
- Creates a new "merge commit"
- Keeps all history from both branches
- Shows that branches diverged and came back together

**Result**:
```
        f455814 (your commit)
       /         \
d960dd9          merge commit
       \         /
        1ef3e46 (remote commit)
```

**Pros**:
- ✅ Preserves complete history
- ✅ Shows when branches diverged
- ✅ Safe - no commits are modified
- ✅ Good for teams/collaborative projects

**Cons**:
- ❌ Creates extra merge commits
- ❌ History can become cluttered
- ❌ Harder to read linear history

**When to use**:
- Working on a team with merge-based workflow
- Want to preserve exact history of who did what when
- Working on public/shared branches

---

### Method 2: Rebase (--rebase) ⭐ RECOMMENDED

**Command**: `git pull --rebase`

**What it does**:
- Takes your local commits
- Temporarily removes them
- Applies remote commits
- Re-applies your commits on top

**Result**:
```
d960dd9 ← 1ef3e46 ← f455814' (your commit replayed)
```

**Pros**:
- ✅ Clean, linear history
- ✅ Easy to read and understand
- ✅ No extra merge commits
- ✅ Professional-looking history

**Cons**:
- ❌ Rewrites commit history (new commit IDs)
- ❌ Can be confusing for beginners
- ❌ Dangerous on shared/public branches

**When to use**:
- Personal feature branches
- You haven't shared your commits yet
- Want clean, linear history
- Working solo or on your own branch

---

### Method 3: Fast-Forward Only (--ff-only)

**Command**: `git pull --ff-only`

**What it does**:
- Only updates if your branch is directly behind
- Refuses to merge or rebase if branches diverged

**Result**:
- Either succeeds (if you're just behind)
- Or fails with error (if diverged)

**Pros**:
- ✅ Very safe
- ✅ Prevents accidental merges
- ✅ Forces you to be explicit

**Cons**:
- ❌ Fails when branches diverge
- ❌ Requires manual resolution
- ❌ Can't handle our scenario

**When to use**:
- Want to be extra cautious
- Don't want automatic merges
- Prefer explicit control

---

## Step-by-Step Resolution

Let's walk through resolving the divergent branches error from the beginning.

### Scenario Setup

You're in this situation:
```bash
$ git push origin main
! [rejected]        main -> main (fetch first)
error: failed to push some refs
```

### Step 1: Don't Panic - Assess the Situation

```bash
# Check current status
git status
```

**Output**:
```
On branch main
Your branch is ahead of 'origin/main' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
```

**What this tells you**:
- You're on the `main` branch
- You have 1 commit that GitHub doesn't have
- Your working directory is clean (no unsaved changes)

### Step 2: Fetch Latest Changes

```bash
# Fetch remote changes without merging
git fetch origin
```

**What this does**:
- Downloads commits from GitHub
- Doesn't change your local files
- Updates your view of what's on GitHub

### Step 3: Compare Your Branch with Remote

```bash
# See the divergence
git log --oneline --graph --all -10
```

**Output**:
```
* f455814 (HEAD -> main) add comment to index.js
| * 1ef3e46 (origin/main) Create README.md
|/
* d960dd9 feat: add CI pipeline
* bdebf73 remove all files
* 4faaaaf add hello file
```

**Understanding the graph**:
- `*` = A commit
- `|` = A branch line
- `/` = Branches splitting/merging
- `(HEAD -> main)` = Your current position
- `(origin/main)` = Where GitHub's main is

**What you see**:
- Both branches split from `d960dd9`
- Your local has `f455814`
- Remote has `1ef3e46`

### Step 4: View What Changed on Remote

```bash
# See what's in the remote commit you don't have
git log origin/main --not main
```

**Output**:
```
commit 1ef3e46
Author: Oanh Nguyen
Date: ...

    Create README.md
```

**Alternative - see the actual changes**:
```bash
git diff main origin/main
```

### Step 5: View What Changed Locally

```bash
# See what's in your local commit that remote doesn't have
git log main --not origin/main
```

**Output**:
```
commit f455814
Author: Oanh Nguyen
Date: ...

    add comment to index.js
```

### Step 6: Choose Resolution Method

#### Option A: Rebase (Recommended for this scenario)

```bash
# Pull with rebase
git pull --rebase origin main
```

**What happens**:
```
Step 1: Git removes your commit temporarily
        d960dd9 (your commits stored safely)

Step 2: Git applies remote commit
        d960dd9 ← 1ef3e46

Step 3: Git replays your commit on top
        d960dd9 ← 1ef3e46 ← f60918e (new commit ID)
```

**Expected output**:
```
From https://github.com/nguyentrongoanh/dev-ops-jsmastery
 * branch            main       -> FETCH_HEAD
Successfully rebased and updated refs/heads/main.
```

#### Option B: Merge

```bash
# Pull with merge
git pull --no-rebase origin main
```

**What happens**:
```
Git creates a merge commit combining both:

        f455814
       /       \
d960dd9        [merge commit]
       \       /
        1ef3e46
```

**Expected output**:
```
Merge made by the 'recursive' strategy.
 README.md | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
```

### Step 7: Verify Resolution

```bash
# Check status
git status
```

**After rebase**:
```
On branch main
Your branch is ahead of 'origin/main' by 1 commit.
  (use "git push" to publish your local commits)
```

**After merge**:
```
On branch main
Your branch is ahead of 'origin/main' by 2 commits.
  (use "git push" to publish your local commits)
```

```bash
# View history
git log --oneline --graph -5
```

**After rebase**:
```
* f60918e (HEAD -> main) add comment to index.js
* 1ef3e46 (origin/main) Create README.md
* d960dd9 feat: add CI pipeline
```

**After merge**:
```
*   a1b2c3d (HEAD -> main) Merge branch 'main' of github.com:...
|\
| * 1ef3e46 (origin/main) Create README.md
* | f455814 add comment to index.js
|/
* d960dd9 feat: add CI pipeline
```

### Step 8: Push to GitHub

```bash
git push origin main
```

**Expected output**:
```
To https://github.com/nguyentrongoanh/dev-ops-jsmastery.git
   1ef3e46..f60918e  main -> main
```

**Success!** Your changes are now on GitHub.

### Step 9: Verify on GitHub

```bash
# Open repository in browser
gh repo view --web

# Or check remotely
git ls-remote origin main
```

---

## Understanding Merge Conflicts

Sometimes when merging or rebasing, you'll encounter **merge conflicts** - when Git can't automatically combine changes.

### What Causes Merge Conflicts?

**Scenario**: Two people edited the **same lines** in the **same file**.

```javascript
// Original file (commit A)
function greet() {
  console.log("Hello");
}

// Person 1 changes it to (commit B):
function greet() {
  console.log("Hello, World!");
}

// Person 2 changes it to (commit C):
function greet() {
  console.log("Hi there!");
}
```

Git doesn't know which version to keep!

### How Conflicts Look

When you try to merge/rebase:
```bash
$ git pull --rebase origin main
Auto-merging index.js
CONFLICT (content): Merge conflict in index.js
error: could not apply f455814... add comment to index.js
```

**The file will look like this**:
```javascript
function greet() {
<<<<<<< HEAD
  console.log("Hello, World!");
=======
  console.log("Hi there!");
>>>>>>> f455814 (add comment to index.js)
}
```

### Conflict Markers Explained

```javascript
<<<<<<< HEAD
  console.log("Hello, World!");    // ← Current version (remote)
=======
  console.log("Hi there!");        // ← Your version (local)
>>>>>>> f455814
```

- `<<<<<<< HEAD`: Start of conflict - current version (what's on the branch you're merging INTO)
- `=======`: Separator between the two versions
- `>>>>>>> commit-id`: End of conflict - incoming version (what you're trying to merge)

### Resolving Conflicts

#### Step 1: See What Files Have Conflicts

```bash
git status
```

**Output**:
```
On branch main
You have unmerged paths.
  (fix conflicts and run "git commit")

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   index.js
```

#### Step 2: Open and Edit the Conflicted File

```bash
# Open in your editor
code index.js
# or
vim index.js
```

**You'll see**:
```javascript
function greet() {
<<<<<<< HEAD
  console.log("Hello, World!");
=======
  console.log("Hi there!");
>>>>>>> f455814
}
```

#### Step 3: Decide What to Keep

**Option A**: Keep remote version (HEAD)
```javascript
function greet() {
  console.log("Hello, World!");
}
```

**Option B**: Keep your version
```javascript
function greet() {
  console.log("Hi there!");
}
```

**Option C**: Keep both (if it makes sense)
```javascript
function greet() {
  console.log("Hello, World!");
  console.log("Hi there!");
}
```

**Option D**: Write something completely new
```javascript
function greet() {
  console.log("Hello, World! Hi there!");
}
```

#### Step 4: Remove Conflict Markers

Make sure to remove ALL markers:
- `<<<<<<< HEAD`
- `=======`
- `>>>>>>> commit-id`

**Final clean file**:
```javascript
function greet() {
  console.log("Hello, World! Hi there!");
}
```

#### Step 5: Mark as Resolved

```bash
# Stage the resolved file
git add index.js

# Check status
git status
```

**Output**:
```
On branch main
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:
        modified:   index.js
```

#### Step 6: Complete the Merge/Rebase

**If you were merging**:
```bash
git commit -m "Resolve merge conflict in index.js"
```

**If you were rebasing**:
```bash
git rebase --continue
```

**If you want to abort**:
```bash
# Abort merge
git merge --abort

# OR abort rebase
git rebase --abort
```

#### Step 7: Push

```bash
git push origin main
```

---

## Common Scenarios and Solutions

### Scenario 1: You Edited GitHub Files Directly (Web Editor)

**What happened**:
```
1. You have local commits
2. You also made changes directly on GitHub website
3. Now branches diverged
```

**Example**:
```bash
# Local
You edited index.js and committed

# GitHub
You created README.md via GitHub web interface

# Result
Divergent branches!
```

**Solution**:
```bash
# Fetch changes
git fetch origin

# See what's different
git log --oneline --graph --all -10

# Pull with rebase
git pull --rebase origin main

# Push
git push origin main
```

**Prevention**:
- Avoid editing files directly on GitHub
- If you must, pull before making local changes
- Use feature branches for web edits

---

### Scenario 2: Collaborating on Same Branch

**What happened**:
```
You and teammate both worked on main branch
Both pushed to different commits
```

**Example Timeline**:
```
10:00 AM - You: git pull (at commit A)
10:05 AM - Teammate: git pull (at commit A)
10:10 AM - Teammate: git push (creates commit B)
10:15 AM - You: git push → REJECTED!
```

**Solution**:
```bash
# Fetch latest
git fetch origin

# Check what changed
git log origin/main --not main

# Pull with rebase to keep clean history
git pull --rebase origin main

# If conflicts, resolve them
# (see conflict resolution section)

# Push
git push origin main
```

**Better approach - Use feature branches**:
```bash
# Instead of working on main
git checkout -b feature/my-work

# Make changes, commit
git add .
git commit -m "My changes"

# Push feature branch
git push -u origin feature/my-work

# Create PR, merge on GitHub
# Then update local main
git checkout main
git pull origin main
```

---

### Scenario 3: Force Push Gone Wrong

**What happened**:
```
Someone did git push --force
Your local history no longer matches remote
```

**Example**:
```bash
# Your local main
A → B → C → D

# Remote main (after force push)
A → B → X → Y

# Commits C and D are gone!
```

**Solution A: If you want THEIR changes**:
```bash
# Backup your work first!
git branch backup-main

# Reset to match remote exactly
git fetch origin
git reset --hard origin/main

# Your local commits are now in backup-main
```

**Solution B: If you want YOUR changes**:
```bash
# Fetch remote
git fetch origin

# Create branch from your local state
git branch my-changes main

# Reset to remote
git reset --hard origin/main

# Cherry-pick your commits
git cherry-pick C D

# Or merge your changes
git merge my-changes

# Push
git push origin main
```

**Prevention**:
- **NEVER** use `git push --force` on shared branches
- Use `git push --force-with-lease` if you must force push
- Use branch protection rules on GitHub
- Communicate with team before force pushing

---

### Scenario 4: Accidentally Committed to Wrong Branch

**What happened**:
```
You meant to create feature branch
But committed to main instead
Now main is ahead of origin/main
```

**Example**:
```bash
# You did:
git add .
git commit -m "New feature"

# You're on main!
# But you meant to be on feature/new-feature
```

**Solution**:
```bash
# Create the correct branch (keeps the commit)
git branch feature/new-feature

# Reset main to match remote (removes commit from main)
git reset --hard origin/main

# Switch to feature branch
git checkout feature/new-feature

# Your commit is now on the correct branch!
git push -u origin feature/new-feature
```

**Alternative if you already pushed to main**:
```bash
# Create feature branch
git checkout -b feature/new-feature

# Go back to main
git checkout main

# Revert the commit on main
git revert HEAD

# Push
git push origin main

# Continue work on feature branch
git checkout feature/new-feature
git push -u origin feature/new-feature
```

---

### Scenario 5: Multiple Commits Behind

**What happened**:
```
You haven't pulled in a while
Remote has many commits you don't have
You also have local commits
```

**Example**:
```
Your local:
A → B → C → your-commit

Remote:
A → B → C → D → E → F → G

Both branches diverged at C
```

**Solution**:
```bash
# Check how far behind you are
git fetch origin
git log --oneline origin/main --not main

# See your local commits
git log --oneline main --not origin/main

# Pull with rebase
git pull --rebase origin main

# This will replay your commits after G
# Result: A → B → C → D → E → F → G → your-commit'

# Push
git push origin main
```

**If there are conflicts during rebase**:
```bash
# Git will stop at each conflict
# Resolve the conflict in the file
# Then:
git add <resolved-file>
git rebase --continue

# Repeat until all commits are rebased

# Or abort if you want to start over
git rebase --abort
```

---

### Scenario 6: Rebase in Progress - What Now?

**What happened**:
```
You started a rebase
Conflicts appeared
You're stuck in "rebase in progress" state
```

**How to tell you're in this state**:
```bash
$ git status
interactive rebase in progress; onto 1ef3e46
Last command done (1 command done):
   pick f455814 add comment to index.js
No commands remaining.
You are currently rebasing branch 'main' on '1ef3e46'.
  (fix conflicts and run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Unmerged paths:
  (use "git restore --staged <file>..." to unstage)
  (use "git add <file>..." to mark resolution)
        both modified:   index.js
```

**Solution - Three Options**:

**Option 1: Fix conflicts and continue**:
```bash
# Edit conflicted files
vim index.js

# Remove conflict markers
# Save the file

# Stage resolved files
git add index.js

# Continue rebase
git rebase --continue

# If more conflicts appear, repeat
# Otherwise, rebase completes
```

**Option 2: Skip this commit**:
```bash
# Skip the problematic commit
git rebase --skip

# Use this if the commit is no longer needed
# or was already applied in a different form
```

**Option 3: Abort and start over**:
```bash
# Abort rebase
git rebase --abort

# Returns you to state before rebase
# Try a different approach (merge instead)
git pull --merge origin main
```

---

### Scenario 7: Pulled with Wrong Strategy

**What happened**:
```
You used merge but wanted rebase (or vice versa)
Want to undo and try again
```

**Example**:
```bash
# You did:
git pull --merge origin main

# But wanted:
git pull --rebase origin main
```

**Solution - Undo the merge**:
```bash
# Check reflog to see previous state
git reflog

# You'll see something like:
# a1b2c3d HEAD@{0}: pull: Merge made by the 'recursive' strategy
# f455814 HEAD@{1}: commit: add comment to index.js

# Reset to before the pull
git reset --hard HEAD@{1}

# Now do the correct pull
git pull --rebase origin main
```

**Alternative - Undo merge commit**:
```bash
# If you merged but haven't pushed yet
git reset --hard HEAD~1

# This removes the merge commit
# Then pull with rebase
git pull --rebase origin main
```

---

### Scenario 8: Forgot to Pull Before Making Changes

**What happened**:
```
You worked all day without pulling
Made lots of commits
Now need to sync with remote
Remote has new commits too
```

**Example**:
```
Your local:
A → B → C1 → C2 → C3 → C4 → C5 (your 5 commits)

Remote:
A → B → R1 → R2 → R3 (3 commits from others)

Both diverged from B
```

**Solution**:
```bash
# First, make sure all your work is committed
git status
# Should be clean

# Fetch remote
git fetch origin

# See the divergence
git log --oneline --graph --all -20

# Option 1: Rebase (cleaner)
git pull --rebase origin main

# Git will replay each of your commits
# You might need to resolve conflicts multiple times
# Once for each commit that conflicts

# Option 2: Merge (safer if many commits)
git pull --merge origin main

# Creates one merge commit
# Conflicts are resolved once

# Push when done
git push origin main
```

**Best practice for next time**:
```bash
# Before starting work each day
git checkout main
git pull origin main

# Create feature branch
git checkout -b feature/my-work

# Work, commit, push feature branch
# Keeps main clean
```

---

## Prevention Strategies

### 1. Always Pull Before Starting Work

```bash
# Every morning or before starting
git checkout main
git pull origin main
```

**Why**: Ensures you have latest changes before making new commits.

---

### 2. Use Feature Branches

```bash
# Instead of working on main
git checkout -b feature/my-feature

# Make changes
git add .
git commit -m "Add feature"

# Push feature branch
git push -u origin feature/my-feature

# Create PR on GitHub
# After merge, update main
git checkout main
git pull origin main
```

**Why**: Isolates your work, prevents divergent main branches.

---

### 3. Set Default Pull Strategy

```bash
# Choose one:

# Option A: Always rebase (clean history)
git config --global pull.rebase true

# Option B: Always merge (preserves history)
git config --global pull.rebase false

# Option C: Fast-forward only (safest)
git config --global pull.ff only
```

**Why**: Avoids the "need to specify" error.

---

### 4. Pull Before Push

```bash
# Good workflow
git pull origin main
git push origin main

# Or use this alias
git config --global alias.sync '!git pull && git push'

# Then just use
git sync
```

**Why**: Ensures you have latest changes before pushing yours.

---

### 5. Use Git Hooks

Create `.git/hooks/pre-push`:
```bash
#!/bin/bash

# Get current branch
branch=$(git rev-parse --abbrev-ref HEAD)

# If on main, ensure it's up to date
if [ "$branch" = "main" ]; then
  git fetch origin
  LOCAL=$(git rev-parse @)
  REMOTE=$(git rev-parse @{u})

  if [ $LOCAL != $REMOTE ]; then
    echo "Main branch is not up to date. Please pull first."
    exit 1
  fi
fi
```

```bash
# Make executable
chmod +x .git/hooks/pre-push
```

**Why**: Prevents pushing when you're behind remote.

---

### 6. Use Branch Protection Rules

On GitHub:
1. Go to repository Settings
2. Branches → Add rule
3. Branch name pattern: `main`
4. Enable:
   - ✅ Require pull request reviews
   - ✅ Require status checks to pass
   - ✅ Require branches to be up to date

**Why**: Forces use of PRs, prevents direct pushes to main.

---

### 7. Communicate with Team

**Best practices**:
- Use Slack/Teams to announce: "Pushing to main"
- Use feature branches for all work
- Have a team convention (merge vs rebase)
- Document workflow in README

**Why**: Coordination prevents conflicts.

---

## Advanced Scenarios

### Scenario A: Interactive Rebase to Clean History

**Situation**: You have messy commits you want to clean before pushing.

```bash
# Your commits
git log --oneline
# f1 Fix typo
# f2 Add feature
# f3 Fix bug
# f4 Fix another typo
# f5 More work on feature
```

**You want to combine f1, f2, f5 into one clean commit**.

**Solution**:
```bash
# Interactive rebase last 5 commits
git rebase -i HEAD~5

# Editor opens with:
pick f1 Fix typo
pick f2 Add feature
pick f3 Fix bug
pick f4 Fix another typo
pick f5 More work on feature

# Change to:
pick f2 Add feature
squash f1 Fix typo
squash f5 More work on feature
pick f3 Fix bug
pick f4 Fix another typo

# Save and close
# Git will combine f1, f2, f5 into one commit
# You'll be prompted to write a new commit message

# Result:
# c1 Add complete feature (combined f1, f2, f5)
# c2 Fix bug
# c3 Fix another typo
```

**Commands in interactive rebase**:
- `pick` = keep commit as-is
- `squash` = combine with previous commit
- `reword` = change commit message
- `edit` = pause to edit commit
- `drop` = remove commit

---

### Scenario B: Cherry-Pick Specific Commits

**Situation**: You want specific commits from another branch.

```bash
# Feature branch has commits A, B, C, D
# You only want B and D on main

git checkout main
git cherry-pick <commit-B-hash>
git cherry-pick <commit-D-hash>
```

**With conflict**:
```bash
git cherry-pick abc123
# CONFLICT!

# Resolve conflict
# Then:
git add .
git cherry-pick --continue

# Or abort:
git cherry-pick --abort
```

---

### Scenario C: Recover Lost Commits

**Situation**: You did `git reset --hard` and lost commits!

**Solution - Use reflog**:
```bash
# View reflog (history of HEAD movements)
git reflog

# Output:
# f60918e HEAD@{0}: reset: moving to HEAD~1
# a1b2c3d HEAD@{1}: commit: Important work
# f455814 HEAD@{2}: commit: add comment

# Recover the lost commit
git checkout a1b2c3d

# Or create a branch from it
git branch recovered-work a1b2c3d

# Or reset main to it
git checkout main
git reset --hard a1b2c3d
```

**Reflog is kept for 90 days by default.**

---

### Scenario D: Split a Large Commit

**Situation**: You made one huge commit, want to split into smaller ones.

```bash
# Reset to before the commit (keep changes)
git reset --soft HEAD~1

# Now changes are unstaged
git reset HEAD

# Stage and commit in pieces
git add file1.js
git commit -m "Add feature A"

git add file2.js file3.js
git commit -m "Add feature B"

git add file4.js
git commit -m "Add feature C"
```

---

### Scenario E: Rebase with Conflicts on Multiple Commits

**Situation**: Rebasing 10 commits, conflicts on commit 3, 5, and 8.

```bash
git pull --rebase origin main

# CONFLICT on commit 3
# Resolve conflict
git add .
git rebase --continue

# CONFLICT on commit 5
# Resolve conflict
git add .
git rebase --continue

# CONFLICT on commit 8
# Resolve conflict
git add .
git rebase --continue

# Finally done!
git push origin main
```

**If it gets too messy**:
```bash
# Abort and use merge instead
git rebase --abort
git pull --merge origin main
```

---

## Visual Examples

### Example 1: Clean Rebase

**Before**:
```
Remote:  A → B → C → D
Local:   A → B → X → Y
```

**After `git pull --rebase`**:
```
Both:    A → B → C → D → X' → Y'
```

Your commits (X, Y) are replayed on top of remote commits (C, D).

---

### Example 2: Merge Commit

**Before**:
```
Remote:  A → B → C → D
Local:   A → B → X → Y
```

**After `git pull --merge`**:
```
         A → B → C → D ←---┐
                  \         M (merge commit)
                   X → Y ←-┘
```

Creates a merge commit (M) that combines both histories.

---

### Example 3: Conflict During Rebase

**Before**:
```
Remote:  A → B → C (modified file.js line 5)
Local:   A → B → X (modified file.js line 5)
```

**During rebase**:
```
A → B → C → [CONFLICT - can't apply X]
```

**After resolving and continuing**:
```
A → B → C → X' (your changes integrated)
```

---

## Best Practices

### 1. Choose Your Strategy and Stick to It

**For solo projects**:
```bash
git config pull.rebase true
```
Use rebase for clean history.

**For team projects**:
```bash
git config pull.rebase false
```
Use merge to preserve collaboration history.

---

### 2. Commit Often, Push Regularly

```bash
# Don't do this:
# Work for 3 days
# Make one giant commit
# Try to push

# Do this instead:
# Work for 1 hour
git add .
git commit -m "Descriptive message"
# Push at end of day
git push
```

---

### 3. Use Branches for Everything

```bash
# Not this:
git checkout main
# make changes
git commit
git push  # Might conflict!

# This:
git checkout -b feature/my-work
# make changes
git commit
git push -u origin feature/my-work
# Create PR
# Merge on GitHub
# Update main locally
```

---

### 4. Pull Before Starting, Push When Done

```bash
# Morning:
git checkout main
git pull origin main

# Throughout day:
git checkout -b feature/today-work
# work work work
git add .
git commit -m "Progress"

# End of day:
git push -u origin feature/today-work
```

---

### 5. Communicate About Force Pushes

```bash
# NEVER do this on shared branches:
git push --force

# If you absolutely must:
# 1. Tell your team first
# 2. Make sure no one else has changes
# 3. Use --force-with-lease (safer)
git push --force-with-lease
```

---

### 6. Use Descriptive Commit Messages

```bash
# Bad:
git commit -m "fix"
git commit -m "updates"
git commit -m "asdf"

# Good:
git commit -m "Fix login button alignment on mobile"
git commit -m "Add user authentication with JWT"
git commit -m "Update README with installation instructions"
```

---

### 7. Review Changes Before Committing

```bash
# See what changed
git diff

# See what will be committed
git diff --staged

# Review each file
git add -p  # Interactive staging
```

---

### 8. Keep Main Branch Protected

On GitHub:
- Require pull requests
- Require reviews
- Require passing tests
- Don't allow force pushes
- Don't allow deletions

---

## Summary Cheat Sheet

### When You See "Divergent Branches"

```bash
# Quick fix (rebase - clean history):
git pull --rebase origin main

# Or (merge - preserve history):
git pull --merge origin main

# Then:
git push origin main
```

### When You See "Merge Conflict"

```bash
# 1. Open conflicted file
# 2. Find <<<<<<< ======= >>>>>>>
# 3. Edit to keep what you want
# 4. Remove conflict markers
# 5. Save file
git add <file>
# 6. Continue
git rebase --continue  # if rebasing
git commit             # if merging
# 7. Push
git push origin main
```

### Prevention

```bash
# Daily routine:
git checkout main
git pull origin main
git checkout -b feature/my-work
# work, commit, push feature branch
# create PR, merge on GitHub
git checkout main
git pull origin main
```

---

## Conclusion

**Key Takeaways**:

1. **Divergent branches** happen when local and remote have different commits
2. **Rebase** creates clean linear history (good for solo work)
3. **Merge** preserves all history (good for teams)
4. **Conflicts** occur when same lines are edited differently
5. **Prevention** is better than resolution - use branches and pull often
6. **Communication** with your team prevents most issues
7. **Don't panic** - Git rarely loses data, most mistakes are recoverable

**Remember**:
- `git reflog` is your friend for recovering "lost" work
- `git status` tells you what state you're in
- When in doubt, create a backup branch first!

Happy collaborating! 🚀
