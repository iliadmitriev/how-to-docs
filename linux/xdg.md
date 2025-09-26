# XDG Base Directory Specification

## User Directories

| Variable | Description | Default Value |
|----------|-------------|---------------|
| `XDG_CONFIG_HOME` | User-specific configurations (analogous to `/etc`) | `$HOME/.config` |
| `XDG_CACHE_HOME` | User-specific non-essential (cached) data (analogous to `/var/cache`) | `$HOME/.cache` |
| `XDG_DATA_HOME` | User-specific data files (analogous to `/usr/share`) | `$HOME/.local/share` |
| `XDG_STATE_HOME` | User-specific state files (analogous to `/var/lib`) | `$HOME/.local/state` |
| `XDG_RUNTIME_DIR` | Non-essential user-specific data (sockets, named pipes, etc.) | No default |

### Special Notes for `XDG_RUNTIME_DIR`:
- ⚠️ Not required to have a default value; warnings should be issued if not set
- 🔒 Must be owned by the user with access mode `0700`
- 💾 Filesystem must be fully featured by OS standards
- 🖥️ Must be on the local filesystem
- 🧹 May be subject to periodic cleanup
- ⏰ Modified every 6 hours or set sticky bit for persistence
- 🔐 Can only exist for the duration of the user's login
- 📦 Should not store large files (may be mounted as tmpfs)
- 🔧 `pam_systemd` sets this to `/run/user/$UID`

## System Directories

| Variable | Description | Default Value |
|----------|-------------|---------------|
| `XDG_DATA_DIRS` | Colon-separated list of directories (analogous to `PATH`) | `/usr/local/share:/usr/share` |
| `XDG_CONFIG_DIRS` | Colon-separated list of directories (analogous to `PATH`) | `/etc/xdg` |
