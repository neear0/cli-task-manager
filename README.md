# cli-task-manager
Modern command-line todo / task manager with AI integration
<div align="center">

```
╭──────────────────────────────────────────────────────╮
│  ✦ TODO — C++23 Task Manager                         │
│  ◻ pending  ✓ done  ⚠ overdue  ↻ recurring  ★ crit  │
╰──────────────────────────────────────────────────────╯
```

# ACTM · Annoying C++23 Task Manager

**A zero-dependency, full-screen terminal task manager built in C++23.**  
Interactive TUI that looks like htop. One-shot CLI for scripts. Runs at startup.

[![C++23](https://img.shields.io/badge/C%2B%2B-23-blue?logo=cplusplus&logoColor=white)](https://en.cppreference.com/w/cpp/23)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20Linux%20%7C%20macOS-lightgrey)](#build)
[![Dependencies](https://img.shields.io/badge/dependencies-zero-brightgreen)](#)

[**Features**](#-features) · [**Build**](#-build) · [**Usage**](#-usage) · [**TUI Controls**](#-interactive-tui) · [**File Structure**](#-project-structure)

</div>

---

##  Features

| | |
|---|---|
| **Full-screen interactive TUI** | htop/ranger-style interface — arrow keys, highlight, scroll, inline forms |
| **One-shot CLI** | Every operation available as `todo <command>` for scripts and aliases |
| **OS startup registration** | One command registers the app to open at login on Windows, Linux and macOS |
| **Rich task model** | 5 priority levels, due dates, recurrence rules, tags and free-text notes |
| **Persistent undo** | 20 levels of undo that survive across process restarts |
| **Recurring tasks** | Auto-spawns the next occurrence when you complete a recurring task |
| **Export** | GitHub-flavored Markdown and iCalendar RFC 5545 (`.ics`) |
| **Zero dependencies** | Custom JSON engine built from scratch — no Boost, no vcpkg, nothing |
| **Cross-platform** | Windows (MSVC / MinGW), Linux (GCC / Clang), macOS (Apple Clang) |

---

## Build

### Requirements

| Platform | Compiler | Minimum |
|----------|----------|---------|
| Windows | MSVC | Visual Studio 2022 17.4+ |
| Windows | MinGW-w64 | GCC 13+ |
| Linux | GCC | 13+ |
| Linux | Clang | 17+ |
| macOS | Apple Clang | 15+ (Xcode 15) |

> **C++23 features used:** `std::format`, `std::ranges`, `std::chrono::year_month_day`, designated initialisers, `std::variant`, deduced `this`.

---

### Linux / macOS

```bash
git clone https://github.com/yourname/todo
cd todo
make           # debug build  →  ./todo
make release   # optimised    →  ./todo
make install   # copies to ~/bin/todo
```

### Windows — Visual Studio

Add all `.cpp` files to your project and set these properties:

| Property | Value |
|---|---|
| C++ Language Standard | `ISO C++23 (/std:c++latest)` |
| Additional Options | `/utf-8` |
| Subsystem | Console |

Or from the **Developer Command Prompt:**

```cmd
cl /std:c++latest /utf-8 /EHsc /W3 ^
   main.cpp json.cpp date.cpp task.cpp storage.cpp ^
   tui.cpp exporter.cpp interactive.cpp startup.cpp ^
   /Fe:todo.exe
```

### Windows — MinGW / MSYS2

```bash
g++ -std=c++23 -O2 -o todo.exe \
    main.cpp json.cpp date.cpp task.cpp storage.cpp \
    tui.cpp exporter.cpp interactive.cpp startup.cpp
```

> **💡 Tip:** Run from **Windows Terminal** for full ANSI color and UTF-8 box-drawing support. The classic `cmd.exe` debug console inside Visual Studio will render characters incorrectly.

---

## 🚀 Quick Start

```bash
# Launch the interactive TUI — just run it with no arguments
todo

# Or go straight to CLI mode
todo add "Fix login bug" --priority urgent --due 2026-03-01 --tags work,backend
todo list
todo done 1
```

---

## 📖 Usage

```
todo [--no-color] [--data <path>] <command> [options]
```

### Global Flags

| Flag | Description |
|---|---|
| `--no-color` | Disable all ANSI color output |
| `--data <path>` | Use a custom JSON data file instead of the default |
| `--interactive` | Force interactive TUI mode (used by the startup entry) |

---

### Commands

<details>
<summary><b>add</b> — Create a new task</summary>

<br>

```bash
todo add "Task title" [options]
todo add    # guided interactive prompt if no title given
```

| Flag | Description | Values |
|---|---|---|
| `--priority <p>` | Task priority | `low` `normal` `high` `urgent` `critical` |
| `--due <date>` | Due date | `YYYY-MM-DD` |
| `--recur <r>` | Recurrence rule | `daily` `weekly` `biweekly` `monthly` |
| `--tags <tags>` | Comma-separated tags | e.g. `work,backend` |
| `--notes <text>` | Free-text note | |

```bash
todo add "Deploy hotfix" --priority critical --due 2026-03-01 --tags ops
todo add "Morning standup" --recur daily --priority normal --tags meetings
todo add "Pay rent" --priority urgent --due 2026-03-05 --recur monthly
```

</details>

<details>
<summary><b>list</b> — Show tasks</summary>

<br>

```bash
todo list [filters]
```

| Flag | Description |
|---|---|
| `--all` | Include done and cancelled tasks |
| `--done` | Show only completed tasks |
| `--overdue` | Show only overdue tasks |
| `--tag <tag>` | Filter by tag |
| `--priority <p>` | Filter by priority level |
| `--verbose` / `-v` | Show notes beneath each row |

```bash
todo list
todo list --overdue
todo list --tag work --priority high
todo list --all --verbose
```

</details>

<details>
<summary><b>done &nbsp;·&nbsp; ✗ cancel &nbsp;·&nbsp; 🗑 delete</b></summary>

<br>

```bash
todo done   <id> [id...]    # mark complete; auto-spawns next if recurring
todo cancel <id>            # mark as cancelled
todo delete <id> [id...]    # permanently remove
```

</details>

<details>
<summary><b>edit</b> — Modify a task</summary>

<br>

```bash
todo edit <id> [--flag value...]
```

Accepts the same flags as `add`. Use `none` to clear a field:

```bash
todo edit 3 --priority urgent --due 2026-03-15
todo edit 5 --tags "work,urgent" --notes "Blocking the release"
todo edit 2 --due none --recur none
```

</details>

<details>
<summary><b>show</b> — Full task detail panel</summary>

<br>

```bash
todo show <id>
```

```
╭─ Task #2 ─────────────────────────────────────────────╮
│                                                         │
│  Title       Write unit tests                           │
│  Status      ◻ pending                                  │
│  Priority    ▲ high                                     │
│  Due         2026-03-01 (in 9 days)                     │
│  Tags        #dev #quality                              │
│  Created     2026-02-21                                 │
├─────────────────────────────────────────────────────────┤
│  ✎  Cover all edge cases in the parser                  │
╰─────────────────────────────────────────────────────────╯
```

</details>

<details>
<summary><b>undo</b> — Undo the last action</summary>

<br>

```bash
todo undo
```

Up to **20 levels**, persisted across sessions — so undo works even after closing and reopening the app.

</details>

<details>
<summary><b>stats</b> — Statistics dashboard</summary>

<br>

```bash
todo stats
```

```
╭─ Statistics ───────────────────────────────────────────╮
│  Pending    5     ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░░░░  62%  │
│  Done       2     ▓▓▓▓▓▓▓░░░░░░░░░░░░░░░░░░░░░░░  25%  │
│  Cancelled  1     ▓▓▓░░░░░░░░░░░░░░░░░░░░░░░░░░░  12%  │
│  ⚠ Overdue  1     ▓▓▓░░░░░░░░░░░░░░░░░░░░░░░░░░░  12%  │
├─ By Priority ──────────────────────────────────────────┤
│  ★ critical  1    ▓▓▓░░░░░░░░░░░░░░░░░░░░░░░░░░░  12%  │
│  ◆ urgent    2    ▓▓▓▓▓▓▓░░░░░░░░░░░░░░░░░░░░░░░  25%  │
│  ▲ high      2    ▓▓▓▓▓▓▓░░░░░░░░░░░░░░░░░░░░░░░  25%  │
╰────────────────────────────────────────────────────────╯
```

</details>

<details>
<summary><b>export</b> — Export to Markdown or iCalendar</summary>

<br>

```bash
todo export --md   tasks.md     # GitHub-flavored Markdown with checkboxes
todo export --ical tasks.ics    # iCalendar RFC 5545 — imports into Outlook, Apple Calendar, etc.
```

</details>

<details>
<summary><b> startup</b> — Register to run at OS login</summary>

<br>

```bash
todo startup              # check if registered
todo startup install      # register
todo startup remove       # unregister
```

| Platform | Location |
|---|---|
| Windows | `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` |
| Linux | `~/.config/autostart/todo-taskmanager.desktop` |
| macOS | `~/Library/LaunchAgents/com.todo.taskmanager.plist` |

The startup entry always passes `--interactive` so the TUI opens on login.

</details>

---

## Interactive TUI

Launch with no arguments (or `todo --interactive`) for the full-screen experience.

### Navigation

| Key | Action |
|---|---|
| `↑` `↓` &nbsp;or&nbsp; `k` `j` | Move selection up / down |
| `PgUp` &nbsp;`PgDn` | Scroll a full page |
| `Home` &nbsp;or&nbsp; `g` | Jump to top |
| `End` &nbsp;or&nbsp; `G` | Jump to bottom |

### Actions

| Key | Action |
|---|---|
| `Enter` or `d` | ✓ Mark selected task done |
| `a` | ➕ Add new task (inline form at the bottom) |
| `e` | ✎ Edit selected task (inline form) |
| `x` or `Delete` | 🗑 Delete with `y/N` confirmation |
| `c` | ✗ Cancel selected task |
| `u` | ↩ Undo last action |

### Views & Overlays

| Key | Action |
|---|---|
| `Tab` | Cycle view → **Pending** → **All** → **Done** |
| `s` | Stats overlay |
| `?` | Help overlay |
| `r` | Reload from disk |
| `q` or `Esc` | Quit |
| `Ctrl-C` | Force quit |

---

## Priority Reference

| Icon | Priority | When to use |
|---|---|---|
| `★` | **Critical** | On fire. Blocking everything. Must do today. |
| `◆` | **Urgent** | Hard deadline, high impact. |
| `▲` | **High** | Important but not immediately blocking. |
| `●` | **Normal** | Default. Everyday work. |
| `▽` | **Low** | Nice-to-have. Do it when you have time. |

> Overdue tasks float to the top of every view and are highlighted in **red**.  
> Tasks due today are highlighted in **yellow**.

---

## Data & Storage

| Platform | Default path |
|---|---|
| Linux / macOS | `~/.todo.json` |
| Windows | `%USERPROFILE%\.todo.json` |

Override with `--data <path>` on any command. The file is human-readable JSON and stores both tasks and the undo history — safe to back up, sync with Dropbox/OneDrive, or commit to git.

---

---

## Contributing

Pull requests are welcome! A few things to keep in mind:

- **Style:** `c_` prefix for classes, `_t` suffix for structs/enums, `k_` prefix for constants — keep it consistent.
- **Standard:** C++23 only. No polyfills, no `#ifdef` workarounds for missing stdlib features.
- **Dependencies:** None. Keep it that way.
- **Platforms:** Test on at least Windows and one POSIX platform before submitting.

---

<div align="center">

</div>
