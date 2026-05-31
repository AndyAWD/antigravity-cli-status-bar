# Antigravity CLI 狀態列設定技能

[![版本](https://img.shields.io/badge/版本-1.0.0-blue.svg)](./SKILL.md)
[![授權條款: MIT](https://img.shields.io/badge/授權條款-MIT-yellow.svg)](./LICENSE)
[![平台](https://img.shields.io/badge/平台-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey.svg)]()

繁體中文 | [English](README.md)

本專案提供 Antigravity CLI 狀態列（Statusline / Footer）的客製化與語系設定技能，適用於多平台並確保在各種環境下都能維持高效與穩定的顯示。

## 實際畫面 (Screenshot)

### Windows
![Windows 狀態列截圖](path/to/your/windows_screenshot.png) <!-- TODO: 請在此替換為你的 Windows 截圖路徑 -->

### macOS

| 繁體中文 (zh-tw) | English (us) | 日本語 (jp) |
| :---: | :---: | :---: |
| ![繁體中文](docs/images/agy-cli-statusline-macos-zhtw.png) | ![English](docs/images/agy-cli-statusline-macos-us.png) | ![日本語](docs/images/agy-cli-statusline-macos-jp.png) |

## 功能特色

- **豐富的狀態列指標**：可自由選擇顯示以下資訊：
  - 目前使用的 AI 模型名稱
  - 帳號真實 API 可用額度 (Quota)
  - API 重置時間倒數
  - 目前對話已消耗的 Context 比例
  - 目前 Session 消耗的精確 Token 數量
  - CLI 行程所消耗的 RAM 記憶體量
  - 目前工作區專案的 Git 分支
  - 目前工作區專案路徑 (短路徑 / 完整路徑)
- **多國語言支援**：內建繁體中文、英文與日文，並提供讓 AI 一鍵擴充其他語系的動態架構，任何人都能輕鬆新增專屬的語言版本。
- **設定檔動態解析**：自動處理全域 (`~/.gemini/settings.json`)、CLI 專屬 (`~/.gemini/antigravity-cli/settings.json`) 以及專案層級的設定檔。
- **無 Python 依賴的跨平台架構**：捨棄傳統的 Python 依賴。針對 macOS/Linux 使用原生指令 (`ps`, `lsof`)；針對 Windows 更實作了專屬的靜默 C# 橋接器 (Silent C# Bridge) 並結合 `windowsHide`，徹底解決終端機閃爍問題，同時強化對 Git 環境變數缺失與記憶體 (`agy.exe`) 抓取準確度的深度相容。
- **智慧型換行**：自動偵測終端機寬度，避免狀態列內容超出畫面時發生顯示錯亂。
- **動態視覺色彩回饋**：不僅能依據 API 額度或 Context 消耗比例提供四階段（綠、黃、橘、紅）的警告色，也會根據目前使用的 AI 模型家族自動套用專屬的品牌識別色，提供極致直觀的終端機體驗。

## 環境需求 (Prerequisites)

- **Node.js**: 本技能腳本採用純 Node.js (`.mjs`) 實作，您的系統必須已安裝 Node.js 並且可以在終端機執行 `node` 指令。
- **Git** *(選用)*: 狀態列中會讀取目前專案的 Git 分支。若需正常顯示，建議您的系統有安裝 Git（本專案已對 Windows 下未設定環境變數的情況提供了強化相容）。

## 使用方式

1. 請前往本專案的 **[Releases 頁面](../../releases/latest)** 下載最新的發佈壓縮檔（`.zip` 或 `.tar.gz`）。
2. 解壓縮後，將裡面的 `antigravity-cli-statusline` 資料夾直接放入你的 `~/.gemini/skills/` 目錄中。
3. 透過 Antigravity CLI 執行時，直接輸入 `/antigravity-cli-statusline` 即可啟動本技能。

## 貢獻指南 (Contributing)

非常歡迎大家參與貢獻！關於如何提交 PR（Pull Request）、發現 Bug，或是透過 AI 一鍵新增其他語言翻譯，請參閱我們的 **[貢獻指南 (CONTRIBUTING.md)](CONTRIBUTING.md)**。

## 鳴謝

特別感謝 [60ke/antigravity-statusline](https://github.com/60ke/antigravity-statusline) 專案。本專案的額度監控靈感正是來自於此，由於該原專案主要是使用 Python 撰寫，在 Windows 和 macOS 跨平台執行上可能遇到環境設定的困難，因此我使用 JavaScript (Node.js) 進行改寫，以實現真正的跨平台免安裝依賴執行。

## 授權條款

本專案採用 [MIT License](LICENSE) 授權。
