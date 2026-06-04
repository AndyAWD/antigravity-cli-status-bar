---
name: antigravity-cli-statusline
description: 設定 Antigravity CLI 狀態列（Statusline）的顯示項目與語系設定。當使用者要求「設定狀態列」、「調整 CLI 頁尾」或啟動此技能時使用。
---

# Antigravity 狀態列設定技能

本技能提供有關 Antigravity CLI 狀態列（Statusline / Footer）的客製化與語系設定指引，適用於多平台與開源分享。

## 🎯 設定檔路徑規範（動態解析）

為確保取得最新資料與正確寫入，每次執行本技能時，AI 代理（Agent）**必須先動態解析以下路徑並進行讀取（View File）**，嚴禁使用寫死的絕對路徑：

1.  **全域設定檔（Global Settings）**：
    *   路徑語法：`~/.gemini/settings.json`
    *   *AI 執行規範*：請動態獲取當前作業系統的家目錄（環境變數 `$HOME` 或 `USERPROFILE`），並將其展開為當前使用者的絕對路徑後進行讀取。
2.  **CLI 專屬設定檔（CLI-Specific Settings，最高優先級）**：
    *   路徑語法：`~/.gemini/antigravity-cli/settings.json`
    *   *AI 執行規範*：此檔案由 Antigravity CLI（`agy`）自身維護，其優先級**高於**全域設定檔。AI 代理**必須**在 Pre-check 階段讀取此檔案，確認其中是否存在空的 `statusLine` 物件，若存在則需在步驟 5 中同步覆寫為正確的設定。
3.  **專案設定檔（Project Settings）**：
    *   路徑語法：當前工作區根目錄下的 `.gemini/settings.json`
    *   *AI 執行規範*：請動態獲取目前作用中工作區（Workspace）的根目錄絕對路徑，並讀取該目錄下相對路徑 of `.gemini/settings.json`（若專案設定檔存在）。

自訂語系偏好將記錄於設定檔的 `ui.language` 屬性中。

---

## 🛠️ 狀態列 Hook 技術實作規範（核心標準）

為確保狀態列在任何極端環境下都能維持極致效能與穩定美觀，AI 代理在建立或修改 `statusline-quota.mjs` 與背景快取更新程序 `fetch-local-quota.mjs` 時，**必須嚴格遵守以下開發標準**：

1.  **純 Node.js 跨平台實作**：
    *   全面廢棄依賴 Python (`ps`, `lsof`) 的作法。
    *   在 Windows 必須使用 PowerShell 的 `Get-CimInstance Win32_Process` 查詢行程的 ProcessID 與 CommandLine（例如：`Get-CimInstance Win32_Process -Filter "Name like '%agy%'"`）+ `netstat -ano` 以相容於 Windows 10 與 11，在 macOS/Linux 則使用 `ps auxww` + `lsof`。
    *   所有 `fs.writeFileSync` 呼叫都必須明確帶上 `{ encoding: 'utf8' }`，防止在 Windows 環境下生成 UTF-16 導致 CLI 崩潰。
2.  **智慧型多行換行（Smart Line Wrapping by Feature）**：
    *   讀取 `meta.terminal_width` 或 `process.stdout.columns` 來得知終端機寬度。
    *   組合狀態列字串時，如果下一個加入的「指標」（例如：`API 額度: 94.9%`）會讓目前的整行長度（不含 ANSI 碼）超過終端機寬度，就必須將該指標與後續指標折疊到新的一行（即插入 `\n`）。
3.  **四階段精確色彩辨識（4-Tier ANSI Colors）**：
    *   針對百分比設計顏色：`100~75%` = 綠色、`74~50%` = 黃色、`49~25%` = 橘色、`24~0%` = 紅色。

---

## 🔄 執行步驟指南

