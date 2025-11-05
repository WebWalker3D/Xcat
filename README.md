Absolutely — here’s an **updated `README.md`** reflecting **xcat 1.0.0-dev** with all the new flags, behavior, and features.
It’s rewritten for clarity and polished for GitHub (proper tables, emoji section headers, and examples).

---

````markdown
# xcat

`xcat` is a feature-packed, dependency-light Bash utility that **copies files, stdin, or directory contents to your clipboard** — with optional line ranges, preview, compression, verification, and backend autodetection.  

It’s made for those “show me the exact lines you’re talking about” moments: logs, snippets, configs, or code blocks — with safety, style, and speed.  

---

## ✨ Features

- 📋 **Copy to clipboard** from:
  - A file  
  - Standard input (pipes)  
  - All regular files in a directory (non-recursive)
- 🔢 **Optional line ranges**:
  - `-s START` / `-e END` (1-based, inclusive)  
  - `-L START:END` shorthand (either side can be omitted)
- 🖥️ **Auto-detects clipboard backend**:
  - `xclip` (X11)
  - `wl-copy` / `wl-clipboard` (Wayland)
  - `pbcopy` (macOS)
  - `clip.exe` (WSL)
- 📦 **Auto-install** clipboard tools where possible using:
  - `apt-get`, `dnf`, `yum`, `pacman`, `zypper`, or `brew`
- 🧱 **Directory mode**:
  - Non-recursive  
  - Concatenates all regular text files  
  - Supports `--types=ext1,ext2` filters  
  - Prefixes each chunk with its full path
- 🧠 **Advanced options**:
  - `--preview` — syntax-highlight preview (with `bat` or `batcat`)  
  - `--verify` — show clipboard contents after copy  
  - `--compress` — gzip + base64 encode payloads before copying  
  - `--tee` — print and copy simultaneously  
  - `--backend=NAME` — override autodetection  
  - `--list-backends` — show available backends  
  - `--quiet` / `--verbose` — control output  
  - `--version` — print version info  
- 🛡️ **Safety features**:
  - Detects and refuses binary input  
  - Validates numeric options  
  - Uses `set -euo pipefail`  
  - Clear, colored messages for feedback
- 🧮 **Integrity & info**:
  - Shows SHA-1 checksum of copied payload  
  - Reports compression and line range info

---

## ⚙️ Requirements

- **Bash ≥ 4.0**
- One of the following clipboard utilities:
  - **Linux / BSD:** `xclip` (X11) or `wl-clipboard` (Wayland)
  - **macOS:** `pbcopy`
  - **WSL:** `clip.exe`

If none are available, `xcat` will attempt to install `xclip` or `wl-clipboard` automatically using your system package manager.  
If installation fails, it will print manual instructions.

---

## 🧩 Installation

```bash
sudo curl -L https://raw.githubusercontent.com/WebWalker3D/Xcat/main/xcat \
  -o /usr/local/bin/xcat
sudo chmod +x /usr/local/bin/xcat
````

Verify it’s working:

```bash
xcat -h
```

You should see the usage help.

---

## 💡 Usage

### **Synopsis**

```bash
xcat [OPTIONS] [FILE]
```

* If `FILE` is omitted, `xcat` reads from **stdin**.
* If `FILE` is a **directory**, it:

  * Concatenates all regular text files (non-recursively)
  * Optionally filters by extension via `--types=`
  * Prefixes each section with its full path
  * Sends the combined output to your clipboard

---

### **Options**

| Option              | Description                                                  |
| :------------------ | :----------------------------------------------------------- |
| `-s START`          | 1-based start line (inclusive)                               |
| `-e END`            | 1-based end line (inclusive)                                 |
| `-L A:B`            | Shorthand line range (`-L 10:20`, `-L 15:`, `-L :50`)        |
| `-p`, `--preview`   | Show syntax-highlighted preview (requires `bat` or `batcat`) |
| `--verify`          | Print first 10 lines of clipboard after copying              |
| `--compress`        | gzip + base64 encode payload before copying                  |
| `--tee`             | Print output to stdout and clipboard simultaneously          |
| `--backend=NAME`    | Force backend (`xclip`, `wl-copy`, `pbcopy`, `clip.exe`)     |
| `--types=ext1,ext2` | Limit directory mode to specific file types                  |
| `--list-backends`   | Show available clipboard backends and exit                   |
| `-q`, `--quiet`     | Suppress non-error output                                    |
| `-v`, `--verbose`   | Show backend and diagnostic info                             |
| `-V`, `--version`   | Print version info and exit                                  |
| `-h`, `--help`      | Show usage help and exit                                     |

Numeric options are validated; `START` and `END` must be positive integers, and `END` cannot be less than `START`.

---

## 🧪 Examples

### Copy a whole file

```bash
xcat ~/.bashrc
```

### Copy specific line ranges

```bash
xcat -s 10 -e 30 app.log
xcat -L 15: app.log
xcat -L :50 app.log
```

### Copy from stdin (e.g. pipeline)

```bash
grep ERROR app.log | xcat -L :50
kubectl get pods -A | xcat
```

### Copy all text files in a directory

```bash
xcat /etc/apache2/sites-available/
```

Filter by extension:

```bash
xcat /etc --types=conf,service
```

### Use preview and verify modes

```bash
xcat -p --verify app.log
```

### Compress output before copying

```bash
xcat --compress large.txt
```

### Override clipboard backend

```bash
xcat --backend=wl-copy myfile.txt
```

### Tee output to both clipboard and stdout

```bash
xcat --tee script.sh | wc -l
```

### List available backends

```bash
xcat --list-backends
```

---

## 🧠 Clipboard Detection & Auto-Install

On first run, `xcat` tries the following in order:

1. `xclip -selection clipboard`
2. `wl-copy`
3. `pbcopy`
4. `clip.exe`

If none are available:

* Detects **Wayland** vs **X11**
* Attempts to install the matching tool via your package manager:

  * `wl-clipboard` (for Wayland)
  * `xclip` (for X11)
* Supported managers: `apt-get`, `dnf`, `yum`, `pacman`, `zypper`, `brew`

If all fail, `xcat` prints clear manual install instructions.

---

## 🚪 Exit Codes

| Code | Meaning                                                     |
| :--: | :---------------------------------------------------------- |
|  `0` | Success                                                     |
|  `1` | Runtime failure (e.g. file not found, no clipboard backend) |
|  `2` | Invalid usage (bad options, non-numeric values, etc.)       |

---

## 🧩 Version & Integrity Info

Each successful copy displays:

* SHA-1 checksum of the payload
* Optional compression and range info
* Active clipboard backend (in verbose mode)

---

## 💬 Notes & Tips

* Works beautifully with `grep`, `rg`, `sed`, `awk`, etc.
* Great over SSH with X11 forwarding or WSL clipboard bridges. (Original use case was copying text files over SSH)
* Safe with binary detection — won’t copy executables accidentally.
* Directory mode + `--types` is handy for aggregating config files.
* Combine with `fzf` or `bat` for interactive snippet sharing.
* Designed to be portable, predictable, and copy-ready anywhere.

---

## 🧾 License

**MIT License**
© 2025 Blaine Bouvier (@WebWalker3D)
See [`LICENSE`](LICENSE) for full text.
