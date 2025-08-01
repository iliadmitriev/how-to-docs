# Mac OS X tricks

## Fix application

```bash
xattr -cr /Applications/NewApp.app
xcode-select --install
sudo codesign --force --deep --sign - /Applications/NewApp.app
```

## Reinstall Rosetta 2

```bash
softwareupdate --install-rosetta
```

## Start screensaver

```bash
open -b com.apple.ScreenSaver.Engine
```
