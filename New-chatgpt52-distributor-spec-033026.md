WOW Medical Device Distribution Intelligence Studio (HF Spaces / Streamlit) — Updated Technical Specification (v2)
本規格書為「WOW 配送/流向分析工作室」的更新版技術規格：在保留所有既有功能（WOW UI、主題/語言/畫家風格、狀態指標、API Key 管理、資料匯入與標準化、互動儀表板、A/B 比較、Agentic AI pipeline、AI Note Keeper、多供應商模型路由、agents.yaml + SKILL.md 管理、Hugging Face Spaces 部署、安全與效能策略）基礎上，新增「醫療器材配送資料集工作台」能力：支援上傳/貼上資料集、多格式預覽、2000–3000 字的資料集總結報告、由 Agent 產生的 5 個 WOW 資訊圖（以 Python/Plotly 生成並在頁面互動呈現）、進階篩選條件、以及更完整的 agents.yaml 設定檔上傳/貼上/下載與自動標準化轉換。

1. 產品定位與目標
1.1 產品定位
WOW Studio 是部署於 Hugging Face Spaces 的 Streamlit 單頁 Web 應用，提供「醫療器材配送/流向資料」的端到端工作流程：

資料集匯入（預設/貼上/上傳；txt/csv/json）
欄位推斷 + 標準化（canonical schema）
資料預覽與品質稽核提示
互動式探索儀表板（WOW 視覺化 + 篩選 + 下鑽 + 下載）
AI 代理（agents.yaml）逐步執行，支援每一步 prompt / model / max_tokens 調整與輸出編修再傳遞
AI Note Keeper：貼上筆記→整理成 markdown、關鍵字上色、AI 魔法工具
醫療器材配送資料集「綜整報告」（2000–3000 字 Markdown）
Agent 產生 5 個 WOW 資訊圖（Python/Plotly），含可點擊節點的配送網路
1.2 核心使用者
合規/稽核：追溯性、批號/序號完整性、許可證一致性、流向異常
營運/物流分析：供應商→許可證→型號→客戶路徑、集中度、趨勢
資料團隊：欄位 mapping、非標準資料清理規則、agents.yaml 維運
管理層：AI 報告與高層摘要、可視化決策儀表板
2. 部署與執行環境（保持 + 強化）
2.1 平台與框架
Hugging Face Spaces（Streamlit Space）
Python + Streamlit（單一 app，模組化邏輯）
視覺化：Plotly（含 graph_objects / express）；必要時採用 Plotly 事件擴充以支援點擊節點互動
LLM Providers：
OpenAI API
Gemini API
Anthropic API
Grok API (xAI)
2.2 Repository 檔案與配置（既有保留）
app.py（主應用；本規格不提供程式碼）
agents.yaml（可由 UI 管理、上傳/貼上/下載、標準化）
SKILL.md（作為 agents 的全域系統提示詞）
defaultdataset.json（預設資料集；建議存 records 或 csv-string 封裝）
requirements.txt（依賴清單；互動點擊功能可列 optional）
3. WOW UI 2.0（既有保留 + 統一覆蓋新模組）
3.1 WOW UI 視覺系統（保持）
Light / Dark Theme 切換
English / 繁體中文（zh-TW）切換
20 種畫家風格（例如：Monet、Van Gogh、Hokusai…）
Jackpot：一鍵隨機挑選畫家風格
視覺語言：
Glassmorphism 卡片（半透明、模糊、陰影）
漸層背景（radial / linear gradients）
強調色（accent）映射到圖表 palette、按鈕、狀態 chip
3.2 WOW 狀態指標（保持 + 擴充）
頂部狀態列（sticky header）顯示：

Provider keys 狀態：OpenAI / Gemini / Anthropic / Grok
env（環境變數提供）
session（使用者在頁面輸入）
missing（不可用）
資料狀態：
目前主資料集筆數（rows）、欄位數（cols）
是否完成標準化（canonical ready）
資料品質警示計數（例如日期解析失敗欄位數、缺值比；以「提示」不造假）
視覺化狀態：
互動點擊功能（events）是否啟用
最新一次 infograph 生成時間戳與版本號（避免混淆）
3.3 API Key 輸入規則（保持）
若 key 由環境變數取得：不顯示輸入框（避免洩露、避免誤貼）
若環境無 key：顯示 password 欄位供使用者輸入，僅存於 session state
UI 永不回顯明文 key；僅顯示 **** 或狀態標籤
4. 資料集工作台（Dataset Studio）— 新增醫療器材資料集能力
目標：讓使用者上傳或貼上「醫療器材配送/經銷」資料集（txt/csv/json），快速預覽、標準化、視覺化、產生 AI 報告與資訊圖，並可持續用 Agent 對資料提問。

