# wf-demos

Demo recording and archive tool for [Warfork](https://store.steampowered.com/app/671610/Warfork/) race runs.

Automatically cycles through a 10-slot rolling demo buffer when you join a server, and gives you a CLI to save, browse, play, and manage your favorite runs.

- **Linux** — `wf-demos` (bash), interactive picker powered by `fzf`
- **Windows** — `wf-demos.py` / `wf-demos.bat` (Python), numbered terminal menu

## Features

- **Auto-recording** — demos start automatically every time you press your join key
- **10-slot rolling buffer** — slots cycle `run_00` → `run_09` → `run_00`, overwriting the oldest
- **Favorites archive** — save good runs with auto-generated names (`player_mapname_Xm00s_YYYYMMDD.wfdz22`)
- **Safe deletion** — trash folder instead of instant delete; `clear-temp` to confirm permanent removal
- **In-game feedback** — Warfork console echoes which slot is being recorded

---

## Requirements

### Linux

| Dependency | Arch/CachyOS | Ubuntu/Debian |
|-----------|--------------|---------------|
| [fzf](https://github.com/junegunn/fzf) | `paru -S fzf` | `apt install fzf` |
| Python 3 | pre-installed | pre-installed |
| bash 4+ | pre-installed | pre-installed |
| Steam + Warfork | `paru -S steam` | `apt install steam` |

### Windows

| Dependency | Notes |
|-----------|-------|
| Python 3.8+ | [python.org](https://www.python.org/downloads/) — tick "Add Python to PATH" |
| Steam + Warfork | [Steam page](https://store.steampowered.com/app/671610/Warfork/) |

No `fzf` needed — the Windows version uses a built-in numbered menu.

---

## Install

### Linux

```bash
git clone https://github.com/mikul-/wf-demos.git
cd wf-demos
bash install.sh
```

The installer will ask you:
- **Warfork demos directory** — where Warfork stores `.wfdz*` files (default auto-detected)
- **Archive directory** — where to keep `favorites/` and `trash/` (default: `~/demos`)
- **Join keybinds** — keys that join the server and start a new recording slot (e.g. `4,F3`)
- **Practice mode keybinds** — keys that stop recording but don't start a new slot (e.g. `1,F5`)

It then installs `~/.local/bin/wf-demos`, writes `~/.config/wf-demos/config`, and generates a custom `autoexec.cfg` for your Warfork mod directory.

### Windows

```
git clone https://github.com/mikul-/wf-demos.git
cd wf-demos
python wf-demos.py --setup
```

Or download the zip and run `wf-demos.bat` (which calls `python wf-demos.py`).

The setup wizard will ask the same questions as the Linux installer:
- **Warfork demos directory** — auto-detected from `Documents\My Games\Warfork 2.1\racemod_2.1\demos`
- **Archive directory** — where to keep `favorites\` and `trash\` (default: `Documents\wf-demos`)
- **Join keybinds** — comma-separated Warfork key names (e.g. `4,F3,ENTER`)
- **Practice mode keybinds** — keys for practice mode (e.g. `1,F5,F11`)

When an existing `autoexec.cfg` is found, setup offers to **merge** (insert/replace the wf-demos block and keep the rest), **print** (show the block so you can paste it manually), or **skip**.

Config is saved to `%APPDATA%\wf-demos\config`. Run setup again any time to change paths or keybinds.

---

## Usage

### Linux

```
wf-demos                interactive menu
wf-demos save           fzf-pick a run from the rolling buffer → save to favorites
wf-demos list           browse favorites  (Enter = play,  Ctrl-D = move to trash)
wf-demos play run_05    play a rolling buffer slot directly
wf-demos clear-temp     permanently delete all files in trash/
```

### Windows

```
wf-demos.bat                    interactive menu
wf-demos.bat save               numbered list of rolling buffer → pick to save
wf-demos.bat list               numbered list of favorites → pick to play or trash
wf-demos.bat play run_05        play a rolling buffer slot directly
wf-demos.bat clear-temp         permanently delete all files in trash\
```

Or call Python directly: `python wf-demos.py [command]`

The Windows menu uses a numbered picker instead of fzf. After choosing a favorite in `list`, a sub-menu lets you **1. Play**, **2. Move to trash**, or **q. Back**. Type `q` or press Ctrl-C at any prompt to cancel.

### Typical workflow (both platforms)

1. Join a race server → demo starts recording automatically
2. Finish a run you like
3. Run `wf-demos` → pick **save** → select the run → it's archived with an auto-generated name
4. Later: `wf-demos list` to browse, Enter (Linux) or `1` (Windows) to watch a run in Warfork

---

## How the auto-recording works

The setup/installer drops an `autoexec.cfg` into your Warfork mod directory. It sets up a 10-alias chain and overrides your join/practice-mode binds:

**Join key** (`4` by default):
```
stop; join; record run_XX; (advance slot counter)
```

**Practice mode key** (`1`, `F5` by default):
```
stop; practicemode
```

The slot counter (`run_00` … `run_09`) advances each join and resets to `run_00` when Warfork starts. Oldest demos are silently overwritten.

---

## Configuration

Edit the config file directly to change paths without re-running setup.

**Linux** — `~/.config/wf-demos/config`:
```bash
DEMO_DIR="${HOME}/.local/share/warfork-2.1/racemod_2.1/demos"
ARCHIVE_DIR="${HOME}/demos"
APPID=671610
WF_MOD=racemod_2.1
```

**Windows** — `%APPDATA%\wf-demos\config`:
```
DEMO_DIR=C:\Users\you\Documents\My Games\Warfork 2.1\racemod_2.1\demos
ARCHIVE_DIR=C:\Users\you\Documents\wf-demos
APPID=671610
WF_MOD=racemod_2.1
```

---

## File layout

### Linux

| Path | Purpose |
|------|---------|
| `~/.local/bin/wf-demos` | Main CLI |
| `~/.local/bin/wf-demo-info` | Demo header parser (Python) |
| `~/.config/wf-demos/config` | User config |
| `~/.local/share/warfork-2.1/racemod_2.1/autoexec.cfg` | Auto-recording config (default path) |
| `~/demos/favorites/` | Saved demos |
| `~/demos/trash/` | Trashed demos (pending `clear-temp`) |

### Windows

| Path | Purpose |
|------|---------|
| `wf-demos.py` | Main CLI (run from repo folder, or add to PATH) |
| `wf-demos.bat` | Launcher — calls `python wf-demos.py %*` |
| `%APPDATA%\wf-demos\config` | User config |
| `Documents\My Games\Warfork 2.1\racemod_2.1\autoexec.cfg` | Auto-recording config (default path) |
| `Documents\wf-demos\favorites\` | Saved demos |
| `Documents\wf-demos\trash\` | Trashed demos (pending `clear-temp`) |

---

## Uninstall

**Linux:**
```bash
rm ~/.local/bin/wf-demos ~/.local/bin/wf-demo-info
rm -rf ~/.config/wf-demos
# optionally keep your saved demos:
# rm -rf ~/demos
```

**Windows:** Delete the repo folder and `%APPDATA%\wf-demos`. Optionally delete `Documents\wf-demos`.

On both platforms: remove the wf-demos block from your `autoexec.cfg` to restore your original binds.

---

## Notes

- Demo playback launches Warfork via `steam -applaunch` (Linux) or via the registry-detected `steam.exe` (Windows). Steam must be running.
- The qfusion engine auto-prepends `demos/` to demo names — do not include it yourself.
- Warfork demo extensions look like `.wfdz22`; the version number may vary. The glob `*.wfdz*` handles all versions.
- The demo header parser reads the first 512 bytes of each file to extract map name, duration, and timestamp — no external libraries required.
- On Windows, if Steam is not found automatically, you'll be prompted to copy the demo manually and launch it from within the game.
