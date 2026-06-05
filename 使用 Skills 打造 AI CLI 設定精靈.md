# 使用 Skills 打造 AI CLI 設定精靈

###### tags: `GenAI`

***

## 擁抱 AI Agent 時代：新世代「設定精靈」的設計與架構思考

> **主旨：** 探討如何運用 AI Skills（代理技能）設計出高容錯、跨平台且使用者體驗極佳的設定精靈（Setup Wizard），並剖析背後的架構取捨與提示詞（Prompt）工程心法。

---

## 議程 (Agenda)

1. **架構選型之爭**：建構設定精靈的技術典範轉移
2. **漸進式揭露 (Progressive Disclosure)**：與上下文記憶體管理
3. **提示詞工程的護欄 (Guardrails)**：馴服異質模型的認知邊界
4. **理論依據與底層原理解析 (The Underlying Mechanics)**
5. **實戰總結**：建構高階設定精靈的系統性 Check List

---

## 1. 架構選型之爭：建構設定精靈的技術典範轉移

在開發互動式設定引導（Setup Wizard）——如 CLI 環境初始化、複雜系統依賴配置時，我們正經歷從「決定論程式設計」向「機率性代理工作流」的技術典範轉移。以下深度剖析三種主要架構的優劣與底層特性：

### 1.1 傳統硬編碼腳本 (Hardcoded Deterministic Scripts)
傳統上，開發者依賴 Bash, Node.js (`inquirer.js`), 或 Python 來撰寫互動式腳本。
*   **技術優勢 (Advantages)**：
    *   **絕對決定性 (Absolute Determinism)**：執行緒（Process）的狀態轉移 100% 可預測，無「幻覺（Hallucination）」風險。
    *   **低延遲與封閉性 (Low Latency & Encapsulation)**：執行速度達毫秒級，且完全不依賴外部 LLM 推理 API，具備高度安全與隱私性。