4.1 支援的匯入形式（新增）
Upload：檔案上傳（.txt, .csv, .json）
Paste：貼上原始文本（可為 CSV、JSON、TSV、或固定寬度文本；系統以 heuristics 判別）
Default dataset：一鍵載入預設資料集（保留）
4.2 資料集預覽（新增）
預覽分為兩區：
Raw Preview：原始解析結果（未標準化）
Standardized Preview：canonical schema 結果
預覽筆數：
預設顯示 20 records
使用者可選擇顯示 N 筆（例如 20/50/100/200；上限可配置以控效能）
預覽表格需支援：
欄位排序、搜尋（簡易 contains）
顯示欄位型別推斷（string/date/int）
匯入解析警告（例如 JSON 結構不是 list、CSV 分隔符不一致）
4.3 Canonical Schema（保持 + 擴充欄位以支援新篩選）
既有 canonical 欄位（保留）：

Canonical field	說明	類型目標
supplier_id	供應商代碼	string
deliver_date	出貨/交貨日期	datetime
customer_id	客戶代碼	string
license_no	許可證字號	string
category	分類/品項類別（原始描述）	string
udi_di	UDI/DI	string
device_name	品名/裝置名稱	string
lot_no	批號	string
serial_no	序號	string
model	型號	string
quantity	數量	int
為支援新增需求中的「classification No / date zone」與更高品質的分析，新增派生欄位（不破壞既有欄位）：

Derived field	來源/規則	用途
classification_no	從 category 擷取（例如 E.3610）；若 category 不含代碼則為空	篩選、分群、時間關聯圖
date_zone	由 deliver_date 依使用者選擇生成（Day/Week/Month/Quarter/Year）	時間序列彙總與互動
device_id_hint	優先 serial_no，其次 lot_no，再其次 udi_di+model	設備唯一數估計（需標註為估計）
path_key	supplier_id>license_no>model>customer_id 組合	network / sankey 聚合
注意：classification_no 可能與法規分類不同；此欄位在報告中需標註「依資料欄位推斷之分類代碼」。

4.4 欄位對映與非標準資料支援（保持 + 強化）
允許來源欄位同義詞（中英、大小寫、符號差異）
針對常見欄位別名：
SupplierID / Supplier / vendor_id → supplier_id
Deliverdate / deliver_date / ship_date → deliver_date
LicenseNo / license / permit → license_no
UDID / UDI / DI → udi_di
LotNO / LOT / batch → lot_no
SerNo / SN / serial → serial_no
Number / qty / quantity → quantity
日期解析：
支援 YYYYMMDD、YYYY-MM-DD 等
解析失敗不得硬造日期；需標記為 NaT 並在品質提示顯示
數量解析：
字串數字、逗號分隔
解析失敗轉為空/0 的策略需可配置並在品質提示揭露（避免誤導）
5. 互動儀表板（保持 + 對齊醫療器材資料集篩選）
5.1 全域篩選器（保持 + 新增條件）
既有篩選（保留）：

supplier_id（多選）
license_no（多選）
model（多選）
customer_id（多選）
date range（起迄）
keyword search（跨多欄位 contains）
新增篩選條件（依需求）：

