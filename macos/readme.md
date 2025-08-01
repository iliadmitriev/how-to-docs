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

## List all applications bundle identifiers

```bash
for app in /Applications/*.app; do
    if [[ -d "$app" ]]; then
        bundle_id=$(defaults read "${app}/Contents/Info.plist" CFBundleIdentifier 2>/dev/null)
        if [[ -n "$bundle_id" ]]; then
            echo "$app: $bundle_id"
        fi
    fi
done
```
