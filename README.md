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
