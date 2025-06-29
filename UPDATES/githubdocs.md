Great question! Here are the key reasons why we decided to create a new folder and new repository:

## �� **Primary Problem: Git History Pollution**

### The Core Issue:
- Your original repository had **large files (model.pth) committed as regular git files**
- These files were **100MB+ each** but stored directly in git history
- GitHub **rejects pushes** when files >100MB are not tracked by LFS
- Simply adding LFS tracking later **doesn't remove large files from history**

### What We Tried First:
1. **Git LFS tracking** - Didn't work because files were already in history
2. **Rebase operations** - Created conflicts and was very complex
3. **History rewriting** - Risky and time-consuming
4. **Force pushing** - Still failed due to large files in history

## 🟡 **Secondary Problems:**

### 1. **Divergent Branches**
- Local: 77 commits
- Remote: 65 commits  
- **77 + 65 = 142 different commits** to reconcile
- This created complex merge conflicts

### 2. **LFS Configuration Issues**
- Some files were tracked by LFS when they shouldn't be (config.yaml)
- Some large files weren't tracked when they should be
- Mixed LFS/non-LFS tracking caused conflicts

### 3. **Authentication and Permission Issues**
- SSH key problems
- Credential helper conflicts
- Repository access issues

## 🟢 **Why New Repository Was the Best Solution:**

### **Clean Slate Benefits:**
1. **No polluted history** - Fresh start with clean git history
2. **Proper LFS setup** - Large files tracked correctly from the beginning
3. **No conflicts** - No need to resolve 142+ divergent commits
4. **Guaranteed success** - No risk of push failures
5. **Time efficient** - Much faster than fixing complex history issues

### **Safety:**
- **Original repo preserved** - Your deployment stays working
- **No data loss** - All files copied exactly
- **Easy rollback** - Can always go back to original

## �� **Comparison:**

| Approach | Time | Risk | Success Rate | Complexity |
|----------|------|------|--------------|------------|
| Fix existing repo | Days | High | Low | Very High |
| New repository | Hours | Low | 100% | Low |

## 🎯 **The Decision Process:**

1. **Identified the root cause**: Large files in git history
2. **Tried multiple fixes**: All failed or were too complex
3. **Evaluated options**: Fix vs. Fresh start
4. **Chose fresh start**: Guaranteed success, minimal risk
5. **Executed cleanly**: Copy files, setup LFS, push

## �� **Key Lesson:**
**Git LFS must be configured BEFORE the first commit of large files.** Once large files are in git history, removing them requires rewriting history, which is complex and risky.

**The new repository approach was the most reliable, fastest, and safest solution to your specific problem.**


# Project Git & GitHub Troubleshooting Documentation

---