*   **架構痛點 (Pain Points)**：
    *   **環境異質性成本 (Environmental Heterogeneity Cost)**：開發者必須窮舉並處理所有跨平台邊界情況（Edge Cases），如 Windows 的 `\` 與 UNIX 的 `/` 路徑解析差異、各種 Shell 環境變數的展開規則。
    *   **狀態剛性 (State Rigidity)**：缺乏自癒能力（Self-Healing）。當遇到未預期例外（如目錄缺失、權限不足）時，腳本通常會拋出 Fatal Error 並中斷，無法動態尋找替代路徑。
    *   **語意僵化 (Semantic Inflexibility)**：多語系（i18n）支援需耗費大量資源維護靜態字典檔，且無法理解並回應使用者在設定過程中的自由發問（Free-form Inquiries）。

### 1.2 純 AI 代理工作流 (Pure Prompt-Based Agentic Workflow)
將所有設定邏輯僅透過一份 Prompt 交付給 LLM 代理執行。
*   **技術優勢 (Advantages)**：
    *   **強大的拓撲自癒力 (Topological Self-Healing)**：Agent 能利用其內建的「世界知識 (World Knowledge)」動態解決環境阻礙（例如：發現無目錄則自動呼叫 `mkdir`），實現跨平台相容性的降維打擊。
    *   **動態語意介面 (Dynamic Semantic UI)**：天生具備零成本的多國語言適應力，並能在流程中動態穿插知識解答，提供極佳的使用者體驗（UX）。
*   **架構痛點 (Pain Points)**：
    *   **推理延遲與成本 (Inference Latency & Cost)**：每個決策節點皆需經過神經網路的前向傳播（Forward Pass），受限於網路 I/O 與 Token 計費。
    *   **操作非決定性 (Operational Non-determinism)**：在執行高頻率、需對底層資源進行絕對精確控制的任務（如即時狀態列更新）時，LLM 產生的微小輸出偏差可能導致毀滅性的系統崩潰。

### 1.3 終極解答：大腦與肌肉解耦的混合架構 (Decoupled Hybrid Architecture)
當代最先進的實踐是**混合架構 (Hybrid Approach)**，其核心思想是將「決策大腦 (Cognitive Brain)」與「執行肌肉 (Execution Muscle)」進行解耦（Decoupling）：
1.  **AI Skills 擔任協調者 (Orchestrator)**：利用宣告式（Declarative）的 `SKILL.md` 負責高維度的流程控制、跨平台路徑推導、多語系互動，以及錯誤的動態修復。
2.  **腳本擔任執行端點 (Execution Endpoint)**：對於底層的、不容出錯的核心業務邏輯（例如跨行程通訊、系統資源監控），由 AI 在設定過程中，將預先驗證過（Pre-validated）的靜態腳本準確部署至本機執行。
*   **結論**：此架構完美融合了 LLM 的高階容錯力與靜態程式碼的底層穩定性，是建構新世代互動工具的黃金標準。

---

## 2. 漸進式揭露 (Progressive Disclosure) 與上下文記憶體管理

在混合架構中，我們不可避免地需要將實作腳本（如 `.mjs`, `.py`）交付給目標環境。然而，如何「傳遞」這些腳本，直接關係到 Agent 的運行效率與推理品質。

### 2.1 Context 污染與「注意力稀釋 (Attention Dilution)」危機
若將數百行的腳本原始碼直接嵌入 `SKILL.md`（Prompt）中，會引發嚴重的**上下文污染 (Context Pollution)**：
*   **Token 浪費**：這些靜態程式碼在 Agent 進行高階流程決策時毫無用處，卻會持續消耗系統的 Context Window 額度。
*   **「Copy-Paste 症候群」與幻覺**：長篇的程式碼區塊極易干擾 LLM 的注意力權重分配。模型可能會在部署階段分心，產生「幻覺性重構 (Hallucinatory Refactoring)」——即擅自修改、發明或省略腳本中的某些關鍵邏輯，導致部署出殘缺的程式碼。

### 2.2 實踐漸進式揭露：抽離與參照 (Extraction and Referencing)
**解法方案**：必須嚴格遵守軟體工程的**關注點分離 (Separation of Concerns)** 原則。
1.  **實體抽離 (Physical Extraction)**：將所有可執行的程式碼、長篇參考資料（Schemas）從 `SKILL.md` 中移除，存入獨立的 `scripts/` 或 `references/` 目錄。
2.  **指標性參照 (Pointer Referencing)**：在 `SKILL.md` 中，僅保留對該檔案的**明確讀取指令**（例如：`請使用 read_file 工具讀取腳本 scripts/xxx.mjs 並寫入目標位置`）。
*   **架構效益**：透過這種「延遲載入 (Lazy Loading)」的機制，`SKILL.md` 得以保持極致精簡（Concise is Key）。LLM 只在「真正需要部署的那一刻」才會去獲取腳本內容，大幅降低了模型的認知負擔，並徹底根絕了竄改腳本的幻覺風險。

### 2.3 漸進式揭露的反例：高頻 + 逐字內容的「子代理摘要陷阱 (Sub-agent Summarization Trap)」

漸進式揭露是 Antigravity 官方支援的設計模型（如 [官方 Skills 文件](https://antigravity.google/docs/skills) 所述：代理初始僅載入 frontmatter，技能被選中後才載入完整 `SKILL.md` body，而 `scripts/` 與 `references/` 則按需執行或讀取）。其核心承諾是「延遲載入」（Lazy Loading），但這個機制存在一個**官方文件未明文涵蓋、卻在跨代理實作上普遍可見的隱性失效模式**：當主代理透過子代理（Sub-agent / Explore Agent）讀取 `references/` 時，回傳的常是「自然語言摘要」而非原文。

> **注意（跨代理相容性）**：agy CLI 官方文件目前未明確規範 `references/` 是由主代理直接讀取或可委派子代理。但若本技能被 Claude Code 等其他支援 Skills 規範的代理載入（透過共享 Skills 目錄結構），子代理摘要行為就會浮現。因此即使單純為 agy CLI 設計，仍建議實作此防禦，以保留跨代理可移植性。

對於只需傳達語意的描述性段落，這個機制沒問題；但對於**需要逐字呼叫**的內容（如 agy CLI `ask_question` 工具的 JSON Schema、PowerShell 編碼鐵則的精確語法、跨平台路徑跳脫範本），子代理會自動「整理」、「精簡」、或將不熟悉格式「翻譯」成自己熟悉的標準，導致原文細節徹底遺失。

**陷阱的三重觸發路徑 (Triple Triggers)**：

1. **子代理摘要傳回 (Sub-agent Summarization)**：主代理委派 Explore 子代理讀取 reference 後，回傳的可能僅是「裡面是一份繁中問卷，三個語言選項」，原始 JSON 引號、`questions[].is_multi_select`、字串陣列 `options`、以及頂層必填的 `toolSummary` / `toolAction` 欄位就此消失。
2. **長對話 Context 壓縮 (Long-context Compaction)**：即使主代理親自 Read 載入了原文，當對話 Context 逼近上限觸發壓縮機制時，靜態 JSON 區塊極易被壓縮層判定為「資訊密度低的可摘要文字」而被自動改寫；尤其像 `toolSummary` / `toolAction` 這類看似可被推導的 metadata 欄位，最易在壓縮中被誤刪。
3. **異質格式自動「修正」(Cross-CLI Format Coercion)**：當主代理（例如 Claude）誤將 agy CLI 的 `ask_question` 識別為自身熟悉的 `AskUserQuestion`，會基於指令依從性自動把字串陣列 `options` 改寫為帶 `label` + `description` 的物件陣列、把 `is_multi_select` 改成 `multiSelect`、並丟棄 `toolSummary` / `toolAction`——表面合理、實際完全壞掉。
   * **agy CLI 正確規格（必須 verbatim 保留）**：頂層含 `questions[]`、`toolSummary`（2~5 字名詞片語）、`toolAction`（2~5 字句首動作）；每題含 `question`、`options`（純字串陣列，至少 2 項，**禁止手動添加「其他/Other」**因系統自動提供 Write-in）、可選 `is_multi_select`（預設 `false`）。強烈建議選項可在字串開頭加 `(Recommended)` 前綴讓 UI 標示。

**防禦架構：高頻逐字內容必須回流主檔 + 中頻內容加 Verbatim 護欄**

漸進式揭露不能無腦套用。在拆檔之前，必須對每塊內容做兩維度評估：

| 維度 1：呼叫頻率 | 維度 2：逐字精度需求 | 拆檔策略 |
|---|---|---|
| 高（每次必執行）| 高（必須逐字呼叫）| **強制留在 SKILL.md 主檔**（如 `ask_question` JSON）|
| 中（條件觸發）| 高（必須逐字呼叫）| 留在 `references/` 但檔頭加上「禁止子代理摘要 + 禁止改寫」的明確 Verbatim 護欄 |
| 任意頻率 | 低（可摘要重述）| 適合拆至 `references/`，可由子代理自由總結 |

對於必須留在 reference 的高精度內容，應在檔頭加入下列宣告，明確要求主代理親自處理：

```markdown
> [!IMPORTANT]
> 本檔案含逐字（verbatim）使用的指令片段與 JSON 結構。
> 1. 禁止透過 Explore / 子代理摘要本檔案 — 主代理必須親自以 Read 工具讀取
> 2. 禁止改寫、簡化、重新格式化任何程式碼區塊
> 3. 禁止將指令翻譯成「等效」做法
> 4. 若內容與你熟悉的其他 CLI / API 格式不同，一律以本檔案為準
```

**架構意義 (Architectural Significance)**：這揭示了漸進式揭露的設計邊界——它是 **Context 經濟（Context Economy）**與**內容保真度（Content Fidelity）**的權衡，而非一刀切的最佳化原則。架構師在拆檔前必須先問：「這塊內容若被摘要傳回，下游動作會不會壞？」若答案為 Yes，則必須回流主檔或加上 Verbatim 護欄。最終的設計哲學是：**逐字使用的內容靠近執行點、可摘要的內容才適合延遲載入**。

---

## 3. 提示詞工程的護欄 (Guardrails)：馴服異質模型的認知邊界

開發可廣泛發布的 Agent Skill 時，最大的變數在於「基礎模型（Foundation Model）的異質性」。不同的模型具有截然不同的認知偏好與行動閾值，必須透過精密的**防禦性提示詞 (Defensive Prompting)** 來建立穩固的護欄。

### 3.1 模型行為光譜分析 (Behavioral Spectrum Analysis)
*   **高行動力模型 (如 Gemini 3.1/3.5 Flash)**：
    *   **認知特徵**：推理速度極快，對工具呼叫（Tool-use）具有極高的傾向性（Bias toward Action）。
    *   **行為陷阱：過度積極 (Overzealousness)**。面對單純的正則表達式，會傾向於動用外部工具（如撰寫 Python 腳本）來解題；面對變數佔位符（如 `[使用者名稱]`），可能為了追求速度而跳過環境變數展開，直接進行字面映射（Literal Mapping）。
*   **高推理力模型 (如 Gemini 3.1 Pro)**：
    *   **認知特徵**：邏輯網絡深廣，具備強大的長文本全局推理與自我約束能力。
    *   **行為陷阱：過度解讀 (Over-interpretation)**。當給予過高的自由度時，可能會因為試圖優化流程而偏離原本的宣告式路徑；而在遭遇過度嚴苛的防呆限制時，則可能顯得決策死板。

### 3.2 跨模型相容的「木桶效應 (Lowest Common Denominator)」防禦策略
為確保 Skill 的泛用性，必須以約束高行動力模型（Flash）為設計基準，實施以下核心護欄策略：
1.  **收束過度行動力（禁止工具幻覺）**：
    *   ❌ *高風險寫法*：`提取識別碼，請使用正則表達式 /\(([^)]+)\)$/`
    *   ✅ *高防禦寫法*：`請您在腦內參數層（作為純文字推理邏輯）套用正則表達式提取識別碼。⚠️ 防呆警告：絕對禁止為此呼叫工具撰寫或執行任何 Node.js/Python 腳本。`
    *   *架構意義*：強制將該任務從「External Execution」降級為「Internal Reasoning」，有效節省 API I/O 成本並消除腳本生成錯誤率。
2.  **打破字面依從性（佔位符防呆）**：
    *   ❌ *高風險寫法*：`寫入路徑 node /Users/[動態解析的使用者名稱]/.gemini/...`
    *   ✅ *高防禦寫法*：`寫入路徑 node <真實的使用者家目錄絕對路徑>/.gemini/...。⚠️ 絕對禁止字面照抄範例，必須動態推導並替換為系統真實路徑！`
    *   *架構意義*：利用極端視覺特徵（尖括號）與強烈否定詞，打破模型在快速推理下的「Copy-Paste 捷徑思維」，強制喚醒其環境變數推導邏輯。

---

## 4. 理論依據與底層原理解析 (The Underlying Mechanics)

要設計出穩健的 Agent Skill，不能僅依靠試錯（Trial and Error），必須深刻理解大型語言模型（LLM）在代理（Agentic）環境下的神經網路行為特徵。以下針對上述防呆技巧，提供深度的底層原理解析，供未來的系統架構師與 AI 代理參考。

### 4.1 介面統一的陷阱與「意圖解析衝突 (Intent Parsing Conflict)」
在現代 Agent 架構（如 Gemini CLI）中，為了降低使用者的認知負擔，系統常採用**介面統一 (Interface Unification)** 的設計：將傳統的底層系統指令（如 `/clear`）與自訂技能的喚醒路徑（如 `/my-skill`）整合為相同的斜線語法（Slash Shortcuts）。

然而，這在架構上是一個**兩階段管線 (Two-Stage Pipeline)**：
1.  **系統層次攔截 (System-Level Routing)**：CLI 攔截到斜線輸入，作為一個**硬性觸發器 (Hard Trigger)**，將對應的 `SKILL.md` 動態載入並推入 LLM 的 Context Window 中。此階段是決定論的 (Deterministic)。
2.  **神經網路意圖解析 (Neural Intent Parsing)**：LLM 接收到包含了 `SKILL.md` 的 Context 後，開始進行意圖推斷。

**底層衝突的根源**：LLM 具有高度的**指令依從性 (Instruction Following)** 權重。如果 `SKILL.md` 內包含過於嚴謹的假範例（例如：`## 使用方式：輸入 "/my-skill 開始執行" 才會啟動`），LLM 的推理引擎會比對使用者實際輸入（僅有 `/my-skill`）與說明文件。由於缺乏 `開始執行` 四個字，LLM 會基於保守策略判定「觸發條件未完全滿足」，進而進入待命狀態。
**解決架構**：必須在 Prompt 中實作**強制對齊 (Alignment Override)**。透過宣告「一旦載入即視為觸發，強制忽略後續字面比對，直接進入 Step 0」，我們在意圖解析層直接覆寫了預設的保守策略，確保了從系統層到認知層的順暢傳遞。

