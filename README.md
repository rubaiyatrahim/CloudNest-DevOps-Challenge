# CloudNest DevOps Challenge

**Student Name:** Md. Rubaiyat Rahim <br/>
**Batch:** DevOps Batch 14 <br/>
**Assignment Title:** The Friday Night Fix

## Task 1: Starting Fresh

To keep new work separate from the main branch, we need to create and switch to a new feature branch.

```
git checkout -b feature/new-client-demo
```

_Why:_ This creates a safe, isolated branch (`feature/new-client-demo`) based on the most up-to-date version of `main`. If things break, `main` remains untouched. <br/>

## Task 2: Interrupted Work

We are halfway through writing code when the urgent bug comes in. We must temporarily shelf our work without committing incomplete code.

```
# Make some incomplete changes
echo "console.log('work in progress');" > feature.js
git add feature.js
# Stash the work
git stash

# Fix the urgent bug
git checkout main
echo "Bug fixed" > bugfix.txt
git add bugfix.txt
git commit -m "Fix critical client bug"

# Restore the incomplete work
git checkout feature/new-client-demo
git stash pop
git commit -m "WIP: Feature for client demo"
```

_Why:_ `git stash` safely stores our modified tracked files and staged changes on a stack of unfinished changes, giving us a clean working directory to address the urgent bug. <br/>
![Task 2](screenshots/task2.png)

## Task 3: Cleaning the History

Nadia wants to see two different ways of bringing a feature branch up to date: a rebase (linear history) and a merge (preserved history). So, we duplicate our current feature branch so we can demonstrate both methods side-by-side.

```
git checkout feature/new-client-demo
git branch demo-rebase
git branch demo-merge

# The Merge Approach (Preserves history)
git checkout demo-merge
git merge main
# (If a text editor opens, save and close it to accept the merge commit)

# The Rebase Approach (Linear history)
git checkout demo-rebase
git rebase main
```

_The Merge Approach Result:_ This creates a "merge commit." It preserves the exact chronological history and shows that the feature branch lived independently before being joined back.<br/>
![Merge](screenshots/task3-1.png)
_The Rebase Approach Result:_ This rewinds our feature branch commits, pulls in the new main commits, and replays our feature commits on top. It looks like we wrote our feature after the latest main updates, keeping a clean, straight line of history.<br/>
![Rebase](screenshots/task3-2.png)

## Task 4: The Embarrassing Message

Fix a bad commit message.

```
git checkout feature/new-client-demo

# Create a file named asdf.txt
git add asdf.txt
git commit -m "asdf fix"

# Fix the message
git commit --amend -m "Fix database connection timeout issue"
```