classification_no（多選；若資料無此欄則顯示「未提供/無法推斷」）
lot_no / serial_no（可切換欄位後查詢或多選；避免選項爆量，建議提供「contains 搜尋 + TopN 列表」）
date_zone（選擇 Day/Week/Month/Quarter/Year，影響時間彙總與動態資訊圖）
進階：可加入「排除空值」勾選（例如排除無 license_no）
5.2 KPI 卡與摘要（保持）
Rows、Total Quantity
Unique suppliers / customers / models / licenses
日期範圍（min/max）
分類代碼數（unique classification_no，若有）
6. 5 個 WOW Infograph（新增，Agent 產生 Python/Plotly 規格並由系統渲染）
需求重點：Agent 會生成 5 個 WOW 資訊圖（以 Python 代碼形式或等價的繪圖規格），其中必須包含「配送網路（SupplierID > license number > model > customerID）」且可點擊節點顯示詳細資訊。
本規格不提供程式碼，但定義：資訊圖必須遵循的資料彙總、互動、篩選、降載、與節點明細行為。

6.1 共通規格（所有 infograph 適用）
資料來源：標準化後 df（可被全域篩選器與 infograph 專屬篩選器共同作用）
必須支援 filters（需求指定）：
classification_no
supplier_id
license_no
model
lot_no / serial_no
customer_id
date_zone（影響彙總粒度）
降載策略（避免節點/邊爆量）：
TopN nodes per layer（預設 30，可調）
min_flow_threshold（預設依總量的某比例或固定值）
長尾合併為「Other」
Hover/Tooltip 規範：
顯示 node 類型（Supplier/License/Model/Customer）
顯示 quantity（sum）
顯示 unique 連結數（度數）
顯示佔比（相對於當前篩選集合）
色彩策略：
依畫家風格 accent 設定主色
類別顏色需是合法色碼（避免 Plotly validator 問題）
匯出能力：
圖表可下載為 PNG（由 Plotly 模式列）
圖表支援「生成圖表描述文字」供報告引用（可由 Agent 撰寫）
6.2 Infograph #1：Interactive Distribution Network Graph（必備）
目的：以可點擊節點的網路/分層圖呈現配送路徑：
SupplierID > LicenseNo > Model > CustomerID

資料彙總

Nodes：四層（Supplier, License, Model, Customer）
Links：相鄰層 pair 聚合
Supplier→License：groupby(supplier_id, license_no) sum(quantity)
License→Model：groupby(license_no, model) sum(quantity)
Model→Customer：groupby(model, customer_id) sum(quantity)
Node id 命名需避免重名：加前綴 S:, L:, M:, C:
互動

點擊節點：右側/下方顯示「節點詳細面板」
節點層級與原始值
篩選後該節點的：
Top 10 相鄰節點（含 quantity）
對應的原始紀錄預覽（預設 20 筆，可調）
「一鍵套用為全域篩選器」：將該 node 值加入相對應 filter（例如點 Supplier 即篩 supplier_id）
點擊邊（可選強化）：顯示該 link 的聚合規則與明細（同樣只預覽 N 筆）
視覺布局

建議採分層布局（四列或四欄），使 flow 清晰
邊粗細代表 quantity；可切換「顯示數量」與「顯示佔比」
6.3 Infograph #2：Sankey Flow (Supplier→License→Model→Customer)
目的：快速看主要流向與集中度，適合高層簡報。

規格

維度 path 固定四層
value：sum(quantity)
TopN 與 Other 合併必須存在（Sankey link 過多會不可讀）
Hover：顯示來源、去向、quantity、佔比
6.4 Infograph #3：Hierarchical Share Sunburst / Treemap（層級占比）
目的：從全局看供應商/許可證/型號/客戶的占比結構。

規格

Path：Supplier → License → Model → Customer（可配置 maxdepth，例如預設 3）
Values：sum(quantity) 或 unique device count（若可估計，須標註估計）
Color：
以 Supplier 或 classification_no 映射色彩（需保持一致）
6.5 Infograph #4：Heatmap Matrix（Supplier×Model / Customer×Model）
目的：以矩陣呈現供應商-型號或客戶-型號的強度分布，利於找關聯與異常集中。

規格

模式切換：
Supplier×Model
Customer×Model
Values：
raw sum(quantity)
share（列/欄正規化）
z-score（用於異常；需提示統計假設）
TopN：依總量排序，避免矩陣過大
Tooltip：row、col、value、rank
6.6 Infograph #5：Temporal Dynamics（趨勢 + 結構變化）
目的：呈現配送活動隨時間的變化，可輔助稽核（尖峰/異常週期）與營運規劃。

規格