### 4.2 克服「注意力發散」：將文字映射為有限狀態機 (FSM Mapping)
LLM 處理長文本的基礎是注意力機制 (Attention Mechanism)。當 `SKILL.md` 內包含大量的文字化條件分支（例如：「如果 A 條件成立，請執行步驟 X；若 B 條件成立，則需先檢查 C 才能執行 Y...」）時，模型對這些跨段落依賴條件的 Attention 權重容易被稀釋，導致**文字迷航 (Lost in the Middle)** 或執行順序錯亂。

**結構化解法 (Structural Prompting)**：
在文件中引入 ASCII 樹狀圖（如 `├─` , `└─` , `→`），並非單純為了排版美觀。對於 LLM 而言，這些具備強烈幾何特徵與結構性的符號，能促使模型將該區塊辨識為一個**有限狀態機 (Finite State Machine, FSM)** 的拓撲圖。
在此模式下，LLM 的行為邏輯會從「連續性的語意生成」轉變為「離散的節點跳轉 (Node Traversal)」。每完成一個動作，模型會回頭對照（Attend to）這張結構化地圖，尋找目前的狀態節點與下一個邊界條件（Edge）。這等同於在純粹的 Prompt 層次上，低成本地實作了高階 Agent 框架（如 LangGraph）所提供的**路由控制 (Routing Control)** 能力。

