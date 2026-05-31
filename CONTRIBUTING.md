# Contributing to Antigravity CLI Statusline

[English](#english-us) | [繁體中文](#繁體中文-zh-tw)

---

## English (us)

### 💡 Add Your Language Translation via AI

You can easily generate a new language translation for this Skill using an AI agent (like Gemini). To ensure even fast models (like Gemini Flash) can process this perfectly, just copy and paste the prompt below.

Replace `[LANG_CODE]` with your target language code (e.g., `ja`), and `[Target Language]` with the language name (e.g., "Japanese"):

````text
I want to add a [Target Language] translation version for this Antigravity CLI Skill (Language Code: [LANG_CODE]).

Since this project dynamically handles languages within single files, please DO NOT create new directories. Please follow these explicit steps to modify the existing files:

### 1. Modify `SKILL.md`
- **Find Step 2 (步驟 2)**: Add the `"[Target Language] ([LANG_CODE])"` option to the `options` JSON array.
- **Find Step 4 (步驟 4)**: Add a bullet point explaining the language format for your code (e.g., `* 若為 [LANG_CODE]，請使用[Target Language]說明配上英文識別碼。`).

### 2. Modify `scripts/statusline-quota.mjs`
- **Find the `const i18n = {` dictionary**: Add a new `[LANG_CODE]` object inside it. Translate all 9 status indicators into [Target Language]. Look at the `zh-tw` or `us` examples and keep the ANSI color variables (like `${BOLD}`) exactly the same.
- **Find the `getGitBranch(lang)` function**: Update **all** the hardcoded ternary operators (there are two places) to include the translation for "No VC" in [Target Language].
- **Find the `countdownVal` variable**: Update the hardcoded ternary operator to include the translation for "N/A" or "None" in [Target Language].
````

### 🤝 Contributing

We welcome contributions! If you have ideas for new features, bug fixes, or improvements, feel free to open an issue or submit a Pull Request. Whether it's adding a new status indicator, improving cross-platform compatibility, or fixing a typo, your help is greatly appreciated.

---

## 繁體中文 (zh-tw)

### 💡 使用 AI 一鍵新增其他語言

你可以使用 AI 助理快速產生其他語言的翻譯版本。為了讓快速模型（如 Gemini Flash）也能精確執行，請直接複製下方的提示詞（Prompt）。

請把 `[LANG_CODE]` 換成你的語系代碼（例如 `ja`），把 `【目標語言】` 換成語言名稱（例如「日文」）：

````text
我想為這個 Antigravity CLI Skill 新增【目標語言】翻譯版本（語系代碼：[LANG_CODE]）。

由於本專案是透過單一檔案動態處理多國語系，請「不要」建立新的資料夾。請嚴格遵守以下步驟，直接修改現有檔案：

### 1. 修改 `SKILL.md`
- **尋找「步驟 2」**：在第一階段問卷 JSON 的 `options` 陣列中，新增 `"【目標語言】 ([LANG_CODE])"` 的選項。
- **尋找「步驟 4」**：在列舉說明格式的地方，補上你的語系（例如：`* 若為 [LANG_CODE]，請使用【目標語言】說明配上英文識別碼。`）。

### 2. 修改 `scripts/statusline-quota.mjs`
- **尋找 `const i18n = {` 字典**：在裡面新增一個 `[LANG_CODE]` 的物件，並把裡面 9 個狀態列指標翻譯成【目標語言】。請參考 `zh-tw` 或 `us` 的格式，務必保留原有的 ANSI 色彩變數。
- **尋找 `getGitBranch(lang)` 函式**：修改裡面**所有**寫死的三元運算子（共兩處），加上【目標語言】對於「無版本控制」的翻譯。
- **尋找 `countdownVal` 變數**：修改後面的三元運算子，加上【目標語言】對於「無 / N/A」的翻譯。
````

### 🤝 貢獻指南

非常歡迎大家提交 PR（Pull Request）來參與貢獻！如果你對這個專案有任何新功能的想法、發現 Bug，或是想要最佳化程式碼，都歡迎隨時發起 PR 或建立 Issue。不論是新增更多的狀態列指標、改善跨平台相容性（Compatibility），或是修正錯字，都非常期待你的加入！