### 步驟 1：取得最新資料（Pre-check）
在向使用者提問前，**必須動態展開並讀取上述三個設定檔**（全域、CLI 專屬、專案）。檢查目前 `ui.footer.items` 啟用了哪些項目、`ui.language` 設定，以及各設定檔中是否存在空的 `statusLine` 物件。

**【Windows BOM 預檢】**：讀取每一份 `settings.json` 與 `trusted_hooks.json` 時，必須同時檢查檔案的前 3 個位元組是否為 `EF BB BF`（UTF-8 BOM）。若是，請將該檔案路徑記錄為「需於步驟 6 自動修復」的目標。BOM 會導致 agy CLI（Go）於 JSON 解析階段崩潰，報 `invalid character 'ï' looking for beginning of value`，且錯誤訊息完全看不出是編碼問題，極難追查。可使用下列 PowerShell 片段檢測：

```powershell
$b = [System.IO.File]::ReadAllBytes($path)
$hasBOM = ($b.Length -ge 3 -and $b[0] -eq 0xEF -and $b[1] -eq 0xBB -and $b[2] -eq 0xBF)
```

### 步驟 2：第一階段問卷（選擇語系）
呼叫 `ask_question` 工具並傳送以下問卷，收集使用者的語言偏好（由於 Antigravity CLI 原生支援以 `/statusline` 切換啟用或關閉狀態列，因此無需提供冗長的還原選單，最大化簡化問答流程）：

```json
{
  "questions": [
    {
      "question": "選擇顯示語系 / Select Display Language / 表示言語の選択",
      "is_multi_select": false,
      "options": [
        "繁體中文 (zh-tw)",
        "English (us)",
        "日本語 (jp)"
      ]
    }
  ]
}
```

### 步驟 3：流程控制
*   根據第一階段所選取的語言代碼（如 `zh-tw`, `us`, `jp`），動態決定第二階段問卷的顯示語系，並直接進入步驟 4。

### 步驟 4：第二階段問卷（動態語系細部設定 - 勾選啟用項目）
請**根據階段一所選取的語言**，動態組合出以下對應語系的問卷內容，並呼叫 `ask_question` 發送。
由於 `ask_question` 的 `options` 僅支援字串陣列（不支援 `label` / `description` 物件），為同時提供直觀的語言翻譯與系統識別碼，我們統一採用 `「說明文字 (識別碼)」` 格式。

*   若為 `zh-tw`，請使用中文說明配上英文識別碼。
*   若為 `us`，請使用英文說明配上英文識別碼。
*   若為 `jp`，請使用日文說明配上英文識別碼。

由於 CLI 狀態列支援「多行智慧折行」且容納指標無上限，**嚴禁將指標拆分為多個問題發送**。必須將所有指標合併至單一多選問題中，以求極致直觀的 UX 體驗。

以下為 `zh-tw` 的問卷 JSON 結構範例：

```json
{
  "questions": [
    {
      "question": "選擇要顯示的狀態列指標 (下一步將進行排序)",
      "is_multi_select": true,
      "options": [
        "目前使用的 AI 模型名稱 (model-name)",
        "帳號真實 API 可用額度 (quota)",
        "目前對話已消耗的 Context 比例 (context-used)",
        "CLI 行程所消耗的 RAM 記憶體量 (memory-usage)",
        "目前 Session 消耗 of 精確 Token 數量 (token-count)",
        "API 重置時間倒數 (quota-reset-countdown)",
        "目前工作區專案的 Git 分支 (git-branch)",
        "目前工作區專案短路徑 (project-path)",
        "目前工作區專案完整路徑 (project-full-path)",
        "目前訂閱方案等級 (plan-tier)",
        "帳號電子郵件 (account-email)",
        "AI 額度點數 (ai-credits)"
      ]
    }
  ]
}
```

### 步驟 5：第三階段問卷（手動排序與最終篩選）
在向使用者詢問排序前，AI 代理必須將**步驟 4 中使用者勾選的指標**按順序整理出來，並為它們標上數字編號、中文說明（如 `1. 目前使用的 AI 模型名稱 (model-name)`），且**每一項均需以換行符號 `\n` 分隔**。