### 4.3 工具調用的邊界約束與「適當自由度 (Appropriate Degrees of Freedom)」
在賦予了工具調用（Tool-use）能力的 Agent 環境中（模型可以自由決定是否呼叫 `read_file`, `run_shell_command` 等 API），模型的**行動力閾值 (Action Threshold)** 會大幅降低。
特別是針對基礎智力與推理速度極高的模型（如 Gemini Flash 系列），當它們面對諸如字串擷取、正則表達式等任務時，往往會產生**幻覺性過度優化 (Overzealous Optimization Hallucination)**——誤以為必須透過撰寫並執行外部 Python/Node.js 腳本來達成這項任務，而不依賴自身的腦內神經推理 (Internal Reasoning)。

**防禦性設計 (Defensive Guardrails)**：
此時，我們必須在 Prompt 中實行**嚴格的邊界約束**（例如：`⚠️ 防呆警告：絕對不要為此撰寫腳本，僅需文字處理`）。
這在架構設計上稱為**設定適當的自由度 (Setting Appropriate Degrees of Freedom)**。對於這類只需單純萃取字串的「低脆弱性任務」，我們必須收束其自由度，關閉該節點的工具調用權限，強迫模型使用內部參數記憶（腦內處理）來完成。這不僅能大幅節省 API 呼叫輪數與 Token 成本，更能防範模型寫出錯誤腳本而導致整個任務管線崩潰。這種「降級相容」的護欄設計，是確保單一 `SKILL.md` 能穩定運行於各種大小模型上的不二法門。

