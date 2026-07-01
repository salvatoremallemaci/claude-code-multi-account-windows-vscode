# Isolating Two Claude Code Accounts on Windows 11 using VS Code and VS Code Insiders

Run a personal and a work Claude account side by side in separate VS Code instances — no logout required, no credential mixing, fully isolated billing and usage.

---

## ❌ The Problem

Claude Code stores all credentials and configuration in a single global directory:

```
%USERPROFILE%\.claude
```

This means only one account can be active system-wide at any time. Switching accounts requires a full `/logout` → `/login` cycle through the browser OAuth flow every time.

---

## ✅ The Solution

Claude Code respects the `CLAUDE_CONFIG_DIR` environment variable. When set, it reads and writes all credentials, session data, and configuration from that path instead of the default `%USERPROFILE%\.claude`.

By launching a second editor instance with this variable set to a different directory, you get two completely isolated environments:

| Context | Editor | Account | Config Directory |
| :--- | :--- | :--- | :--- |
| 🔵 System default | VS Code (blue) | Work | `%USERPROFILE%\.claude` |
| 🟢 Custom launcher | VS Code Insiders (green) | Personal | `%USERPROFILE%\.claude-personal` |

Each editor maintains its own credentials, session history, and rate limit tracking. There is no shared state between them.

---

## 📋 Requirements

- Windows 11
- [VS Code](https://code.visualstudio.com/) — for your work account
- [VS Code Insiders](https://code.visualstudio.com/insiders/) — for your personal account. VS Code Insiders is the nightly preview build of VS Code, maintained by Microsoft alongside the stable release. It installs as a completely separate application with its own extension registry and user profile, which is exactly what makes it useful here — it shares nothing with standard VS Code by default.
- Claude Code extension installed in both editors
- Two separate Anthropic accounts (any plan combination)

---

## 🛠️ Setup

### Step 1 — Install VS Code Insiders

Download and install [VS Code Insiders](https://code.visualstudio.com/insiders/). It installs independently from standard VS Code and uses a separate extension registry, so there is no conflict between the two.

### Step 2 — Create the launcher script

Create a file named `claude-personal.bat` anywhere convenient (e.g. `C:\Scripts\` or your Documents folder):

```bat
@echo off
set CLAUDE_CONFIG_DIR=%USERPROFILE%\.claude-personal
start "" "%LOCALAPPDATA%\Programs\Microsoft VS Code Insiders\Code - Insiders.exe"
```

This script sets `CLAUDE_CONFIG_DIR` for the child process only, then launches VS Code Insiders. Your system default and any open terminals are not affected.

### Step 3 — Create a desktop shortcut (optional)

For convenience, create a shortcut to the `.bat` file on your desktop:

1. Right-click `claude-personal.bat` → **Send to** → **Desktop (create shortcut)**
2. Rename the shortcut to something recognizable, e.g. `VS Code Personal`
3. Right-click the shortcut → **Properties** → **Change Icon**
4. Browse to `%LOCALAPPDATA%\Programs\Microsoft VS Code Insiders\` and select `Code - Insiders.exe` to use the green Insiders icon

### Step 4 — Authenticate the personal account

1. Launch VS Code Insiders using the `.bat` file or the desktop shortcut
2. Install the Claude Code extension if not already present (`Ctrl+Shift+X` → search "Claude Code")
3. Open the Claude Code panel or a terminal and run:
   ```
   claude
   ```
4. Complete the browser OAuth flow with your personal account credentials

> 💡 Claude Code will write its credentials to `%USERPROFILE%\.claude-personal`, leaving your work account in `%USERPROFILE%\.claude` untouched.

---

## 🚀 Daily Usage

| | Account | How to open |
| :--- | :--- | :--- |
| 🔵 | **Work** | Open standard VS Code normally from taskbar or Start Menu |
| 🟢 | **Personal** | Double-click the `claude-personal.bat` launcher or desktop shortcut |

Integrated terminals inside each editor inherit the correct environment automatically. There is no manual switching, no credential swapping, and no shared state.

---

## ✔️ Verification

To confirm which account is active in any Claude Code session, run:

```
/status
```

This displays the authenticated email address, organization, subscription plan, and rate limit tier for the current session.

---

## 📝 Notes

- 🔒 The `CLAUDE_CONFIG_DIR` variable is set only for the process launched by the script. All other terminals and applications on the system are unaffected.
- 📁 Each config directory is fully independent: credentials, session history, project settings, and MCP server configurations are not shared.
- 💳 This approach works with any two plan combinations (Pro + Pro, Pro + Team, Team + Team, etc.).
- ⚙️ If your VS Code Insiders installation path differs, update the path in the `.bat` file accordingly. The default path is `%LOCALAPPDATA%\Programs\Microsoft VS Code Insiders\Code - Insiders.exe`.

---

## 🔗 References

- [Claude Code Documentation](https://code.claude.com/docs) — Official Claude Code docs, including authentication and configuration
- [Claude Code on GitHub](https://github.com/anthropics/claude-code) — Official repository
- [Visual Studio Code](https://code.visualstudio.com/) — Official VS Code site
- [Visual Studio Code Insiders](https://code.visualstudio.com/insiders/) — Official VS Code Insiders site
- [Anthropic](https://www.anthropic.com/) — Claude developer and maintainer
- [`CLAUDE_CONFIG_DIR` environment variable](https://code.claude.com/docs/en/configuration) — Official documentation on Claude Code configuration options

---

## ™️ Trademarks

Claude™ and Claude Code™ are trademarks of Anthropic, PBC. Visual Studio Code® and Visual Studio Code Insiders® are registered trademarks of Microsoft Corporation. Windows® and Windows 11® are registered trademarks of Microsoft Corporation. All other trademarks and registered trademarks are the property of their respective owners.

This project is not affiliated with, endorsed by, or sponsored by Anthropic or Microsoft.