請根據階段一所選取的語言，動態組合出問卷並呼叫 `ask_question`。在問卷說明中引導使用者透過填寫欄位（Other / Write-in）輸入以逗號分隔的序號或英文識別碼來進行自訂排序（支援混合輸入）。

以下為 `zh-tw` 的問卷 JSON 結構範例（其中 `question` 文字已使用 `\n` 進行換行）：

```json
{
  "questions": [
    {
      "question": "請設定狀態列顯示順序。\n\n目前已選取：\n1. 目前使用的 AI 模型名稱 (model-name)\n2. 帳號真實 API 可用額度 (quota)\n3. 目前對話已消耗的 Context 比例 (context-used)\n\n請在下方輸入框「Write-in...」中輸入以逗號分隔的數字序號或英文識別碼 (如：2, 1, context-used)。未填寫的指標將不予顯示。",
      "is_multi_select": false,
      "options": [
        "略過，使用原勾選順序啟用全部指標",
        "手動排序 (請在下方「Write-in...」欄位中填寫)"
      ]
    }
  ]
}
```

### 步驟 6：設定檔資料合併與寫入

> [!CAUTION]
> **【Windows 寫入 JSON 設定檔的編碼鐵則 — 極度重要】**
>
> Windows 上有多個「預設帶 UTF-8 BOM（`EF BB BF`）」的寫檔路徑，無論代理使用 PowerShell 還是 cmd 觸發都可能中招。agy CLI（Go）的 JSON 解析器遇到 BOM 會直接崩潰報 `invalid character 'ï' looking for beginning of value`，且錯誤訊息完全看不出是編碼問題。AI 代理寫入任何 `.gemini` 之下的 JSON 設定檔（`settings.json`、`trusted_hooks.json` 等）時，**絕對禁止**走下列任一路徑:
>
> - ❌ **PowerShell 5.1**：`Set-Content -Encoding UTF8`、`Out-File -Encoding UTF8`、`Add-Content -Encoding UTF8`、`>` / `>>` 重新導向（**全部帶 BOM 且不可關閉**）。
> - ❌ **PowerShell 7+**：`Set-Content -Encoding UTF8`、`Out-File -Encoding UTF8`（**仍預設帶 BOM**；必須改用 `-Encoding utf8NoBOM`）。
> - ❌ **cmd 經由 PowerShell**：`powershell -Command "Set-Content ..."`、`pwsh -c "Out-File ..."`——表面在 cmd 環境，BOM 來源實際仍是 PowerShell。
> - ❌ **.NET 預設多載**：`[IO.File]::WriteAllText($p, $json)`、`[IO.File]::WriteAllText($p, $json, [Text.Encoding]::UTF8)`（兩者皆帶 BOM）。
> - ❌ **Python**：`open(p, 'w', encoding='utf-8-sig')`（顯式 BOM）、或在 Windows 上未指定 `encoding` 退回系統 locale（如 cp950，會直接破壞 JSON）。
> - ❌ **編輯器**：Notepad「UTF-8 with BOM」、舊版 Visual Studio 新檔預設、`code --wait` 後手動儲存為 UTF-8 with BOM。
> - ❌ **剪貼簿管線**：`clip` 或第三方剪貼簿管理員可能注入 BOM。
>
> **必須改用**以下其中一種「保證不寫 BOM」的方式：
>
> 1. **首選**：Agent 內建的檔案寫入工具（純 UTF-8 無 BOM，不經由 Shell 中介）。
> 2. **次選（PowerShell 7+）**：`Set-Content -Encoding utf8NoBOM`。
> 3. **跨版本保險方案**：`[System.IO.File]::WriteAllText($path, $json, (New-Object System.Text.UTF8Encoding($false)))`——透過明確傳入 `UTF8Encoding($false)`，是 PS 5.1 / PS 7 / .NET Framework / .NET Core 全版本通用的「保證無 BOM」寫法。
> 4. **Node 腳本路徑**：以 `node -e "require('fs').writeFileSync(path, json, {encoding:'utf8'})"` 寫入（Node 的 UTF-8 預設不含 BOM）。
>
> **寫入後驗證（強制）**：每寫完一份設定檔，必須再次讀取前 3 個位元組驗證；若為 `EF BB BF`，必須就地剝除並重寫，直到通過為止。這是 Windows 上唯一可靠的最後一道防線：
>
> ```powershell
> $b = [System.IO.File]::ReadAllBytes($path)
> if ($b.Length -ge 3 -and $b[0] -eq 0xEF -and $b[1] -eq 0xBB -and $b[2] -eq 0xBF) {
>   [System.IO.File]::WriteAllBytes($path, $b[3..($b.Length-1)])
> }
> ```
>
> 若步驟 1 預檢階段曾標記出已被 BOM 污染的設定檔，本步驟必須一併執行上述剝除流程，將既有檔案修正為無 BOM 版本。