### 4.4 跨平台邊界防禦與作業系統陷阱 (Cross-Platform Boundary Defenses & OS Pitfalls)
在部署跨平台 Hook 腳本時，AI 代理往往會遭遇作業系統底層架構的巨大差異。若未在 `SKILL.md` 的設計階段建立防禦機制，極易引發效能低落或使用者體驗崩壞。以下為實戰中必須克服的三大 Windows 環境陷阱：

1. **靜默執行與終端機閃爍 (Terminal Flashing & Silent Execution)**：
   * **陷阱**：在 macOS/Linux 中，背景執行子程序通常是無縫且靜默的。但在 Windows 環境中，呼叫如 `powershell.exe`、`cmd.exe` 等 Console Subsystem 應用程式，或是在 Node.js 中使用 `child_process.execSync` 呼叫外部指令（如 `git`, `wmic`），作業系統預設會瞬間彈出並關閉一個黑色終端機視窗。這在需要高頻率背景輪詢（Polling）的狀態列應用中，會造成毀滅性的視覺干擾（持續閃爍）。
   * **防禦架構**：必須將「靜默執行」內化為腳本的底層規範。
     * 對於 Node.js 的系統呼叫，強制規定所有 `spawn` 與 `execSync` 必須攜帶 `{ windowsHide: true }` 參數。
     * 對於缺失的系統依賴（如 Windows 缺乏 `sh.exe`），絕對禁止直接複製 `powershell.exe` 充當替身。AI 代理必須被賦予編譯能力，利用 Windows 內建的 `.NET Compiler (csc.exe)`，現場編譯出一個指定為 `/target:winexe`（GUI Subsystem，天生無窗體）的專屬 C# 橋接器，徹底根絕閃爍。

2. **處理程序層級樹狀結構 (Process Tree Hierarchy) 的記憶體失真**：
   * **陷阱**：在 UNIX 系統中，`process.ppid` 通常能精準指向喚醒該腳本的母程序。但在 Windows 中，因為常需透過 `cmd.exe` 或自製的橋接器代為轉發指令，執行鏈會變成 `主程式 -> sh.exe -> cmd.exe -> node.exe`。此時 Node.js 抓到的 `ppid` 會是 `cmd.exe`，導致計算出的記憶體用量從真實的數百 MB 暴跌至 7MB 的嚴重失真。
   * **防禦架構**：在設計系統狀態偵測腳本時，必須強制解耦對 `ppid` 的依賴。針對 Windows，應改為直接掃描行程名稱（如 `wmic process where "name='主程式.exe'"`），透過確切的名稱辨識來加總記憶體，確保跨平台數據的一致性與精準度。

3. **環境變數依賴與預設路徑自癒 (PATH Dependency & Auto-fallback)**：
   * **陷阱**：依賴系統環境變數 (`%PATH%`) 來尋找外部指令（如 `git`）是極度脆弱的。許多 Windows 使用者安裝軟體時並未將其加入 PATH，導致腳本拋出 `ENOENT` 錯誤並功能降級。
   * **防禦架構**：AI 技能腳本必須內建**自動尋路 (Auto-fallback)** 機制。在攔截到指令遺失的例外錯誤時，不應直接報錯，而是要透過 `process.platform === 'win32'` 判斷，自動前往標準預設路徑（如 `C:\Program Files\Git\cmd\git.exe`）進行第二階段的強行調用，達成開箱即用的極致容錯率。

