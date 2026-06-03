# Antigravity CLI Statusline Skill

[![Version](https://img.shields.io/badge/version-1.1.1-blue.svg)](./SKILL.md)
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
- **Multi-language Support**: Natively supports Traditional Chinese, English, and Japanese. Features a dynamic architecture that allows you to easily add new languages using AI.
- **Global & CLI-Specific Settings**: Dynamically parses and updates `~/.gemini/settings.json`, `~/.gemini/antigravity-cli/settings.json`, and project-level settings.
- **Python-Free Cross-Platform Architecture**: Completely removes Python dependencies. Uses native commands (`ps`, `lsof`) for macOS/Linux. For Windows, it implements a bespoke Silent C# Bridge with `windowsHide` to eliminate terminal flickering, while significantly improving compatibility for environments without Git in PATH and ensuring accurate memory tracking for the `agy.exe` process.
- **Smart Line Wrapping**: Automatically wraps the statusline to a new line if it exceeds the terminal width.
- **Dynamic Visual Color Feedback**: Provides intuitive ANSI color coding, such as a 4-tier warning system (Green/Yellow/Orange/Red) based on API quota or context usage, and unique brand colors automatically applied to different AI model families.

## Prerequisites

- **Node.js**: This skill is implemented in pure Node.js (`.mjs`). Your system must have Node.js installed, and the `node` command must be available in your terminal.
- **Git** *(Optional)*: The statusline reads the Git branch of the current project. If you want this feature to work, it's recommended to have Git installed on your system.

## Getting Started

1. Go to the **[Releases page](../../releases/latest)** of this repository and download the latest release archive (`.zip` or `.tar.gz`).
2. Extract the archive and place the `antigravity-cli-statusline` folder into your `~/.gemini/skills/` directory.
3. When interacting with Antigravity CLI, simply run `/antigravity-cli-statusline` to trigger the skill.

## Contributing

We welcome contributions! For details on how to submit Pull Requests, report bugs, or add new language translations via AI, please see our **[Contributing Guide (CONTRIBUTING.md)](CONTRIBUTING.md)**.

## Acknowledgements

Special thanks to [60ke/antigravity-statusline](https://github.com/60ke/antigravity-statusline) for the inspiration behind the quota monitoring feature. Their original project paved the way for statusline integrations. Since their implementation was written in Python, which could be challenging to execute consistently across Windows and macOS, this project was rewritten in JavaScript (Node.js) to ensure seamless cross-platform compatibility without relying on external Python dependencies.

## License

This project is licensed under the [MIT License](LICENSE).