*   確定「語言代碼」：採用第一階段問卷的選擇結果。
*   **解析與排序項目清單（混合模式解析）**：
    1.  **提取步驟 4 的基礎勾選項目**：在腦中套用正則表達式 `/\(([^)]+)\)$/`，從步驟 4 勾選的項目字串尾端提取出英文識別碼，得到一個依勾選順序排列的「已勾選英文識別碼陣列」（例如 `['model-name', 'quota', 'memory-usage']`）。
    2.  **讀取步驟 5 的排序輸入**：
        *   若使用者在步驟 5 選擇了預設選項（"略過，使用原勾選順序啟用全部指標"、"Skip" 或其他語系的同義略過項），或者 Write-in 欄位為空白，則**直接使用步驟 4 得到的「已勾選英文識別碼陣列」**作為最終的順序。
        *   若使用者在 Write-in 欄位輸入了自訂排序字串（例如 `"3, model-name, 2"`）：
            1. 將字串以逗號 `,` 拆分，去除每個元素前後的空白。
            2. 遍歷每個拆分項目：
               * 若項目為正整數數字 $N$：
                 * 檢查 $N$ 是否介於 `1` 至「已勾選英文識別碼陣列」的長度之間。
                 * 若符合，則將其對應至該陣列中索引值為 $N-1$ 的英文識別碼。
               * 若項目不是數字：
                 * 檢查其是否精確等於「已勾選英文識別碼陣列」中的某個英文識別碼。
                 * 若符合，則直接使用該識別碼。
               * 若都不符合（如打錯字或數字超出範圍），則直接排除該項目。
            3. **去重（Deduplication）**：若解析出來的識別碼已存在於排序結果中，則予以忽略，以第一個出現的為準。
            4. **捨棄未提及項目**：最終寫入的 `ui.footer.items` **僅保留**此解析排序過程中成功匹配到的英文識別碼，其餘在步驟 4 勾選但未在此處提及的項目將被剔除（不啟用）。
            5. 例如，已勾選為 `['model-name', 'quota', 'memory-usage']`。若排序輸入 `"3, 1, invalid-item"`，則解析結果為 `['memory-usage', 'model-name']`（已排除 `invalid-item`，且未提及的 `quota` 被捨棄）。