4. **文字編碼預設與 UTF-8 BOM 污染 (Encoding Defaults & UTF-8 BOM Contamination)**：
   * **陷阱**：在 UNIX 環境中寫入 UTF-8 文字檔不會帶上 Byte Order Mark（BOM），但 Windows 上**存在一整個生態系預設帶 BOM 的寫檔路徑**：PowerShell 5.1 的 `Set-Content` / `Out-File` / `Add-Content -Encoding UTF8` 全部強制注入 `EF BB BF` 且無法關閉；PowerShell 7+ 的 `-Encoding UTF8` 仍預設帶 BOM（須改用 `utf8NoBOM`）；.NET 的 `[IO.File]::WriteAllText` 在不顯式傳入 `UTF8Encoding($false)` 時也帶 BOM；Python 的 `open(..., encoding='utf-8-sig')` 顯式帶 BOM；Notepad「另存為 UTF-8 with BOM」亦同。值得注意的是，**cmd 本身的 `echo > file` 重新導向並不會加 BOM**，但只要從 cmd 呼叫了上述任一工具（例如 `powershell -c "Set-Content..."`），BOM 就會被注入——因此這個陷阱屬於**工具層**而非 Shell 層，無法靠「改用 cmd」迴避。當 AI 代理寫了一份「會被 Go 程式（如 agy CLI、Hashicorp 工具鏈、多數 Go CLI）解析的 JSON 設定檔」時，下次 CLI 啟動就會在 `json.Unmarshal` 階段崩潰，報 `invalid character 'ï' looking for beginning of value`，而且錯誤訊息完全看不出是編碼問題，極難追查。這個陷阱與 §3.1 的「過度積極」高度耦合——當代理在 Windows 上直覺伸手呼叫 PowerShell、.NET API 或 Python helper 來處理文字檔時，就會踩中。
   * **防禦架構**：必須在 `SKILL.md` 層級對「寫設定檔」這個動作做工具層的明確收束：
     * **第一道防線（工具選擇）**：禁止使用 PowerShell 的 `Set-Content` / `Out-File` / `Add-Content -Encoding UTF8` 與 `>` / `>>` 重新導向來寫任何被下游 Go/Rust/嚴格 JSON 解析器消費的設定檔。一律改用 Agent 框架內建的純 UTF-8 無 BOM 寫檔工具（或透過 `node -e` 呼叫 `fs.writeFileSync` 寫入，Node 的 UTF-8 預設不含 BOM），必要時可使用 `[System.IO.File]::WriteAllText($path, $json, (New-Object System.Text.UTF8Encoding($false)))`——後者透過明確傳入 `UTF8Encoding($false)`，是 PS 5.1 / PS 7 / .NET Framework / .NET Core 全版本通用的「保證無 BOM」寫法。
     * **第二道防線（寫入後驗證）**：每次寫完設定檔後，強制讀回前 3 個位元組與 `EF BB BF` 比對；若命中即就地剝除並重寫。這道驗證的價值在於：即便代理在某次推理中違反了第一道防線，或設定檔已被歷史污染，第二道防線也能在 Skill 結束前自動清乾淨，達成**自癒式（Self-Healing）跨平台容錯**。
   * **架構意義**：這個案例揭示了一個更普遍的原理——在跨平台 Agent 設計中，「文字檔」並不是平台無關的抽象，它的編碼預設行為會隨 OS 與 Shell 版本而變動，而且這些差異往往**只在被下游嚴格解析器消費時才會現形**。因此凡是要被本機原生程式（尤其 Go/Rust 等對 BOM 敏感的語言生態）讀取的設定檔，都必須在 Skill 中做「寫入工具白名單 + 寫入後位元組驗證」雙保險，不能僅依賴「我寫的時候有指定 UTF-8」這種一廂情願的假設。

5. **作業系統指令棄用危機 (OS Command Deprecation Crises)**：
   * **陷阱**：在跨平台環境中，依賴特定系統內建工具（如 Windows 上的 `wmic`）來獲取行程（Process）資訊是危險的。微軟已在現代 Windows（如 Windows 11）中預設棄用並移除了 `wmic` 工具。若提示詞（Prompt）或腳本硬編碼強依賴此工具，會直接導致例外崩潰，使得狀態列（Statusline）在背景獲取進程 PID 與記憶體用量時失敗，無聲無息地回傳「未知」與「無」。
    * **防禦架構**：在 Windows 上推薦直接改用 PowerShell 的 `Get-CimInstance Win32_Process` 命令（例如：`Get-CimInstance Win32_Process -Filter "Name like '%agy%'"`）來取代 `wmic`。此命令同時內建於 Windows 10 與 Windows 11，且能穩定獲取 `ProcessID` 與 `CommandLine`（用以解析 `--csrf_token` 參數），是實現高相容性、零 dependency 的跨 Windows 世代黃金方案。