X：時間（由 date_zone 決定日/週/月/季/年）
Y：sum(quantity)、unique customers、unique models（可切換）
支援多條線 overlay 或 small multiples
支援滑鼠框選時間段並同步更新其他圖（若 Streamlit rerun 造成限制，至少提供「套用選取時間為 filter」）
7. 動態資訊圖（新增）：Classification No × Date Zone 關聯演化圖
需求：使用者選擇 classification No 與 date zone 後，圖表顯示「經銷商數、器材數、客戶數」隨時間的互動關係。

7.1 圖表目的
觀察特定分類代碼的市場活動變化
同時追蹤三個核心量：
distributors（可定義為 supplier_id 數或中間經銷節點；需在報告中定義）
devices（可定義為 unique model 或 device_id_hint；需可切換）
customers（unique customer_id）
7.2 互動規格
使用者選擇：
classification_no（單選或多選）
date_zone（Day/Week/Month/Quarter/Year）
圖表呈現：
時間序列折線（3 條）或雙軸/三軸（建議使用同軸但標準化，或提供切換）
Hover 顯示當期三個指標值、相對前期變化（若可計算）
Drilldown（可選）：
點擊某時間點：顯示該期 Top suppliers / Top models / Top customers
7.3 指標定義（避免歧義）
distributors：
預設 = unique supplier_id
可選 = unique license_no（若代表經銷許可證節點）
devices：
預設 = unique model
可選 = unique device_id_hint（需標註為估計，視 serial/lot 完整性）
customers：
unique customer_id
8. AI 資料集總結報告（新增，2000–3000 字 Markdown）
8.1 觸發與輸入
在 Dataset Studio 中提供「Generate Dataset Summary Report」：

使用者可選 provider/model（沿用可選模型清單）
可調 max_tokens（預設 12000；但輸出需控制在 2000–3000 字）
輸入給 Agent 的內容採「最小必要」：
canonical 欄位清單與 mapping report
資料列數、日期範圍、欄位缺失概況（由系統計算或提示計算方式）
TopN 彙總表（供應商/客戶/型號/許可證/分類）
路徑彙總（Top paths by quantity）
資料品質檢核摘要（缺值、日期解析失敗、疑似重複 key 策略）
原則：不得捏造未計算出的統計數字；若系統未計算則 Agent 必須改以「方法」描述。

8.2 報告結構（必須遵循）
報告需包含（建議章節）：

Executive Summary（高層摘要）
Dataset Overview（資料來源、筆數、欄位、時間範圍）
Schema & Standardization（欄位對映、推斷、缺口）
Distribution Network Insights（供應商→許可證→型號→客戶的集中度與主要路徑）
Temporal Patterns（依 date_zone 的趨勢、季節性/尖峰）
Product & Classification Insights（分類代碼、型號、品名）
Data Quality & Risk Notes（缺值、序號/批號完整性、重複與可追溯性風險）
Recommended Next Analyses（下一步分析問題清單、建議 KPI、建議視覺化）
8.3 與儀表板/infograph 的串接
報告需引用已生成的 infograph（以「圖 1～圖 6」方式標號）
報告中的「可操作建議」可提供一鍵按鈕：
「Apply recommended filters」：將建議條件套用到全域 filter（僅規格要求；實作可後續）
9. Agent Studio（保持 + 強化資料集導向）
9.1 模型與參數可調（保持）
在執行每個 agent 前，使用者可調整：

prompt（user prompt）
max_tokens（預設 12000）
model 選擇清單（保持並延伸相容）：
gpt-4o-mini, gpt-4.1-mini
gemini-2.5-flash, gemini-2.5-flash-lite, gemini-3-flash-preview
Anthropic models（以配置呈現，如 claude-3-5-sonnet-latest）
grok-4-fast-reasoning, grok-3-mini
9.2 逐步執行 + 輸出可編修傳遞（保持）
每個 agent 執行後，輸出以：
Text view / Markdown view 切換
可編輯 edited_output
使用者可指定「下一個 agent 的 input」來源：
原始輸出
edited_output（手動修訂後）
或「資料集摘要 + edited_output」混合（避免脈絡遺失）
9.3 新增：Dataset Prompting Workspace（新增）
除 agents.yaml 之外，提供「對資料集持續提問」工作區：