*   將 `ui.language` 與解析完成的最終識別碼陣列（`ui.footer.items`）寫入設定檔中。
*   **【關鍵狀態列註冊（多層設定檔同步寫入）】**：Antigravity CLI 有多個設定檔層級。**請務必同時更新以下 3 個設定檔**，確保 `statusLine` 物件存在且正確，否則狀態列將會失效：
    1.  **全域設定檔**：`~/.gemini/settings.json`
    2.  **CLI 專屬設定檔（最高優先級，極度重要）**：`~/.gemini/antigravity-cli/settings.json`
        > [!CAUTION]
        > 這是最容易被忽略的致命盲點！Antigravity CLI（`agy`）**優先讀取此檔案**。若忽略此檔案，全域設定將被無聲無息地覆蓋！AI 代理**必須**在此檔案中也寫入完整的 `statusLine` 設定。
    3.  **專案設定檔**（若存在）：當前工作區根目錄下的 `.gemini/settings.json`

    寫入 `command` 時，**絕對禁止**照抄範例中的 `<真實的使用者家目錄絕對路徑>`，必須將其替換為當下系統環境中真實的絕對路徑（例如 `/Users/andy/.gemini/antigravity-cli/hooks/statusline-quota.mjs`）：

    > [!CAUTION]
    > **Windows 平台關鍵限制與完美越獄（無彈窗版）**：CLI 的 `statusline_runner.go` 底層強制依賴 `sh -c` 執行指令。由於 Windows 預設沒有 `sh.exe`，若直接複製 `powershell.exe` 當作替身，每次狀態列背景更新時都會閃爍一個黑色的終端機視窗。
    > **解決方案**：AI 代理必須找到 CLI 所在的執行目錄（透過 `node -e "console.log(require('path').dirname(process.execPath))"` 取得），並利用 Windows 內建的 `.NET Compiler (csc.exe)` 編譯一個無窗體（`winexe`）的 `sh.exe` 橋接器，徹底消除彈窗。

    *   在 **macOS / Linux** 系統下，寫入：
        ```json
        "statusLine": {
          "enabled": true,
          "type": "command",
          "command": "node <真實的使用者家目錄絕對路徑>/.gemini/antigravity-cli/hooks/statusline-quota.mjs"
        }
        ```
    *   在 **Windows** 系統下，寫入（務必以 `{ encoding: 'utf8' }` 寫入設定檔，並使用正確的雙反斜線跳脫）：
        ```json
        "statusLine": {
          "enabled": true,
          "type": "command",
          "command": "node <真實的使用者家目錄絕對路徑>\\.gemini\\antigravity-cli\\hooks\\statusline-quota.mjs"
        }
        ```

*   **【安全性信任機制註冊（防呆機制）】**：為防止 CLI 將 Hook 靜默阻擋，請同步修改全域的 `~/.gemini/trusted_hooks.json` 檔案。
    在當前工作區目錄、使用者家目錄，以及 `"*"` 鍵值的陣列中，確保加入以下字串（注意：請務必替換為真實絕對路徑）：
    *   macOS/Linux：`"statusLine:node <真實的使用者家目錄絕對路徑>/.gemini/antigravity-cli/hooks/statusline-quota.mjs"`
    *   Windows：`"statusLine:node <真實的使用者家目錄絕對路徑>\\.gemini\\antigravity-cli\\hooks\\statusline-quota.mjs"`（同時追加 forward slash 與 `%USERPROFILE%` 變體以確保相容性）

*   完成寫入後，繼續執行**步驟 7（自動部署跨平台 Hook）**。

### 步驟 7：自動部署 Hook 腳本原始碼（開源通用閉環）

> [!IMPORTANT]
> AI 代理必須動態展開家目錄絕對路徑（如 `join(os.homedir(), '.gemini', 'hooks', ...)`），將以下程式碼寫入對應路徑，且寫入時**務必設定 `{ encoding: 'utf8' }`**。

#### 📂 建立/覆蓋背景快取更新腳本：`~/.gemini/antigravity-cli/hooks/fetch-local-quota.mjs`
請優先讀取當前工作區（Workspace）根目錄下 `scripts/fetch-local-quota.mjs` 檔案（若處於該專案的開發環境）；若檔案不存在或在其他工作區，則退回從本技能的目錄中讀取 `scripts/fetch-local-quota.mjs` 檔案（請自行推導出該檔案的真實絕對路徑），並將其內容寫入到目標路徑。

