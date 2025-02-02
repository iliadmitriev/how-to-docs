# GIT

## Clean commit history

Clean commit history from dangling commits

```bash
git reflog expire --expire=now --all
git gc --prune=now
```

```bash
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git repack -a -d
```

## Git initial setup

```bash
git config --global user.name "Ilia Dmitriev"
git config --global user.email "ilia.dmitriev@gmail.com"
git config --global core.pager ""
```

## Cherry-pick commits from another remote

```bash
# clone repository
git clone git@github.com:kubernetes/minikube.git

# checkout target branch
cd minikube
git checkout v1.22.0

# fetch remote branch
git fetch git@github.com:zhan9san/minikube.git feature/ingress-mac

# cherry-pick commits
git cherry-pick b9802818b68b769edd9489e9d1d4050943005760
```
