# 🧹 cleanzone
> Lightweight Bash tool to remove `:Zone.Identifier` files in WSL2

![Shell](https://img.shields.io/badge/shell-Bash-green)
![Platform](https://img.shields.io/badge/platform-WSL2%20%7C%20Linux-orange)
![Release](https://img.shields.io/badge/release-v1.1.0-brightgreen)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## 📖 Introduction

`cleanzone` is a lightweight Bash utility for developers working with **WSL2** on Windows.  

When you edit or move files using Windows Explorer or certain IDEs, Windows often creates hidden  
`*:Zone.Identifier` files (Alternate Data Streams). These files are harmless but clutter your project directories,  
can confuse build tools, and sometimes even cause issues with Git.

`cleanzone` keeps your workspace **clean and consistent** by listing or removing these files – with **safe defaults**,  
a **confirmation threshold**, **logging**, flexible **ignore rules**, and **warning capture**.

---

## 📥 Installation

### System-wide (/usr/local/bin)

**Option A: Copy the file from the repo**
```bash
sudo cp cleanzone /usr/local/bin/
sudo chmod +x /usr/local/bin/cleanzone
```
✅ Done! You can now run `cleanzone` from anywhere inside WSL2/Linux.

**Option B: Install via cURL (directly from GitHub)**
```bash
sudo curl -fsSL https://raw.githubusercontent.com/Rufnex/cleanzone/main/cleanzone -o /usr/local/bin/cleanzone
sudo chmod +x /usr/local/bin/cleanzone
```

### User-local (~/.local/bin, no sudo)
```bash
mkdir -p ~/.local/bin
curl -fsSL https://raw.githubusercontent.com/Rufnex/cleanzone/main/cleanzone -o ~/.local/bin/cleanzone
chmod +x ~/.local/bin/cleanzone
# If ~/.local/bin is not in PATH yet:
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Test & Uninstall
```bash
cleanzone -V
cleanzone -h
which cleanzone
```
Uninstall:
```bash
sudo rm -f /usr/local/bin/cleanzone
# or (user-local)
rm -f ~/.local/bin/cleanzone
```

---

## ✨ Features
- 🗑️ Remove or list `:Zone.Identifier` files
- 🔁 Recursive mode (with default excludes like `.git`, `node_modules`, `.venv`)
- 📂 Project (`./.cleanzoneignore`) and 🌍 global (`~/.cleanzoneignore`) ignore files
- ⚙️ Ad‑hoc excludes via `-x "dist build .cache"` (additive, repeatable)
- ➕ **Alternative**: Excludes **after `--`** without `-x`, e.g. `-- dist build .cache`
- 🔤 Short option **clustering**: `-rl`, `-rfy`, `-rfyx` etc.
- 🛡️ **Safe by default**: dry‑run unless `-f` is used
- ❓ **Confirmation threshold**: default **100**, configurable via `-tN` or `CLEANZONE_THRESHOLD=N`
- 📝 **Logging**: deletions are written to **`./cleanzone.log`** (current directory; falls back to `~/.cleanzone.log` if needed)
- ⚠️ **Warnings capture**: `find` warnings are summarized at the end and logged; use **`-v`** to print them

---

## ✅ Recommended flow (clustered flags)

1. `cleanzone -rl` – **recursive dry‑run** (list only)  
2. `cleanzone -rf` – delete recursively, **ask** when matches ≥ threshold (default 100)  
3. `cleanzone -rfy` – delete recursively, **no prompt**  
4. `cleanzone -rf -t200` *(or `-t 200`)* – prompt only from **200** matches

> Tip: `-t` controls the confirmation threshold; `-y` skips the prompt entirely.

---

## 📌 Usage Examples

```bash
# List in current folder (explicit dry‑run)
cleanzone -l

# Recursively list (dry‑run)
cleanzone -rl

# Really delete (with prompt if many files)
cleanzone -rf

# Really delete without prompt
cleanzone -rfy

# Custom excludes (two ways, additive)
cleanzone -rf -x "dist build .cache"
cleanzone -rf -- dist build .cache

# Change confirmation threshold (here: 200)
cleanzone -rf -t200
```

### Options and Combinations

| Option / Combo   | Meaning                                             | Example                              |
|------------------|-----------------------------------------------------|--------------------------------------|
| `-r`             | Recursive: search subdirectories                    | `cleanzone -r`                       |
| `-l`             | List-only: preview, don’t delete                    | `cleanzone -l`                       |
| `-f`             | Force delete: perform deletion                      | `cleanzone -f`                       |
| `-y`             | No prompt when deleting many files                  | `cleanzone -fy`                      |
| `-tN`            | Confirmation threshold (default 100; `-t 200` ok)   | `cleanzone -rf -t200`                |
| `-x "LIST"`      | Extra excludes (space‑separated, repeatable)        | `cleanzone -rf -x "dist build"`      |
| `-- …`           | Alternative to `-x`: tokens after `--` are excludes | `cleanzone -rf -- dist build .cache` |
| `-rl`            | Recursive + list-only (safe dry-run)                | `cleanzone -rl`                      |
| `-rf`            | Recursive + delete (with prompt if many files)      | `cleanzone -rf`                      |
| `-rfy`           | Recursive + delete, no prompt                       | `cleanzone -rfy`                     |
| `-rfyx …`        | Combine freely, e.g., with extra excludes           | `cleanzone -rfyx "dist build .cache"`|
| `-v`             | Show suppressed find warnings at the end (verbose)  | `cleanzone -rl -v`                   |
| `-h`             | Show help and exit                                  | `cleanzone -h`                       |
| `-V`, `--version`| Show version and exit                               | `cleanzone -V`                       |

---

## 🧪 Threshold via environment variable (`CLEANZONE_THRESHOLD`)

- **One‑off (single command)**:  
  ```bash
  CLEANZONE_THRESHOLD=200 cleanzone -rf
  ```
- **Whole shell session**:  
  ```bash
  export CLEANZONE_THRESHOLD=200
  cleanzone -rf
  ```
- **Precedence**: `-tN` overrides the environment variable:  
  ```bash
  export CLEANZONE_THRESHOLD=200
  cleanzone -rf -t500   # uses 500
  ```

Default is **100** if neither env var nor `-t` is set.

---

## 📂 Ignore Files

Configure directories that should **not** be searched.

### Project‑specific
Create a `.cleanzoneignore` file in your project root. Example:

```
dist
build
.cache
```

→ Applies only inside this project.

### Global
Create `~/.cleanzoneignore` for all projects. Example:

```
node_modules
.venv
__pycache__
```

→ Applies to every run unless overridden by a project file.

### Note on DB volumes (Docker, databases)
Some folders (e.g., database volumes) may be root-owned and emit warnings.  
Prefer project/global `.cleanzoneignore` to exclude them, e.g.:
```
.docker/database/data
mariadb_data
postgres_data
```

> Tip: Since logs are created in the current directory (`./cleanzone.log`), consider adding `cleanzone.log` to your `.gitignore`.

---

## ⚙️ Internals & Safety (Deep Dive)

**Targeted matching**  
- Only **regular files** ending with `*:Zone.Identifier` are selected:  
  `-type f -name '*:Zone.Identifier'`

**Robust path handling**  
- Gather with `find … -print0`, read via `mapfile -d ''` (safe for spaces/newlines).  
- Delete in a single batch: `rm -f -- "${array[@]}"`.

**Safe defaults**  
- Default = dry‑run; real deletion only with `-f`.  
- Confirmation when matches ≥ N (default 100).  
  - Per‑run: `-tN` (e.g. `-t200`)  
  - Env var: `CLEANZONE_THRESHOLD=N` (overridden by `-tN`)  
  - `-y` suppresses the prompt.

**Logging & warnings**  
- Writes deletions to **`./cleanzone.log`** (fallback `~/.cleanzone.log`).  
- `find` warnings (e.g., *Permission denied*) are captured; short summary printed at the end; full details appended to `cleanzone.log`.  
- Use **`-v`** to print warnings to the terminal as well.

**Excludes & performance**  
- Defaults: `.git`, `node_modules`, `.venv`.  
- Ignore files: project first, then global.  
- Large dirs are **pruned** with `-prune` for speed.

**Limits**  
- Outside WSL/DrvFS these ADS files usually don’t exist.  
- Files intentionally named with this suffix will match.  
- No `sudo` inside the script.

---

## 🧪 Mini-CI (ShellCheck + Smoke-Test)

Add this workflow at **`.github/workflows/ci.yml`** to auto‑check your script on each push/PR (and manually via *Run workflow*).

```yaml
name: ci
on: [push, pull_request, workflow_dispatch]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  lint-and-smoke:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install ShellCheck
        if: ${{ hashFiles('cleanzone') != '' }}
        run: sudo apt-get update && sudo apt-get install -y shellcheck

      - name: ShellCheck (non-blocking)
        if: ${{ hashFiles('cleanzone') != '' }}
        run: shellcheck -S warning -x ./cleanzone || true

      - name: Smoke test
        if: ${{ hashFiles('cleanzone') != '' }}
        shell: bash
        run: |
          set -e
          chmod +x ./cleanzone

          mkdir -p demo/dist demo/keep
          touch "demo/a.txt:Zone.Identifier"
          touch "demo/dist/b.txt:Zone.Identifier"

          echo "== Dry-run (recursive) with -- excludes =="
          out=$(cd demo && ../cleanzone -rl -- dist)
          echo "$out" | grep -F "a.txt:Zone.Identifier" >/dev/null
          if echo "$out" | grep -F -q "dist/b.txt:Zone.Identifier"; then
            echo "Should have excluded dist/"; exit 1; fi

          echo "== Delete without prompt (rfy) =="
          (cd demo && ../cleanzone -rfy -- dist)

          echo "== Logging in current directory =="
          test -f "demo/cleanzone.log"

      - name: Note if script is missing
        if: ${{ hashFiles('cleanzone') == '' }}
        run: echo "No 'cleanzone' file found in repo root; skipping checks."
```

Add the **CI badge** if desired:
```
![CI](https://github.com/USER/REPO/actions/workflows/ci.yml/badge.svg)
```

---

## ⚠️ Disclaimer
This tool deletes files. Review paths with a dry‑run before removing them.  
Provided **as‑is**, **without warranty** or liability of any kind.  
Use at your own risk and only if you understand the consequences.  
For full terms, see the **MIT License**.

---

## 📜 License
MIT License – do whatever you want, but no warranty.
