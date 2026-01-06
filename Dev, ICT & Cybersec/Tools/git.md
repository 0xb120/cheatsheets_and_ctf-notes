---
Description: Git is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.
URL: https://git-scm.com/book/en/v2/Getting-Started-The-Command-Line
---

### git main commands

| Command | Description |
| --- | --- |
| git add \<file\> | Add file to a commit |
| git add \<file\> | Remove file from a commit |
| git commit -m \<name\> | Create the new commit |
| git branch | List branches |
| git checkout -b \<branch_name\> | Create a new branch and move to it |
| git checkout \<branch_name\> | Switch to a branch |
| git push origin \<branch_name\> | Update a branch remotely |
| git pull origin \<branch_name\> | Update a branch locally from remote |
| git status | Show the status of the commit |
| git merge \<branch1\> \<branch2\> | Merge branch2 to branch1 |
| git log | Show commit log |
| git show \<id\> | Show the commit with that specific id |
| git checkout - | Go to the latest checkout |
| git checkout \<id\> | Go to checkout id |

## master to main

```bash
git branch -m master main 
git push -u origin main 
git remote set-head origin main
```

## Clone via SSH

```bash
git clone git@github.com:username/your-repository.git
```

## Push via SSH

```bash
git remote add origin git@github.com:MaOutis/maoutis.github.io.git
git remote set-url origin git@github.com:MaOutis/maoutis.github.io.git
```

## grep inside all the existing commits

```bash
git grep "token:" $(git rev-list --all)
```

## patch diffing

```bash
git clone <url>
git tag | grep v1
...
v1.1
v1.0
...
git diff v1.1 v1.0
...
git diff v1.1 v1.0 > patch.diff
grep +++ patch.diff
```


---

# External tools integrating with git

- [gitleaks](https://github.com/gitleaks/gitleaks)
- [git-dumper](https://github.com/arthaud/git-dumper)
- [gitDump](https://github.com/Ebryx/GitDump)

---


# Using git as an attack technique

- [Commit Stomping](../../Readwise/Articles/Erik%20-%20Last%20Week%20in%20Security%20(LWiS)%20-%202025-05-19.md)
- Git (and GitHub) evasion techniques used in [the TJ Actions Changed Files GitHub Actions Compromise](../../Readwise/Articles/Blog%20–%20Snyk%20-%20Reconstructing%20the%20TJ%20Actions%20Changed%20Files%20GitHub%20Actions%20Compromise.md)