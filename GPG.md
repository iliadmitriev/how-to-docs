# How-To setup pgp commit signing using git/github

this manual for macOS (tested on Big Sur)

## installation

1. install Gnu Version of GPG for macOS
```shell
brew install gnupg
brew install pinentry-mac
```

1.1. Find pinentry-mac installation path
```shell
% which pinentry-mac  
/opt/homebrew/bin/pinentry-mac
```
1.2. Setup pinentry
```shell
cat >> ~/.gnupg/gpg-agent.conf << EOF_
pinentry-program /opt/homebrew/bin/pinentry-mac
EOF_
```
1.3. restart `gpg-agent`
```shell
killall -1 gpg-agent
```

2. Generate private and public part of key
```shell
gpg --gen-key
```
Enter pass phrase twice (should leave it empty in order PyCharm work)

3. Print out keys list
```shell
gpg --list-secret-keys --keyid-format LONG
```

```text
/Users/dmitriev/.gnupg/pubring.kbx
----------------------------------
sec   rsa3072/5B34A94A1139F75E 2021-03-04 [SC] [   годен до: 2023-03-04]
      B15A5B729A67752D7AA0F5645B34A94A1139F75E
uid               [  абсолютно ] Ilia Dmitriev <ilia.dmitriev@gmail.com>
ssb   rsa3072/318B84706760C3D7 2021-03-04 [E] [   годен до: 2023-03-04]
```

4. add this key to git config
`git config --global user.signingkey <key id>`
```shell
git config --global user.signingkey 5B34A94A1139F75E
```

5. add variable GPG_TTY to your shell
this variable points to your tty
```shell
echo 'export GPG_TTY=$(tty)' >> ~/.zshrc
source ~/.zshrc
env | grep GPG_TTY
```

6. print out public key part
```shell
gpg --armor --export ilia.dmitriev@gmail.com
```

7. add public key part to github setting
* https://github.com/settings/keys
* press `New GPG key` button 
* paste your public key and press `Add GPG key`


## git command line usage

1. use `-S` option to sing your commit explicitly
```shell
git commit -S -m 'Signed commit'
```

2. show signature in commit log
```shell
git log --show-signature
```

3. show signature of defined commit
```shell
git show <commit id> --show-signature
```

## PyCharm/IntelliJ IDEA

1. disable tty and activate agent daemon
```shell
   cat >> ~/.gnupg/pgp.conf << _EOF
   no-tty
   use-agent
   _EOF
```

2. to cache key pass phrase run once
```shell
gpg --status-fd=2 -bsau <keyid> << _EOF
Test message
_EOF
```
Enter pass phrase

3. make key trustable
```shell
(echo 5; echo y; echo save) | gpg --command-fd 0 --no-tty --no-greeting -q --edit-key <key id> trust
```

4. enable all commit signing
```shell
git config --global commit.gpgsign true
```
