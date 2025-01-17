---
id: zsh_install
aliases: []
tags: []
---

# zsh

## install zsh with dependencies

```bash
sudo apt install zsh zsh-antigen zsh-doc zsh-syntax-highlighting zsh-autosuggestions
```

## set zsh as default shell

```bash
chsh -s /usr/bin/zsh
```

and don't forget to restart your gnome session to apply changes

on first start press 2
to populate `~/.zshrc` with default recommended configuration

## install oh-my-zsh dependencies

```bash
sudo apt install git curl
```

## install oh-my-zsh

using https://ohmyz.sh/#install install `oh-my-zsh`

```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## choose theme

```bash
omz theme list
omz theme use theme
omz theme set theme
```

edit file `~/.zshrc` to set defautl theme

```bash
gedit ~/.zshrc
```

edit line `ZSH_THEME="robbyrussell"`

set to desirable theme: `michelebologna`

more themes here https://github.com/ohmyzsh/ohmyzsh/wiki/Themes

add to the end of file:

```
source /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh
```

or just run script

```bash
omz theme list
omz theme set michelebologna

# or
sed -i.bak s/ZSH_THEME=".*"/ZSH_THEME=\"michelebologna\"/ ~/.zshrc
```

```bash
>>~/.zshrc<<_EOF_
source /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh
_EOF_
```

## install nerd fonts

### using `font-manager`

```bash
# install font mamanger
sudo apt install font-manager
```

download nerd fonts `Jetbrains Mono` from https://www.nerdfonts.com/font-downloads

unpack somewhere

install fonts with font-manager

set defualt font for teminal `JetBrainsMonoNL Nerd Font Mono`

### yum based repo

[Fedora 41 docs](https://docs.fedoraproject.org/en-US/quick-docs/fonts/)

## syntax highlight

macOS

```bash
brew install zsh-fast-syntax-highlighting
>>~/.zshrc<<_EOF_
source ${HOMEBREW_PREFIX}/share/zsh-fast-syntax-highlighting/fast-syntax-highlighting.plugin.zsh
_EOF_
```

Linux

```bash
apt install zsh-fast-syntax-highlighting
source /usr/share/zsh-fast-syntax-highlighting/fast-syntax-highlighting.plugin.zsh
```

fix slow pasting with zsh-autosuggestions [see](https://github.com/zsh-users/zsh-autosuggestions/issues/238#issuecomment-389324292)

```zsh
# This speeds up pasting w/ autosuggest
# https://github.com/zsh-users/zsh-autosuggestions/issues/238
pasteinit() {
  OLD_SELF_INSERT=${${(s.:.)widgets[self-insert]}[2,3]}
  zle -N self-insert url-quote-magic # I wonder if you'd need `.url-quote-magic`?
}

pastefinish() {
  zle -N self-insert $OLD_SELF_INSERT
}
zstyle :bracketed-paste-magic paste-init pasteinit
zstyle :bracketed-paste-magic paste-finish pastefinish
```