#### 📂 建立/覆蓋 Hook 主腳本：`~/.gemini/antigravity-cli/hooks/statusline-quota.mjs`
請優先讀取當前工作區（Workspace）根目錄下 `scripts/statusline-quota.mjs` 檔案（若處於該專案的開發環境）；若檔案不存在或在其他工作區，則退回從本技能的目錄中讀取 `scripts/statusline-quota.mjs` 檔案（請自行推導出該檔案的真實絕對路徑），並將其內容寫入到目標路徑。

#### 📂 【僅 Windows】修復 CLI 原生 sh 依賴：編譯靜默版 sh.exe
為了解決 CLI 底層寫死呼叫 `sh -c` 的系統缺陷，且避免使用 PowerShell 造成的終端機閃爍問題，AI 代理在 Windows 平台上必須執行以下步驟，編譯一個靜默無窗體的 `sh.exe`：
1. 透過 `cmd.exe /c where agy` 指令取得 CLI 執行檔的精確路徑，並提取其所在的資料夾（例如 `C:\Users\...\AppData\Local\agy\bin`）。
2. 透過 `node -e "console.log(require('path').dirname(process.execPath))"` 取得當前 Node.js 的全域目錄。若該目錄下存在之前錯誤放置的 `sh.exe`，請**務必**提示使用者手動將其刪除（請提供精確的絕對路徑，例如 `C:\Program Files\nodejs\sh.exe`），以免嚴重污染系統環境。
3. 請優先讀取當前工作區（Workspace）根目錄下 `scripts/sh_hidden.cs` 檔案（若處於該專案的開發環境）；若檔案不存在或在其他工作區，則退回從本技能的目錄中讀取 `scripts/sh_hidden.cs` 檔案（請自行推導出該檔案的真實絕對路徑），將其複製或寫入到暫存資料夾。
4. 使用 Windows 內建的 C# 編譯器（通常位於 `C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe`）將其編譯至真正的 CLI 目錄：
   `C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe /target:winexe /out:"<CLI 執行目錄>\sh.exe" "暫存目錄\sh_hidden.cs"`
這樣當 CLI 發出 `sh -c "node ..."` 時，就會被這個靜默程式完美攔截，在完全沒有黑框閃爍的情況下執行背景更新！

### 步驟 8：回報與重新載入提示
完成檔案編輯與自動 Hook 腳本部署後，回報成功訊息。**AI 代理必須嚴格遵守：根據使用者在第一階段所選擇的語系（zh-tw / us / jp）來撰寫最終的回覆訊息**（例如，若使用者選擇日本語 jp，請務必全程使用流利的日文回覆）。請在回覆中告知使用者：「狀態列設定支援熱更新（Hot Reload），設定已自動在 CLI 底部即時生效，無需重新啟動。」此外，**請在最後溫馨提醒使用者：若他們不再使用舊版的 Gemini CLI，可以自行安全地手動刪除 `~/.gemini/hooks/statusline-quota.mjs` 與 `~/.gemini/hooks/fetch-local-quota.mjs` 這兩個舊版腳本檔，以節省空間並避免混淆。**

---

## 🔌 Dependencies (依賴環境)

