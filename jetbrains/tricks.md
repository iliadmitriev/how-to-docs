# add fsnotifier for aarch64

```bash
#!/bin/sh

set -e

for f in fsnotifier.h \
         inotify.c \
         main.c \
         make.sh \
         util.c
do
  curl \
    "http://git.jetbrains.org/?p=idea/community.git;a=blob_plain;f=native/fsNotifier/linux/$f;hb=HEAD" \
    -o $f
done

bash make.sh
```

copy file `fsnotifier` to your jetbrain directory `/opt/pycharm-community-2022.2.3/bin/fsnotifier`
