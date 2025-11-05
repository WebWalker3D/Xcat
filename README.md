# xcat

`xcat` is a tiny shell helper that **cats a file (or stdin) and copies it to your clipboard**, with optional line-range selection and **auto-install** of a clipboard tool on most Linux distros.

It’s meant for those “show me the exact lines you’re talking about” moments — logs, snippets, configs, code blocks, you name it.

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
  - Concatenates all regular files  
  - Prefixes each chunk with its full path
- 🛡️ **Safe defaults**:
  - `set -euo pipefail`
  - Validates numeric options
  - Clear error messages when something’s wrong

---

## ⚙️ Requirements

- **Bash**
- One of the following clipboard utilities (installed or installable):
  - **Linux / BSD:** `xclip` (X11) or `wl-clipboard` (`wl-copy` on Wayland)
  - **macOS:** `pbcopy` (usually available by default)
  - **WSL:** `clip.exe` (part of Windows)

If none of these are available, `xcat` will try to install `xclip` or `wl-clipboard` using your system package manager.  
If that fails, it will tell you what to install manually.

---

## 🧩 Installation

Save the script somewhere in your `$PATH`, for example:

```bash
# Adjust the source URL/path as needed
sudo curl -L https://raw.githubusercontent.com/WebWalker3D/Xcat/main/xcat -o /usr/local/bin/xcat
sudo chmod +x /usr/local/bin/xcat
````

Or just copy the script contents into a file named `xcat` and make it executable:

```bash
cp xcat /usr/local/bin/xcat
chmod +x /usr/local/bin/xcat
```

Verify it’s available:

```bash
xcat -h
```

You should see the usage help.

---

## 💡 Usage

### **Synopsis**

```bash
xcat [-s START] [-e END] [-L START:END] [FILE]
```

* If `FILE` is omitted, `xcat` reads from **stdin**.
* If `FILE` is a **directory**, `xcat` will:

  * Concatenate all regular files in that directory (non-recursively),
  * Prefix each file’s content with its full path,
  * Send the combined output to your clipboard.

---

### **Options**

| Option         | Description                                                 |
| :------------- | :---------------------------------------------------------- |
| `-s START`     | 1-based start line (inclusive)                              |
| `-e END`       | 1-based end line (inclusive)                                |
| `-L START:END` | Shorthand for a line range (`-L 10:20`, `-L 15:`, `-L :50`) |
| `-h`           | Show help and exit                                          |

Numeric options are validated; `START` and `END` must be positive integers, and `END` cannot be less than `START`.

---

## 🧪 Examples

### Copy a whole file to clipboard

```bash
xcat ~/.bashrc
```

### Copy a specific line range from a file

```bash
xcat -s 10 -e 30 app.log
# or
xcat -L 10:30 app.log
```

### Copy from a given line to the end of a file

```bash
xcat -L 15: app.log
```

### Copy the first N lines of a file

```bash
xcat -L :50 app.log
```

### Copy from piped input

```bash
grep ERROR app.log | xcat -L :50
kubectl get pods -A | xcat
```

### Copy all files in a directory

```bash
xcat /etc/apache2/sites-available/
```

Apply a line range to the combined stream:

```bash
xcat -L :200 /etc/apache2/sites-available/
```

> **Note:** Directory mode is non-recursive and only reads regular files in that directory.
> If there are no regular files, you’ll get an error.

---

## 🧠 Clipboard Detection & Auto-Install

On first run, `xcat` tries to find a suitable clipboard command in this order:

1. `xclip -selection clipboard`
2. `wl-copy`
3. `pbcopy`
4. `clip.exe`

If none are available, it:

1. Checks if you’re on **Wayland** or **X11** and attempts to install:

   * `wl-clipboard` (Wayland)
   * `xclip` (X11)
2. Uses your system package manager if found:

   * `apt-get`, `dnf`, `yum`, `pacman`, `zypper`, or `brew`

If it still can’t find a working clipboard tool, it prints a helpful message with manual install commands and exits.

---

## 🚪 Exit Codes

| Code | Meaning                                                      |
| :--: | :----------------------------------------------------------- |
|  `0` | Success                                                      |
|  `1` | Runtime failure (e.g., file not found, no clipboard backend) |
|  `2` | Invalid usage (bad options, non-numeric values, etc.)        |

---

## 💬 Notes & Tips

* Works great with `grep`, `rg`, `sed`, `awk`, etc., to quickly share snippets.
* Plays well over SSH with X11 forwarding or WSL clipboard bridges.  (Original use case was copying text files over SSH)
* Directory mode is handy for collecting multiple small config files in one go.
