## Git initial setup

```shell
git config --global user.name "Ilia Dmitriev"
git config --global user.email "ilia.dmitriev@gmail.com"
git config --global core.pager ""
```

## Cherry-pick commits from another remote

```shell
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