## Table of Contents
1. [Overview](#overview)
2. [Large Files and Git LFS](#large-files-and-git-lfs)
3. [Git Submodules vs. Regular Folders](#git-submodules-vs-regular-folders)
4. [Non-Fast-Forward Push Rejection](#non-fast-forward-push-rejection)
5. [Merge/Rebase Conflicts](#mergerebase-conflicts)
6. [Stale Swap Files During Commit](#stale-swap-files-during-commit)
7. [No Changes to Commit](#no-changes-to-commit)
8. [General Best Practices](#general-best-practices)
9. [Authentication Errors with GitHub](#authentication-errors-with-github)
10. [Using Homebrew to Install Tools](#using-homebrew-to-install-tools)
11. [How We Removed LFS Tracking to Fix Push Problems](#how-we-removed-lfs-tracking-to-fix-push-problems)
12. [Authentication Troubleshooting Analysis](#authentication-troubleshooting-analysis)
13. [Summary of Issues Faced and How They Were Solved](#summary-of-issues-faced-and-how-they-were-solved)
14. [Git Branches in GitHub: Concepts & Best Practices](#git-branches-in-github-concepts--best-practices)
15. [References](#references)

---

# Overview

This document summarizes the main issues encountered while managing your multilingual TTS project with git and GitHub, and provides step-by-step solutions, commands, and concepts for each. It is intended as a reference for you and your collaborators to avoid and resolve similar issues in the future.

---

# Large Files and Git LFS

### Issue
- GitHub rejected pushes due to files >100MB (e.g., model.pth).
- Some files were tracked by regular git instead of Git LFS, causing history bloat and push failures.

### Solution
- Use [Git Large File Storage (LFS)](https://git-lfs.github.com/) for large files.
- Only track large files (e.g., model.pth) with LFS, not small files (e.g., .npz).

#### Commands
```sh
# Install Git LFS (if not already)
git lfs install

# Track large files
git lfs track "Fastspeech2_HS/marathi/male/model/model.pth"
git lfs track "Fastspeech2_HS/english/male/model/model.pth"
git lfs track "Fastspeech2_HS/hindi/male/model/model.pth"
git lfs track "Fastspeech2_HS/ekalipi/male/model/model.pth"

# Add LFS tracking to git
git add .gitattributes

# Add large files (force if needed)
git add -f Fastspeech2_HS/marathi/male/model/model.pth

# Commit and push
git commit -m "Track large model files with LFS"
git push origin main
```

**Concept:**  
Git LFS replaces large files in your repo with lightweight pointers, storing the actual content on a separate LFS server.

---

# Git Submodules vs. Regular Folders

### Issue
- The `docs` folder was a git submodule, making it unbrowsable on GitHub and hard to update.

### Solution
- Remove the submodule and re-add `docs` as a regular folder.

#### Commands
```sh
# Remove submodule reference
git rm --cached docs
rm -rf .git/modules/docs
rm -f .gitmodules

# Move docs content out and back in if needed
cp -r docs docs_temp
rm -rf docs
mv docs_temp docs

# Add as regular folder
git add docs
git commit -m "Convert docs from submodule to regular folder"
git push origin main
```

**Concept:**  
A submodule is a separate git repo inside your main repo. For documentation, a regular folder is simpler and more accessible.

---

# Non-Fast-Forward Push Rejection

### Issue
- Pushes were rejected with:
  ```
  ! [rejected] main -> main (non-fast-forward)
  error: failed to push some refs to 'github.com:...'
  hint: Updates were rejected because the tip of your current branch is behind
  ```

### Solution
- Pull and rebase to integrate remote changes, then push.

#### Commands
```sh
git pull --rebase origin main
# Resolve any conflicts as prompted
git push origin main
```

**Concept:**  
A non-fast-forward error means your local branch is behind the remote. You must update your branch before pushing.

---

# Merge/Rebase Conflicts

### Issue
- During rebase, conflicts occurred (e.g., file deleted in remote, modified locally).

### Solution
- Decide whether to keep your version or accept the remote change.
- Mark as resolved and continue the rebase.

#### Commands
```sh
# To keep your version
git add <conflicted-file>
git rebase --continue

# To accept the remote deletion
git rm <conflicted-file>
git rebase --continue
```

**Concept:**  
A conflict happens when the same file is changed in different ways in two branches. You must resolve it manually.

---

# Stale Swap Files During Commit

### Issue
- Editor (e.g., vim) left a `.swp` file, causing a prompt during commit.

### Solution
- Delete the swap file when prompted.

**Concept:**  
Swap files are temporary files created by editors to prevent data loss. If you see a prompt, it's safe to delete if no other editing session is open.

---

# No Changes to Commit

### Issue
- Running `git add` and `git commit` but seeing `nothing to commit, working tree clean`.

### Solution
- Ensure you have saved your changes in your editor before adding/committing.
- Use `git status` to check for unstaged changes.

#### Commands
```sh
git status
```

---

# General Best Practices

- Always use `git status` to check your repo state.
- Use `.gitattributes` to control LFS tracking.
- Use `.gitignore` to avoid committing unnecessary files.
- For documentation, keep folders as regular directories, not submodules.
- For large files, always use Git LFS.

---

# Authentication Errors with GitHub

### Common Symptoms
- `Permission denied (publickey)`
- `fatal: Could not read from remote repository.`
- `Authentication failed for ...`
- Push/pull/clone operations fail due to authentication.

### Root Causes
- SSH key not added to GitHub account.
- Wrong remote URL (HTTPS vs SSH).
- SSH agent not running or not loaded with your key.
- GitHub account permissions (no write access).
- Outdated or missing credential helper.
- Using HTTPS and not providing a Personal Access Token (PAT) when prompted.

### How We Solved It

#### 1. Checking SSH Key Setup
- **Generate a new SSH key (if needed):**
  ```sh
  ssh-keygen -t ed25519 -C "your_email@example.com"
  ```
- **Add your SSH key to the ssh-agent:**
  ```sh
  eval "$(ssh-agent -s)"
  ssh-add ~/.ssh/id_ed25519
  ```
- **Copy your public key to GitHub:**
  ```sh
  cat ~/.ssh/id_ed25519.pub
  ```
  - Go to [GitHub SSH Keys](https://github.com/settings/keys) and add the key.

#### 2. Verifying SSH Connection
```sh
ssh -T git@github.com
```
- You should see:  
  `Hi <username>! You've successfully authenticated...`

#### 3. Ensuring Correct Remote URL
- **Check your remote:**
  ```sh
  git remote -v
  ```
- **Set to SSH if needed:**
  ```sh
  git remote set-url origin git@github.com:yourusername/yourrepo.git
  ```

#### 4. Using HTTPS with Personal Access Token (PAT)
- If using HTTPS, generate a [GitHub PAT](https://github.com/settings/tokens) and use it as your password when prompted.

#### 5. Credential Helper Issues
- **Clear old credentials:**
  ```sh
  git credential-cache exit
  git credential reject
  ```
- **Set up credential helper:**
  ```sh
  git config --global credential.helper osxkeychain  # for macOS
  ```

---

# Using Homebrew to Install Tools

### What is Homebrew?
- Homebrew is a package manager for macOS (and Linux) that makes it easy to install and manage software.

### How We Used It

#### 1. Install Homebrew (if not already installed)
```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### 2. Install Git LFS with Homebrew
```sh
brew install git-lfs
git lfs install
```

#### 3. Install/Update Git
```sh
brew install git
brew upgrade git
```

---

# How We Removed LFS Tracking to Fix Push Problems

### The Problem
- You could not push to GitHub because some files (e.g., model files) were too large and were not properly tracked by Git LFS, or were tracked by LFS after already being committed as regular files.
- GitHub rejects files >100MB unless tracked by LFS from the start.
- Simply running `git lfs untrack` or changing `.gitattributes` does **not** remove large files from git history.

### Why This Happens
- If a large file is committed as a regular file, it is stored in git history, even if you later add LFS tracking.
- Removing LFS tracking or changing `.gitattributes` does not rewrite history; the large file remains in the repo's history, causing push failures.

### How We Solved It

#### 1. Remove LFS Tracking for Unnecessary Files
- Edit `.gitattributes` to only track the files you want with LFS (e.g., only the four large male model files).
- Remove LFS tracking for other files:
  ```sh
  git lfs untrack "Fastspeech2_HS/other_language/other_file"
  git add .gitattributes
  git commit -m "Untrack unnecessary files from LFS"
  ```

#### 2. Remove Large Files from Git History
- If a file was already committed as a regular file (not LFS), you must **rewrite git history** to remove it.
- The most reliable way: **Start a new, clean repository**:
  1. Copy your project to a new folder.
  2. Remove the `.git` directory.
  3. Re-initialize git and LFS.
  4. Track only the necessary large files with LFS.
  5. Add, commit, and push everything to a new GitHub repo.

  **Commands:**
  ```sh
  # In the new folder
  git init
  git lfs install
  git lfs track "Fastspeech2_HS/marathi/male/model/model.pth"
  # ...repeat for other large files...
  git add .gitattributes
  git add .
  git commit -m "Initial clean commit: LFS for large model files only"
  git remote add origin git@github.com:yourusername/yourrepo.git
  git push -u origin main
  ```

#### 3. (Alternative) Use BFG Repo-Cleaner or git filter-repo
- If you want to keep your repo history, use [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/) or `git filter-repo` to remove large files from history.
- This is more advanced and was not the main approach used in your case.

### Key Concepts

- **LFS tracking must be set up before the first commit of a large file.**
- **Removing LFS tracking does not remove files from history.**
- **To truly remove large files, you must rewrite history or start fresh.**
- **A clean repo with only the right files tracked by LFS is the most reliable fix.**

### Summary Table

| Problem/Symptom                | Solution/Command(s)                                                                 |
|--------------------------------|-------------------------------------------------------------------------------------|
| Large file push rejected       | Remove LFS tracking for small files, rewrite history or start new repo, track only large files with LFS |
| Untrack file from LFS          | `git lfs untrack "path/to/file"`<br>`git add .gitattributes`<br>`git commit -m "Untrack file"` |
| Remove large file from history | Start new repo, or use BFG Repo-Cleaner / git filter-repo                           |

### References
- [GitHub Docs: Removing files from a repository's history](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)
- [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/)
- [Git LFS Documentation](https://git-lfs.github.com/)

---

# Authentication Troubleshooting Analysis

### Symptoms Observed
- `error: Authentication error: Authentication required: You must have push access to verify locks`
- `ERROR: Permission to lavanyamahalle15/lavanya_tts_final.git denied to lavanyamahalle.`
- `git@github.com: Permission denied (publickey).`
- `Agent pid ...`
- `ssh-add ~/.ssh/id_ed25519_lavanyamahalle15: No such file or directory`
- Repeated attempts to push, set remote, and use SSH keys.

### What Happened?

#### 1. Credential Helper and HTTPS Authentication
- You tried to erase cached credentials with:
  ```sh
  git credential-osxkeychain erase
  ```
- You attempted to push over HTTPS and got authentication errors, likely because:
  - The credential helper had old or incorrect credentials.
  - You may not have had the correct Personal Access Token (PAT) set up for HTTPS.

#### 2. Switching to SSH
- You changed the remote URL to SSH:
  ```sh
  git remote set-url origin git@github.com:lavanyamahalle15/lavanya_tts_final.git
  ```
- You then got `Permission denied (publickey)` errors, which means SSH authentication failed.

#### 3. SSH Key Generation and Management
- You generated a new SSH key with:
  ```sh
  ssh-keygen -t ed25519 -C "lavanyamahalle40@gmail.com"
  ```
- You tried to add a key with:
  ```sh
  ssh-add ~/.ssh/id_ed25519_lavanyamahalle15
  ```
  but the file did not exist.
- You then added the correct key:
  ```sh
  ssh-add ~/.ssh/id_ed25519
  ```
  and saw `Identity added: ...`

#### 4. Testing SSH Connection
- You ran:
  ```sh
  ssh -T git@github.com
  ```
  and still got `Permission denied (publickey).`

#### 5. Debugging SSH
- You used verbose SSH to see which keys were being tried:
  ```sh
  ssh -vT git@github.com
  ```
- The output showed:
  - The correct key (`/Users/kishorbapat/.ssh/id_ed25519`) was offered.
  - But authentication still failed.

#### 6. Key Not Added to GitHub
- You displayed your public key:
  ```sh
  cat ~/.ssh/id_ed25519.pub
  ```
- **If you did not add this key to your GitHub account**, authentication would still fail.

### Key Concepts and Solutions

#### A. SSH Key Authentication
- **You must add your public SSH key to your GitHub account** ([GitHub SSH Keys](https://github.com/settings/keys)).
- The private key must be loaded into your ssh-agent (`ssh-add`).
- The key must match the one registered on GitHub.

#### B. Common Pitfalls
- Generating a new key but not adding it to GitHub.
- Adding the wrong key to the agent.
- Using the wrong remote URL (HTTPS vs SSH).
- Not having push access to the repository (check GitHub permissions).

#### C. Correct Steps to Fix
1. **Generate a new SSH key (if needed):**
   ```sh
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```
2. **Add the key to the agent:**
   ```sh
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/id_ed25519
   ```
3. **Add the public key to GitHub:**
   - Copy the output of:
     ```sh
     cat ~/.ssh/id_ed25519.pub
     ```
   - Go to [GitHub SSH Keys](https://github.com/settings/keys) and add it.
4. **Test the connection:**
   ```sh
   ssh -T git@github.com
   ```
   - You should see a welcome message, not a permission denied error.
5. **Set the correct remote:**
   ```sh
   git remote set-url origin git@github.com:yourusername/yourrepo.git
   ```
6. **Push again:**
   ```sh
   git push origin main
   ```

### Summary Table

| Step/Command                                      | Purpose/Result                                      |
|---------------------------------------------------|-----------------------------------------------------|
| `git credential-osxkeychain erase`                | Remove cached HTTPS credentials                     |
| `git remote set-url origin ...`                   | Switch between HTTPS and SSH remotes                |
| `ssh-keygen -t ed25519 -C ...`                    | Generate a new SSH key                              |
| `ssh-add ~/.ssh/id_ed25519`                       | Add private key to ssh-agent                        |
| `cat ~/.ssh/id_ed25519.pub`                       | Show public key to add to GitHub                    |
| `ssh -T git@github.com`                           | Test SSH authentication to GitHub                   |
| `ssh -vT git@github.com`                          | Debug SSH authentication                            |

### Final Checklist for Authentication Issues

- [x] SSH key generated and added to agent
- [x] Public key added to GitHub account
- [x] Correct remote URL (SSH or HTTPS)
- [x] Sufficient permissions on the repo
- [x] Credential helper cleared if switching from HTTPS

**If you follow these steps and still get errors, double-check:**
- The key you added to GitHub matches the one you're using locally.
- You're using the correct GitHub account.
- You have write access to the repository.

# Summary of Issues Faced and How They Were Solved

## 1. Large File Push Rejection (GitHub 100MB Limit)
**Symptom:**  
- Push fails with error about files >100MB.

**Cause:**  
- Large files (e.g., model.pth) were committed as regular git files, not tracked by Git LFS.

**Solution:**  
- Use Git LFS to track only large files.
- If large files were already committed, start a new clean repo or use BFG Repo-Cleaner to remove them from history.

## 2. Git LFS Tracking/Untracking Confusion
**Symptom:**  
- LFS tracked too many files, or untracked files still caused push errors.

**Cause:**  
- LFS tracking was set after files were already committed, or `.gitattributes` was not properly configured.

**Solution:**  
- Edit `.gitattributes` to only track necessary files.
- Use `git lfs untrack` for small files.
- Start a new repo if history is polluted.

## 3. Submodule Issues with `docs` Folder
**Symptom:**  
- `docs` folder not browsable on GitHub, can't add/commit files normally.

**Cause:**  
- `docs` was a git submodule, not a regular folder.

**Solution:**  
- Remove submodule references, delete `.gitmodules`, and re-add `docs` as a regular folder.

## 4. Non-Fast-Forward Push Rejection
**Symptom:**  
- Push fails with "non-fast-forward" error.

**Cause:**  
- Local branch is behind the remote branch.

**Solution:**  
- Run `git pull --rebase origin main`, resolve any conflicts, then push.

## 5. Merge/Rebase Conflicts
**Symptom:**  
- Rebase or merge stops with conflict messages.

**Cause:**  
- Same file changed in different ways locally and remotely.

**Solution:**  
- Manually resolve conflicts, then run `git add <file>` and `git rebase --continue`.

## 6. Authentication Errors (SSH/HTTPS)
**Symptom:**  
- `Permission denied (publickey)` or `Authentication failed`.

**Cause:**  
- SSH key not added to GitHub, wrong remote URL, or missing permissions.

**Solution:**  
- Generate and add SSH key to GitHub, use correct remote, ensure permissions, and use credential helper if needed.

## 7. Credential Helper Issues
**Symptom:**  
- Old credentials used, can't authenticate.

**Cause:**  
- Credential helper cached old or wrong credentials.

**Solution:**  
- Clear credentials with `git credential-osxkeychain erase` and re-authenticate.

## 8. Stale Swap Files During Commit
**Symptom:**  
- Prompt about `.swp` file when editing commit message.

**Cause:**  
- Editor (e.g., vim) left a swap file from a previous session.

**Solution:**  
- Delete the swap file when prompted.

## 9. "Nothing to Commit" When Expecting Changes
**Symptom:**  
- `git commit` says "nothing to commit, working tree clean".

**Cause:**  
- File changes not saved in editor, or wrong directory.

**Solution:**  
- Save files, check with `git status`, then add and commit.

## 10. Directory Confusion
**Symptom:**  
- Commands run in the wrong project directory, causing confusion about repo state.

**Cause:**  
- Multiple project folders (e.g., `lavanya_tts3` vs `lavanya_tts3_clean`).

**Solution:**  
- Always check your current directory with `pwd` or `ls`, and use `cd` to switch as needed.

## 11. Homebrew and Tool Installation
**Symptom:**  
- Git LFS or git not installed.

**Cause:**  
- Missing dependencies.

**Solution:**  
- Use Homebrew to install:  
  ```sh
  brew install git-lfs
  brew install git
  ```

## 12. General Best Practices Learned
- Always check `git status` before and after changes.
- Use `.gitattributes` and `.gitignore` wisely.
- For documentation, use regular folders, not submodules.
- For large files, always use Git LFS from the start.
- For authentication, ensure SSH keys are set up and added to GitHub.

# Git Branches in GitHub: Concepts & Best Practices

## 🔴 What Are Branches in Git & GitHub?

A **branch** in Git is a pointer to a snapshot of your changes.
Git branches are lightweight and allow you to work on features, bug fixes, or experiments in isolation.

> Think of branches as different timelines in your project — you can work on a new feature without disturbing the main timeline.

### 🌳 Why Use Branches?

| Use Case         | Benefit                                   |
| ---------------- | ----------------------------------------- |
| Add new features | Without disturbing main (production) code |
| Bug fixes        | Fix and test before merging               |
| Experiment       | Try new ideas without risk                |
| Collaboration    | Team members can work in parallel         |

### 🏷️ Common Branch Names

* `main` or `master`: Default production/stable branch
* `dev`: Development branch
* `feature/xyz`: New features
* `bugfix/xyz`: Bug fixes
* `hotfix/xyz`: Quick critical fixes to production

### 🔧 Git Branch Commands

| Task                      | Command                                |
| ------------------------- | -------------------------------------- |
| View branches             | `git branch`                           |
| Create new branch         | `git branch branch_name`               |
| Switch to a branch        | `git checkout branch_name`             |
| Create + switch           | `git checkout -b branch_name`          |
| Rename branch             | `git branch -m old_name new_name`      |
| Delete local branch       | `git branch -d branch_name`            |
| Delete remote branch      | `git push origin --delete branch_name` |
| Push new branch to GitHub | `git push -u origin branch_name`       |
| Merge another branch      | `git merge branch_name`                |
| Rebase another branch     | `git rebase branch_name`               |

### 📘 Branching Workflow with GitHub

#### 1. **Create a New Branch**

```bash
git checkout -b feature/login
```

#### 2. **Do Your Work and Commit**

```bash
git add .
git commit -m "Added login feature"
```

#### 3. **Push to GitHub**

```bash
git push -u origin feature/login
```

#### 4. **Create a Pull Request (PR)**

* Go to GitHub repo
* Click "Compare & Pull Request"
* Review → Merge → Delete Branch (optional)

### ⚠️ Detached HEAD (What You Saw Earlier)

* Happens when you check out a specific **commit** instead of a **branch**.
* You're not on any branch, so changes may be lost if not saved.

✅ Always check out a **branch** before committing:

```bash
git checkout main
```

### 🔁 Merge vs Rebase

| Feature       | `merge`                          | `rebase`                      |
| ------------- | -------------------------------- | ----------------------------- |
| Keeps history | Yes (creates extra merge commit) | No (rewrites history)         |
| Use when      | Team work, safe merges           | Clean history, before pushing |

### 🌐 Syncing with Remote Branch

```bash
# Pull latest changes from origin/main into your current branch
git pull origin main
```

To avoid conflicts, always **pull before you push**.

### 🧼 Best Practices

* Never work directly on `main`. Use feature branches.
* Always pull before you push.
* Name branches clearly: `feature/signup-form`, `bugfix/typo-header`.
* Delete merged branches to keep repo clean.

### 📌 Visual Diagram

```
main
│
├───┐
│   └── feature/login
│        │
│        └─── Pull Request → Merge into main
│
└─── Continue Development
```

---

Let me know if you want a **PDF version**, **visual cheat sheet**, or **interactive GitHub tutorial**.

# References

- [Git LFS Documentation](https://git-lfs.github.com/)
- [GitHub Docs: Removing a submodule](https://docs.github.com/en/get-started/working-with-submodules/removing-a-submodule)
- [GitHub Docs: Resolving merge conflicts](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/about-merge-conflicts)
- [GitHub Docs: Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [GitHub Docs: Creating a personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
- [Homebrew](https://brew.sh/)

---

If you want this as a markdown file in your repo, I can generate and add it for you!  
Let me know if you want a step-by-step for BFG or filter-repo as well.



