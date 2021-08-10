# Install

MacOS (using brew)
```shell
brew install helm
```

# Configure

zsh completion
```shell
echo 'source <(helm completion zsh)' | cat >> ~/.zshrc
source ~/.zshrc
```

add repository
```shell
helm repo add stable https://charts.helm.sh/stable
```

update repository
```shell
helm repo update
```

# Usage 

Search for charts
```shell
helm search repo memcached
```

Install chart with name `mycache`
```shell
helm install mycache stable/memcached
```