類似 chat / iterative prompting
每次提問可指定：
使用哪個模型
是否附帶目前 filters 條件（使回答對齊當前視圖）
是否附帶 TopN 聚合表與 infograph caption（提升回答品質）
回答可一鍵存成 AI Note Keeper 的條目（串接）
10. agents.yaml Setup Feature（新增強化：上傳/貼上/下載 + 非標準轉標準）
需求：使用者可 upload/paste/download agents.yaml；若不標準，系統需轉成標準化 agents.yaml。

10.1 標準 agents.yaml Schema（規範）
系統認定的標準結構：

最外層 key：agents
agents 為 list，每個元素必須有：
id（唯一、snake_case）
name
description
provider（openai/gemini/anthropic/grok）
model
temperature
max_tokens
system_prompt（多行字串）
user_prompt（多行字串）
10.2 上傳/貼上/下載（保持 + 強化）
Upload：接受 .yaml / .yml
Paste：文字框
Download：輸出目前 session 的 agents.yaml
Reset：回到預設（題目給的 Default agents.yaml 必須作為一鍵復原來源）
10.3 非標準轉標準（新增：兩階段）
Stage A：規則式標準化（不依賴 LLM）

目的：快速處理常見變體，例如：
agent: 單物件 → 包成 agents: [ ... ]
欄位名稱不同：prompt → user_prompt，system → system_prompt
缺少 provider/model → 填預設（如 openai + gpt-4o-mini），並標註 needs_review
max_tokens 缺失 → 用預設 2500 或沿用 UI 預設 12000（需定義優先級）
產出：
standardized YAML
validation report（列出修補/推斷與待人工確認項）
Stage B：LLM Assisted Standardization（可選）

若 YAML 結構混亂、巢狀過深或解析失敗：
使用者選 provider/model
系統送出「請轉成標準 agents.yaml」任務（LLM 只輸出 YAML）
需加上安全防護：
輸出必須可被 YAML parser 解析
若 LLM 輸出含非 YAML 文字，需自動剝離或要求重試（規格層面）
11. AI Note Keeper（保持 + 與資料集串接增強）
11.1 既有功能保留
貼上 note（txt/markdown）
系統轉成組織化 markdown
關鍵字以 coral color 上色
可在 markdown / text view 修改
可持續 prompt 在該 note 上（模型可選）
6 個 AI Magics（含 AI Keywords：自選 keywords 與顏色）
11.2 新增：資料集筆記一鍵保存（新增）
Dataset Summary Report 可一鍵「Save to Note Keeper」
Infograph captions / 圖表洞察可一鍵加入 note
Dataset Prompting Workspace 的 Q/A 可選取段落保存
12. 安全、隱私與合規（保持 + 醫療資料情境強化）
API Key：
env 取得不顯示輸入框
session 輸入不回顯、不中途寫檔
資料最小揭露：
預覽預設 20 筆
送 LLM 的資料以摘要與聚合表為主
敏感欄位建議（規格建議，保留未來擴充）：
customer_id、serial_no、lot_no 可能敏感
提供遮蔽策略（hash/partial mask）作為可選功能
不得生成醫療建議（產品定位為物流/流向分析工具；AI 回答需提示非醫療診斷用途）
13. 效能與可靠性（保持 + 新增 infograph/報告壓力控管）
大資料量策略：
所有 network/sankey/heatmap 預設 TopN / threshold
日期彙總先 groupby 再繪圖
Cache 建議：
解析與標準化可 cache（以檔案 hash + mapping 設定為 key）
infograph 的聚合結果可 cache（以 filters + date_zone 為 key）
錯誤處理：
Plotly 色彩必須是合法色碼
JSON 序列化需將 datetime 輸出為 ISO
若資料缺欄（例如缺 license_no），infograph 需 degrade gracefully（顯示提示並用替代 path 或將缺失歸類到 “Unknown”）
14. 驗收（UAT）更新範圍
除既有驗收項目外，新增必測：