6. **靜默覆寫與開發環境腳本複製漏洞 (Development Overwrite & Local Script Sync Holes)**：
   * **陷阱**：在混合架構中，若 AI 技能提示詞中寫死「直接從本技能的目錄（即全域技能存放目錄）讀取腳本進行部署」，這會產生致命的**開發期覆寫漏洞**。當開發者正在當前專案工作區開發與修改最新版的腳本時，AI 代理一旦被喚醒執行該技能，就會無視工作區最新變更，固執地從全域目錄拉取舊腳本將其覆寫回去。
   * **防禦架構**：提示詞中必須實作**工作區優先（Workspace-First）路由**。在 Prompt 內指引代理「優先讀取目前作用中工作區下的 `scripts/` 目錄檔案，若不存在才退回到技能目錄讀取」，確保開發中的最新代碼能被即時部署與測試。

### 4.5 突破工作區盲區：.gitignore 的資安與讀取悖論 (Overcoming Workspace Blind Spots: The .gitignore Paradox)

在處理如 `local.properties`、`.env` 等包含敏感金鑰（API Key）的設定檔時，開發者通常會基於資安考量將這些檔案加入 `.gitignore` 中。然而，這會引發一個架構上的悖論：現代 AI Agent 在進行 Workspace 掃描或使用搜尋工具（如 `grep_search`）時，底層邏輯會預設遵循 `.gitignore` 的規則，導致這些包含金鑰的檔案成為 Agent 的「視覺盲區（Blind Spots）」，進而引發讀取失敗或產生未設定的幻覺。

**防禦性設計與繞過策略 (Defensive Design & Bypass Strategy)**：
為了解決這個矛盾，我們必須在 `SKILL.md` 中對 Agent 下達明確的「盲區突破」指令，強行繞過預設的忽略機制：
1. **強制停用搜尋忽略 (Disable Ignore Rules)**：明確指示 Agent 在使用全域搜尋工具尋找金鑰時，必須將搜尋參數（如 `no_ignore: true`）強制開啟，以納入被 Git 隱藏的檔案。
2. **精確的絕對路徑讀取 (Explicit Path Reading)**：要求 Agent 完全放棄依賴模糊搜尋，改為直接調用 `Read` 工具（如 `read_file`）並傳入檔案的「絕對路徑」進行精確打擊。由於直接的檔案讀取 API 通常不受 Git 索引限制，這能確保 Agent 安全且穩定地取得隱藏在 `.gitignore` 背後的敏感配置。

### 4.6 零信任架構：API Key 外洩風險與權限隔離 (Zero-Trust Architecture: Key Leakage & Privilege Isolation)

當我們成功讓 Agent 突破盲區讀取 `.gitignore` 內的敏感金鑰後，隨之而來的是嚴峻的**資安外洩風險 (Data Leakage Risk)**。一旦 Agent 使用 `read_file` 等工具讀取了 Key，這把 Key 就會被明文寫入 Context Window，引發以下風險：
1. **雲端模型曝露**：Key 會作為 Prompt 的一部份傳送至 LLM 供應商的伺服器。
2. **本地日誌污染**：CLI 通常會在本地隱藏目錄（如 `.gemini/` 或 `.claude/`）儲存對話紀錄，若未妥善加入 `.gitignore`，極易在 Commit 時被推送到版控系統。
3. **幻覺式外洩 (Hallucination Leak)**：Agent 可能會在後續對話中，因為幻覺而無意間將記憶中的 Key 明文寫入其他公開文件（如 `README.md`）或印在終端機上。

**終極防禦架構：大腦與肌肉的權限隔離**
為了達到最嚴謹的資安標準（Zero-Trust），**我們根本不應該讓 AI Agent（大腦）去直接讀取 Key**。

更安全的系統設計應採用「權限隔離」：
1. **認知解耦**：在 `SKILL.md` 中，Agent 不需要知道 Key 的實際內容，它只需要知道「Key 存放在 `local.properties` 且環境已就緒」。
2. **底層腳本代勞**：當需要執行敏感 API 請求時，Agent 僅負責組裝並呼叫底層腳本（肌肉），例如 `node scripts/analyze.js`。
3. **本地環境閉環**：由該腳本在本地執行環境中，**透過程式碼自行讀取**設定檔並發送請求。

透過這種架構，敏感憑證永遠只在「本地硬碟」與「腳本執行緒」中流動，100% 阻斷了 Key 進入 LLM Context Window 的途徑，徹底消除從雲端到本地端的所有外洩隱患。

