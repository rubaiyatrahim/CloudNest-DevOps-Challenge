# CloudNest DevOps Challenge

**Student Name:** Md. Rubaiyat Rahim <br/>
**Batch:** DevOps Batch 14 <br/>
**Assignment Title:** The Friday Night Fix

## Task 1: Starting Fresh

To keep new work separate from the main branch, we need to create and switch to a new feature branch.

```Git
git checkout -b feature/new-client-demo
```

_Why:_ This creates a safe, isolated branch (`feature/new-client-demo`) based on the most up-to-date version of `main`. If things break, `main` remains untouched. <br/>

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
