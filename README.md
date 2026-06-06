# Antigravity CLI Statusline Skill

[![Version](https://img.shields.io/badge/version-1.2.2-blue.svg)](./SKILL.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey.svg)]()

[繁體中文](README.zh-TW.md) | English

This repository provides a Skill for setting up and customizing the Antigravity CLI Statusline (Footer), including display items and localization settings. It is designed to work across multiple platforms and handles configuration efficiently.

## Screenshot

### Windows

| English (us) | Traditional Chinese (zh-tw) | Japanese (jp) |
| :---: | :---: | :---: |
| ![English](docs/images/antigravity-cli-statusline-windows-us.png) | ![Traditional Chinese](docs/images/antigravity-cli-statusline-windows-zhtw.png) | ![Japanese](docs/images/antigravity-cli-statusline-windows-jp.png) |

### macOS

| English (us) | Traditional Chinese (zh-tw) | Japanese (jp) |
| :---: | :---: | :---: |
| ![English](docs/images/agy-cli-statusline-macos-us.png) | ![Traditional Chinese](docs/images/agy-cli-statusline-macos-zhtw.png) | ![Japanese](docs/images/agy-cli-statusline-macos-jp.png) |

## Features

- **Rich Status Indicators**: Choose from a variety of display items, including:
  - Current AI Model Name
  - Account API Available Quota
  - API Reset Countdown
  - Context Window Usage Percentage
  - Session Token Count
  - CLI RAM Usage
  - Current Git Branch
  - Project Path (Short / Full)
  - Account Plan Tier
  - Account Email
  - AI Credits
- **Customizable Display Order & Filtering**: Interactively select which indicators to display and customize their exact order through an intuitive, multi-stage setup wizard.
- **Hot-Reload Support**: Changes to the statusline configuration are applied immediately, without requiring a CLI restart.
- **Multi-language Support**: Natively supports Traditional Chinese, English, and Japanese. Features a dynamic architecture that allows you to easily add new languages using AI.
- **Node.js Environment Pre-Check**: Before writing any configuration, the skill detects whether Node.js is installed. If missing, it emits a clear warning in your selected language (zh-tw / us / jp) — preventing the silent failure pattern where `agy` logs `statusline: command failed: exit status 127` 30 times and auto-disables the statusline — and lets you decide whether to abort and install Node.js first or continue setup and have the configuration take effect later.
- **Three-Layer Settings Synchronization**: Dynamically parses and writes `~/.gemini/settings.json` (global), `~/.gemini/antigravity-cli/settings.json` (CLI-specific, **highest priority**), and `<workspace>/.gemini/settings.json` (project) in lock-step. Missing the CLI-specific layer silently overrides your global settings — the skill handles this for you.
- **Python-Free Cross-Platform Architecture**: Completely removes Python dependencies. Uses native commands (`ps`, `lsof`) for macOS/Linux. For Windows 10 / 11, the skill:
  - Uses `Get-CimInstance Win32_Process` to replace the deprecated `wmic` (removed in Windows 11).
  - Automatically compiles a window-less `sh.exe` bridge (`/target:winexe`) via the built-in `csc.exe` and deploys it to the `agy` CLI bin directory, eliminating the black-box flicker caused by missing `sh.exe`.
  - Performs UTF-8 BOM pre-checks on every settings file and verifies the first three bytes after every write to avoid Go-side JSON parsing crashes (`invalid character 'ï' looking for beginning of value`).
  - Ensures accurate memory tracking for the `agy.exe` process even when Git is not in PATH.
- **Smart Line Wrapping**: Automatically wraps the statusline to a new line if it exceeds the terminal width.
- **Dynamic Visual Color Feedback**: Provides intuitive ANSI color coding — a 24-bit truecolor four-tier palette (Blue `#57caff` → Green `#5cdb6d` → Yellow `#ffd427` → Pink `#ff7daf`) based on API quota or context usage, and unique brand colors automatically applied to different AI model families.

## Prerequisites

- **Node.js**: This skill is implemented in pure Node.js (`.mjs`). Your system must have Node.js installed, and the `node` command must be available in your terminal. If Node.js is missing, the skill will warn you in your selected language and let you choose whether to abort and install it first, or continue (the configuration will be written and take effect once Node.js is installed later).
- **Git** *(Optional)*: The statusline reads the Git branch of the current project. If you want this feature to work, it's recommended to have Git installed on your system.

## Getting Started

1. Go to the **[Releases page](../../releases/latest)** of this repository and download the latest release archive (`.zip` or `.tar.gz`).
2. Extract the archive and place the `antigravity-cli-statusline` folder into your `~/.gemini/skills/` directory.
3. When interacting with Antigravity CLI, simply run `/antigravity-cli-statusline` to trigger the skill.

## Contributing

We welcome contributions! For details on how to submit Pull Requests, report bugs, or add new language translations via AI, please see our **[Contributing Guide (CONTRIBUTING.md)](CONTRIBUTING.md)**.

## Troubleshooting & Advanced References

For deeper technical details, see [`SKILL.md`](SKILL.md) and the three reference documents under `references/`:

- [`references/windows.md`](references/windows.md) — Windows-specific rules (UTF-8 BOM, `sh.exe` bridge, `csc.exe` compilation, `windowsHide`, `Get-CimInstance`)
- [`references/config-files.md`](references/config-files.md) — Three-layer settings file structure, `statusLine` object, `trusted_hooks.json` trust mechanism
- [`references/pitfalls.md`](references/pitfalls.md) — Common pitfall lookup table

### Diagnosing a Suddenly-Disappeared Statusline

If your statusline suddenly vanishes (especially after running `agy` commands like `/statusline` or `/model`), run the read-only diagnostic script from the skill directory and paste its full output to your AI agent for repair guidance:

```bash
node scripts/diagnose-statusline.mjs
```

The script inspects all three settings files, `trusted_hooks.json`, Hook file existence, and — most importantly — whether the highest-priority CLI-specific layer's `statusLine.command` has been silently emptied.

## Acknowledgements

Special thanks to [60ke/antigravity-statusline](https://github.com/60ke/antigravity-statusline) for the inspiration behind the quota monitoring feature. Their original project paved the way for statusline integrations. Since their implementation was written in Python, which could be challenging to execute consistently across Windows and macOS, this project was rewritten in JavaScript (Node.js) to ensure seamless cross-platform compatibility without relying on external Python dependencies.

## License

This project is licensed under the [MIT License](LICENSE).