### 4.7 環境預檢與依賴降級防護 (Environmental Pre-checks & Dependency Degradation Protection)

在設計依賴外部執行環境（如 Node.js、Python）的 AI 代理技能時，最忌諱「假設環境已就緒」。若未在執行初期進行預檢，腳本部署後可能會引發難以追查的底層系統錯誤（如 `exit status 127`），導致功能無聲無息地失效，嚴重破壞使用者體驗。

**防禦性設計與漸進式降級 (Defensive Design & Graceful Degradation)**：
必須在 `SKILL.md` 的最前線（Pre-check 階段）強制代理執行環境依賴掃描，並建立完善的降級處理流程：
1. **跨平台動態偵測**：要求代理使用跨平台通用的指令（如 `node --version`）或依作業系統分支使用對應指令（macOS/Linux 的 `which`、Windows 的 `where`），動態驗證環境是否具備。
2. **故障模式的透明化 (Transparency of Failure Modes)**：當偵測到依賴缺失時，Agent 絕不能直接報錯中斷，而必須向使用者清楚說明**後果**（例如：「缺少 Node.js 將導致 CLI 狀態列完全空白且自動停用」），讓使用者理解錯誤的根源。
3. **互動式決策中斷點 (Interactive Decision Breakpoint)**：透過呼叫互動工具（如 `ask_question`），將控制權交還給使用者。提供「中斷並安裝依賴」或「強制繼續設定（預寫入配置）」的選項。這種設計確保了即使在環境殘缺的狀態下，Agent 也能完成資料配置，待使用者後續補齊環境後即可無縫生效，達成極致的容錯與 UX 體驗。

---

## 5. 實戰總結：建構高階設定精靈的系統性 Check List

在交付任何基於 AI Agent 的設定精靈之前，系統架構師應依據此清單進行最終的架構審查（Architectural Review）：

1.  [ ] **拓撲結構釐清 (Topological Assessment)**：評估任務屬性。若為具備複雜分支與跳轉的非線性任務，是否已導入 ASCII 狀態機（FSM）以建立路由控制？若為純線性任務，章節標籤是否具備足夠的錨點效應？
2.  [ ] **架構解耦驗證 (Architectural Decoupling)**：是否確實貫徹了「混合架構」精神？高頻率的底層核心運算是否已從 Prompt 中抽離為靜態腳本（`scripts/` 目錄），僅保留高維度的對話與部署邏輯給 AI 處理？
    *   **【內容保真度評估 (Content Fidelity Assessment)】**：是否已對每塊待拆分內容進行「呼叫頻率 × 逐字精度」二維評估？高頻 + 逐字（如 `ask_question` JSON）是否強制保留於 `SKILL.md` 主檔；中頻 + 逐字（如平台特定指令、PowerShell 編碼鐵則）是否在 `references/` 檔頭加上「禁止子代理摘要 + 禁止改寫」的 Verbatim 護欄？
3.  [ ] **防禦深度檢查 (Defense-in-Depth Audit)**：
    *   是否已針對易引發「工具幻覺（Tool-use Hallucination）」的步驟（如字串解析），加上了明確的邊界約束指令？
    *   需要動態求值的系統參數，是否使用了具備強烈「反制字面依從性」的佔位符與警告？
    *   針對多層級設定檔，是否加入了防範「靜默覆寫（Silent Overwrite）」的高層級警告（如 `[!CAUTION]`）？
    *   是否已針對 Windows 上的 `wmic` 移除或棄用問題，實作了 fallback 方案（如 `tasklist`）以保證行程查詢的穩健性？
    *   部署步驟是否採用了「工作區優先（Workspace-First）」的腳本路由規則，避免在開發階段將工作區的最新腳本覆寫為全域的舊版腳本？
4.  [ ] **模型降級相容測試 (Degradation Compatibility Test)**：此 Prompt 策略是否已拉高快速模型（Flash 系列）的安全下限，防止其發生暴走或崩潰？同時，對於深度思考模型（Pro 系列），這些約束是否能確保其 100% 穩定執行而不會引發過度解讀？
5.  [ ] **環境依賴預檢 (Environment Dependency Pre-check)**：是否在流程最前端實作了外部依賴（如 Node.js）的跨平台偵測？並具備互動式的降級決策機制？

> **架構宣言**：在 AI Agent 時代，開發「設定精靈」不再只是撰寫流程碼，而是進行一場**認知邊界的設計**。透過將底層運算與高階邏輯徹底解耦，並在提示詞中實施精密的護欄工程（Guardrails Engineering），我們才能打造出既具備人類般靈活智慧，又擁有機器般絕對穩定性的新世代互動系統。