本技能在運作與執行時，依賴以下本機系統環境：
*   **Node.js**: Hook 腳本採用純 Node.js (`.mjs` 模組）實作。執行環境中必須能使用 `node` 指令。
*   **作業系統**: 支援 macOS / Linux / Windows 雙平台與跨平台環境。

---

## 🚀 Quick Start (快速啟用)

在新裝置或當前工作區中啟用此技能並設定彩色狀態列，AI 代理只需引導使用者執行：
1. 在聊天框中輸入：`/antigravity-cli-statusline`
2. 在彈出的問卷第一階段選取顯示語系（例如 `zh-tw`）。
3. 在第二階段勾選要啟用的狀態列指標。
4. 在第三階段自訂填入排序（或略過以使用預設順序）。
5. 代理會全自動註冊與部署 Hook，部署完成後狀態列會即時熱更新見效，無需重開 CLI！
6. 在 CLI 執行期間，使用者可隨時執行 `/statusline` 來即時切換開啟或關閉自訂狀態列。

---

## ⚠️ Common Mistakes (常見錯誤與避坑指南)

當 AI 代理執行此技能時，必須時刻檢視並防範以下 4 個極具破壞性的常見錯誤，以保證極致的健壯性：

### 1. 遺漏全域設定檔中的 `statusLine` Hook 註冊
*   ❌ **錯誤行為**：僅在 `settings.json` 中配置 `ui.footer` 設定，卻忘了在根節點下寫入 `"statusLine"` 物件。這會導致在全新/乾淨的電腦上，CLI 根本不會去載入 Hook，自訂狀態列也完全不會顯現。
*   👉 **修正做法**：必須確保在寫入 `settings.json` 時，同步寫入包含 `"enabled": true`、`"type": "command"` 的 `"statusLine"` 物件，並指向正確的 Hook 腳本路徑。

### 2. 環境變數展開失敗與安全匹配失效（跨電腦移植與執行衝突）
*   ❌ **錯誤行為**：
    *   在 `settings.json` 的 `command` 中僅寫入動態環境變數（例如 `node $HOME/.gemini/...`），但 CLI 在背景以直接呼叫方式執行 Hook 時，**無法透過 Shell 展開環境變數**，導致 Node.js 因找不到路徑而執行失敗。
    *   或者，`settings.json` 中使用了動態環境變數，但 `trusted_hooks.json` 僅信任了絕對路徑，導致安全比對字串不匹配而被系統靜默阻擋。
*   👉 **修正做法（雙保險設計）**：
    *   **寫入 `settings.json` 時**：AI 代理應在執行時動態獲取當前使用者的家目錄絕對路徑，並直接將**絕對路徑**寫入 `command` 中（例如 `node /Users/andy/.gemini/...`），這能 100% 確保在本機上執行成功，免去 Shell 展開的依賴。
    *   **寫入 `trusted_hooks.json` 時**：為了同時相容於絕對路徑與未來的動態移植，AI 代理**必須同時追加註冊**絕對路徑指令與環境變數指令（如 `$HOME` 或 `%USERPROFILE%`）的信任規則，以完美通過 CLI 的安全性（Security）檢驗。
    *   **跨電腦移植時**：由於開源分享或複製設定檔到新電腦時絕對路徑會改變，新電腦的 AI 代理在首次執行本技能時，會全自動動態重新解析新電腦的絕對路徑並覆寫更新設定，因此直接寫入絕對路徑是最健壯且無副作用的方案。

### 3. 在 Windows 平台上寫入腳本時遺漏 UTF-8 強制編碼
*   ❌ **錯誤行為**：在 Windows 平台上呼叫 `writeFileSync` 覆寫 Hook 主腳本或快取程式時，沒有指定 `{ encoding: 'utf8' }`。這會使 Windows 預設使用 UTF-16 編碼寫入檔案，導致 agy CLI 啟動時發生解析錯誤並直接崩潰。
*   👉 **修正做法**：每一次進行 `fs.writeFileSync` 或 `fs.writeFile` 寫入時，**必須且強制指定 `{ encoding: 'utf8' }`** 以達成跨平台最高規格防呆。

### 4. 遺漏 CLI 專屬設定檔（`~/.gemini/antigravity-cli/settings.json`）的同步寫入
*   ❌ **錯誤行為**：僅將 `statusLine` 寫入全域設定檔 `~/.gemini/settings.json`，卻忽略了 Antigravity CLI 自身維護的另一份 **CLI 專屬設定檔** `~/.gemini/antigravity-cli/settings.json`。該檔案的優先級**高於**全域設定檔，若其中已存在一個 `statusLine` 物件且 `command` 為空字串（例如 `{ "type": "", "command": "", "enabled": true }`），全域設定檔中正確的 Hook 設定會被完全覆蓋，導致狀態列**無聲無息地徹底失效**，且沒有任何錯誤訊息或日誌輸出，極難排查。
*   👉 **修正做法**：AI 代理在執行步驟 5 時，**必須同時讀取並更新** `~/.gemini/antigravity-cli/settings.json`。若該檔案存在，將正確的 `statusLine` 物件合併寫入；若不存在則跳過。這是本技能中**最致命且最隱蔽的 Bug**，務必嚴格執行。

### 5. AI Agent 憑空捏造或竄改 Hook 原始碼（Hallucination）
*   ❌ **錯誤行為**：AI Agent 接收到部署狀態列的指令時，沒有讀取 `scripts/` 目錄下的標準程式碼檔案，而是直接呼叫 `write_to_file` 或其他工具，將自己「想像」或「重寫」的錯誤程式碼直接寫入 `~/.gemini/antigravity-cli/hooks/statusline-quota.mjs`。這會導致狀態列功能殘缺、指標抓取失敗（例如抓不到 Token 變成 `--`）、甚至導致 CLI 崩潰！
*   👉 **修正做法（絕對禁令）**：AI 代理**絕對禁止**自行發明、撰寫、或修改 `statusline-quota.mjs` 與 `fetch-local-quota.mjs` 的底層程式碼結構來部署！部署時，**必須 100% 準確地讀取本技能目錄下 `scripts/` 資料夾中的對應腳本檔案**，並原封不動地寫入到使用者的 `hooks` 目錄中。若要更新狀態列邏輯，必須先更新對應的 `scripts/*.mjs` 腳本檔案內容，再進行部署，嚴禁憑空發明或竄改邏輯！

### 6. Windows 下將 `settings.json` 寫成 UTF-8 BOM 造成 agy CLI 崩潰
*   ❌ **錯誤行為**：在 Windows 上以任何「預設帶 BOM」的工具寫入 `~/.gemini/settings.json` 或 `~/.gemini/antigravity-cli/settings.json`。常見肇因包含但不限於：PowerShell 5.1 的 `Set-Content` / `Out-File` / `Add-Content -Encoding UTF8`、PowerShell 7+ 的 `-Encoding UTF8`（仍帶 BOM，須改為 `utf8NoBOM`）、cmd 經由 `powershell -c` 間接呼叫上述指令、`[IO.File]::WriteAllText` 不指定編碼或使用 `Encoding.UTF8` 靜態實例、Python `open(..., encoding='utf-8-sig')`、Notepad 另存為「UTF-8 with BOM」等。任一路徑都會在檔頭注入 `EF BB BF`，導致 agy CLI 啟動時於 JSON 解析階段崩潰，錯誤訊息類似 `invalid character 'ï' looking for beginning of value`，且錯誤訊息完全看不出是編碼問題，極難追查。值得注意的是，這是**工具層**而非 Shell 層的陷阱，**改用 cmd 並無法迴避**——只要從 cmd 呼叫到上述任一工具就會中招。
*   👉 **修正做法**：改用「保證不寫 BOM」的寫檔路徑——Agent 內建的檔案寫入工具、`[System.IO.File]::WriteAllText` 搭配 `New-Object System.Text.UTF8Encoding($false)`、PowerShell 7+ 的 `-Encoding utf8NoBOM`，或透過 `node -e` 以 `fs.writeFileSync(..., {encoding:'utf8'})` 寫入。寫入後**必須**讀回前 3 個位元組驗證不是 `EF BB BF`，否則就地剝除並重寫。步驟 1 的 Pre-check 與步驟 6 已內建此防呆流程，務必嚴格遵守。
