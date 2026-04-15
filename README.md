# wf-demos

Demo recording and archive tool for [Warfork](https://store.steampowered.com/app/671610/Warfork/) race runs.

Automatically cycles through a 10-slot rolling demo buffer when you join a server, and gives you an `fzf`-powered CLI to save, browse, play, and manage your favorite runs.

## Features

- **Auto-recording** — demos start automatically every time you press your join key
- **10-slot rolling buffer** — slots cycle `run_00` → `run_09` → `run_00`, overwriting the oldest
- **Interactive picker** — `fzf` list shows map name, duration, and date for each demo
- **Favorites archive** — save good runs with auto-generated names (`mapname_Xm00s_YYYYMMDD.wfdz22`)
- **Safe deletion** — trash folder instead of instant delete; `clear-temp` to confirm permanent removal
- **In-game feedback** — Warfork console echoes which slot is being recorded
- Works with **bash**, **fish**, **zsh**, and any shell

## Requirements

| Dependency | Install (Arch/CachyOS) | Install (Ubuntu/Debian) |
|-----------|----------------------|------------------------|
| [fzf](https://github.com/junegunn/fzf) | `paru -S fzf` | `apt install fzf` |
| Python 3 | pre-installed | pre-installed |
| Steam | `paru -S steam` | `apt install steam` |
| Warfork | [Steam page](https://store.steampowered.com/app/671610/Warfork/) | same |

## Install

```bash
git clone https://github.com/mikul-/wf-demos.git
cd wf-demos
bash install.sh
```

The installer will ask you:
- **Warfork demos directory** — where Warfork stores `.wfdz*` files (default auto-detected)
- **Archive directory** — where to keep `favorites/` and `trash/` (default: `~/demos`)
- **Join keybinds** — keys that join the server and start a new recording slot (e.g. `4,F3`)
- **Practice mode keybinds** — keys that stop recording but don't start a new slot (e.g. `1,F5,F11`)

It then installs `~/.local/bin/wf-demos`, writes `~/.config/wf-demos/config`, and generates a custom `autoexec.cfg` for your Warfork mod directory.

## Usage

```
wf-demos                interactive menu
wf-demos save           fzf-pick a run from the rolling buffer → save to favorites
wf-demos list           browse favorites  (Enter = play,  Ctrl-D = move to trash)
wf-demos play run_05    play a rolling buffer slot directly
wf-demos clear-temp     permanently delete all files in trash/
```

### Typical workflow

1. Join a race server → demo starts recording automatically
2. Finish a run you like
3. Run `wf-demos` → pick **save** → select the run in fzf → it's archived
4. Later: `wf-demos list` to browse, Enter to watch a run in Warfork

## How the auto-recording works

The installer drops an `autoexec.cfg` into your Warfork mod directory. It sets up a 10-alias chain and overrides your join/practice-mode binds:

**Join key** (`4` by default):
```
stop; join; record run_XX; (advance slot counter)
```

**Practice mode key** (`1`, `F5`, `F11` by default):
```
stop; practicemode
```

The slot counter (`run_00` … `run_09`) advances each join and resets to `run_00` when Warfork starts. Oldest demos are silently overwritten.

## Configuration

`~/.config/wf-demos/config` is sourced by the script on every run. Edit it directly to change paths without reinstalling:

```bash
DEMO_DIR="${HOME}/.local/share/warfork-2.1/racemod_2.1/demos"
ARCHIVE_DIR="${HOME}/demos"
APPID=671610
WF_MOD=racemod_2.1
```

## File layout

| Path | Purpose |
|------|---------|
| `~/.local/bin/wf-demos` | Main CLI |
| `~/.local/bin/wf-demo-info` | Demo header parser (Python) |
| `~/.config/wf-demos/config` | User config (paths, app ID) |
| `~/.local/share/warfork-2.1/racemod_2.1/autoexec.cfg` | Auto-recording config (default path) |
| `~/demos/favorites/` | Saved demos |
| `~/demos/trash/` | Trashed demos (pending `clear-temp`) |

## Uninstall

```bash
rm ~/.local/bin/wf-demos ~/.local/bin/wf-demo-info
rm -rf ~/.config/wf-demos
# optionally keep your saved demos:
# rm -rf ~/demos
```

Remove the wf-demos lines from your `autoexec.cfg` to restore your original binds.

## Notes

- Demo playback launches Warfork via `steam -applaunch`. Steam must be running.
- The qfusion engine auto-prepends `demos/` to demo names — do not include it yourself.
- Warfork demo extensions look like `.wfdz22`; the version number may vary. The glob `*.wfdz*` handles all versions.
- The demo header parser reads the first 512 bytes of each file to extract map name, duration, and timestamp — no external libraries required.