上傳 txt/csv/json 能解析並顯示 raw preview
預覽筆數預設 20，可調整 N 並不造成崩潰
classification_no 能從 category 推斷（有資料時）並可篩選
Agent 生成 2000–3000 字 Markdown 報告，結構符合規範且不捏造統計
5 個 infograph 可渲染；Distribution Network 可點擊節點顯示明細
動態資訊圖可依 classification_no + date_zone 變化，呈現 distributors/devices/customers 隨時間變化
Dataset Prompting Workspace 可在目前 filters 下提問並產出可保存內容
agents.yaml 上傳後，若非標準能成功標準化並提供修補報告；可下載標準版本
15.（附錄）預設資料欄位對齊（題目提供資料集）
題目提供的 default datasets 欄位：

SupplierID, Deliverdate, CustomerID, LicenseNo, Category, UDID, DeviceNAME, LotNO, SerNo, Model, Number

建議 canonical mapping：

SupplierID → supplier_id
Deliverdate → deliver_date
CustomerID → customer_id
LicenseNo → license_no
Category → category（並派生 classification_no）
UDID → udi_di
DeviceNAME → device_name
LotNO → lot_no
SerNo → serial_no
Model → model
Number → quantity
20 個後續釐清問題（Comprehensive Follow-up Questions）
你希望 distributors（經銷商數） 的定義是「supplier_id 唯一數」還是「license_no 唯一數」或兩者都要並可切換？
classification_no 應該從 Category 的哪種格式抽取？若 Category 不是 E.3610... 形式，你希望 fallback 規則是什麼（例如整段當分類、或空值）？
你希望 date_zone 支援哪些粒度：Day/Week/Month/Quarter/Year 全部必須，還是以 Day/Month 為主？Week 的週起始日要用週一或週日？
2000–3000 字的資料集總結報告，你希望輸出語言跟 UI 語言同步（EN/zh-TW），還是固定繁中？
報告中是否允許系統先計算基礎統計（例如 TopN、缺值比例），讓 Agent 引用「已計算數字」？還是你希望 Agent 一律只描述方法不出數字？
5 個 infograph 你希望固定是哪 5 個，還是讓使用者在「候選清單」中勾選 5 個再生成？
Distribution Network Graph 你偏好「分層（layered）」還是「力導向（force-directed）」佈局？是否需要在兩者間切換？
點擊節點顯示明細時，你希望顯示哪些欄位（例如一定要顯示 lot_no/serial_no/udi_di）？是否需要遮蔽規則？
network 的邊過多時，TopN 策略你希望以「節點總量」還是「邊流量」為準？min_flow_threshold 希望用固定值還是百分位數？
lot_no 與 serial_no 的篩選，你希望是「多選下拉」還是「文字搜尋 + 建議列表」？（多選可能爆量）
devices（器材數）在動態資訊圖中，你希望預設用 unique model，還是嘗試用 serial_no/lot_no 推斷「unique device」？若推斷，是否接受標註為估計值？
你是否需要支援「同一資料集多檔案合併」情境（例如多個月份 CSV 合併成一個分析視圖）？合併規則是 append 還是依 key 去重？
你希望資料品質稽核（data_quality_audit）在匯入後自動跑一次並給摘要，還是必須由使用者手動觸發？
Dataset Prompting Workspace 你希望是「聊天式多輪」還是「每次提問產出獨立報告段落」的工作流？需要保留上下文到下一輪嗎？
模型選擇清單中「anthropic models」你希望 UI 呈現固定選項（例如 sonnet/opus/haiku），還是允許使用者自行輸入 model name？
agents.yaml 的「非標準轉標準」：你希望上傳後自動轉換並覆蓋，還是先顯示 diff 讓使用者確認後才套用？
SKILL.md（全域 system prompt）是否需要為「醫療器材/合規」提供一份預設模板（例如禁止醫療診斷、強調不捏造）並允許管理者鎖定不可更改？
你希望資訊圖的視覺風格（配色、字體、背景）要跟 20 畫家風格完全一致（包含 Plotly theme 級別），還是僅用 accent 色點綴即可？
你是否需要「一鍵輸出完整報告包」：包含 2000–3000 字報告 + 5 張圖（PNG）+ 使用的 filters + agents 執行紀錄，打包成 zip 下載？
醫療器材配送資料常會有「退貨/更正」或負數數量情境；你希望 quantity 允許負數嗎？若出現負數或 0，圖表與報告應如何處理（排除、獨立分類、或顯示警示）？
