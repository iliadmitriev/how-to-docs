# Mac OS X tricks

Fix application

```bash
xattr -cr /Applications/NewApp.app
xcode-select --install
sudo codesign --force --deep --sign - /Applications/NewApp.app
```
