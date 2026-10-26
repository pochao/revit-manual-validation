# LKBIMHanger 的實際製作配方

本配方從專案原始碼與 2026-10-24 製作記錄提取，並納入後續維護教訓。路徑以已知專案 `C:\01_Products\LKBIMHanger` 為基準；專案搬移時先重新定位，不能繼續使用下列硬編碼。

## 原始資料與入口

| 專案相對路徑 | 用途與限制 |
|---|---|
| `AGENTS.md`、`README.md`、`Directory.Build.props` | 目前規範、版本、支援矩陣；歷史教材檔名不是當前插件版號 |
| `tests/README.md`、`docs/TESTING.md` | NativeHarness、正式 UI 與回歸驗證方式 |
| `tests/LKBIMHanger.NativeHarness/ManualVerificationBuilder.vb` | `Run(UIApplication, phase, Optional caseNumber=0) As String`；另有 `AuditDocument(Document, outputRoot)` |
| `tests/manual-validation/20261024-1.0.3/cases.json` 與 `README.md` | 初版十九案例文字、尺寸、範圍和恢復方式，包含舊名與舊功能，先核對再採用 |
| `tests/Localize-ManualFamilies.vb` | 舊教材中文化實作範例；硬編碼目標、GUID，只能在確認並調整後執行 |
| `docs/family-name-map-zh-TW.json`、`docs/FAMILY_NAMES_ZH_TW.md` | 頂層族群／檔名及舊別名對照 |
| `assets/families/2025/catalog.json` | 本次族群來源、名稱、檔案與雜湊；維護版可能變更公式，不能沿用初版全數位元不變的說法 |
| `tests/Verify-ManualValidation.ps1` | 讀 `native-audit.json` 核對初版固定期望，成功寫 `checks.json`；不是操作 Revit 的腳本 |
| `docs/test-reports/manual-validation-file-20261024.md` | 初版製作、分階段執行、警告與版面實際經驗 |
| `docs/test-reports/family-names-zh-TW-20261024.md` | 1.0.4 中文化、幾何不變與正式載入驗證 |
| `deliverables/目前使用.txt`、主檔旁 `開始使用.txt` | 定位現用主檔與使用者模型維護紀錄；內文混有歷史段落，須交叉核對最新報告與宿主 |
| `src/LKBIMHanger.Addin/Hosting/ToolCatalog.vb`、`docs/feature-source-matrix.md`、`web/src` | 核對目前穩定操作、工具名稱及 UI 入口；不直接把教材 H 編號當 UI 編號 |

已知現用教材：`deliverables/人工驗證_1.0.4_中文族群_20261024/LKBIMHanger_1.0.4_人工驗證_中文族群.rvt`。不要因現在插件升版就另製一份同內容 RVT。

## 開始前必須消除的硬編碼

原建立器固定：

- `Root`、`MainName` 為初版 1.0.3 交付路徑；`Manifest` 為初版案例檔。
- `Source` 為 `tests/artifacts/formal-ui-remaining-20261024/fixture-complete.rvt`；初始化從這個已開啟的來源文件取 Pipe ID `1087693`、Floor ID `1087704`。
- Revit 主版本為 2025、十九案、各案位置與族群判斷、數值期望、專案／圖紙上的 1.0.3 文字，以及 `AuditDocument` 的允許輸出根目錄。
- 原生 schema：管理所有權 `99C0F726-1B82-467E-A9C4-B4608E60A283`；來源追蹤 `04DECF71-49EE-40F3-BF8B-1C406CD32B03`。查現行產品儲存契約後才使用；不擅自替換 GUID。

重建前讀原始碼，確認來源文件和元素仍存在且型別／系統正確；重新推導範圍與尺寸。需要新建時最小幅度參數化或加入專用產生入口，保留目錄、文件身分、禁止覆寫與重複案例檢查。不能只改 `Root` 就重播。

維護現用主檔時不走 `init/case/finish`。先從目前文件重新發現元素並取完整基線，選擇局部修正與本次只讀稽核；修正只能觸及授權範圍，保留使用者編輯及參考身分。原 `audit/AuditDocument` 也有下述初版 sidecar 限制，不能直接當完整維護稽核。

## 建置與載入

原專案為 VB.NET、Revit 2025、.NET 8 Windows、x64，API 參考 `Private=false`；目前支援範圍仍以專案檔為準。只修改產生器時可從專案根目錄建置：

```powershell
dotnet build tests/LKBIMHanger.NativeHarness/LKBIMHanger.NativeHarness.vbproj -c Release -p:DeployToRevit=false --nologo -v:q
```

在唯一批次的 `tests/artifacts/<run-id>/` 準備 Harness、相符 Core 與其必要相依、`Families/2025` 目錄，核對 SHA256；不能混用宿主已載入舊 Core 與新 Harness。參考 `tests/Prepare-NativeRun.ps1` 的 `AssemblyLoadContext`／解析器載入方式，但該腳本會複製 fixture 並呼叫 **Harness.Run**，不是人工教材建立器，不能直接當本流程入口執行。

從有效 API 回呼取得 `uiapp`，由已核對的批次載入組件並呼叫 `LKBIMHanger.NativeHarness.ManualVerificationBuilder.Run`。工具預覽、確認 token／revision、`transactionMode=manual` 等參數使用本次工具實際支援的 schema；不重播舊批次 payload、舊 token 或未知結果的請求。

## 原建立器階段及必要前提

| 順序 | 呼叫／狀態 | 工作與檢查 |
|---|---|---|
| 1 | `Run(uiapp, "init")` | 來源文件須已開啟；建立新公制專案、必要管／樓板／文字類型、標高與十族群，首次 SaveAs 不覆寫，寫 `identity.txt` |
| 2 | 逐一 `Run(uiapp, "case", n)`，n=1…19 | 背景開指定目標，驗 CreationGUID、非唯讀／非工作共用；案例不重複；建立 A／B、必要連結及圖紙，儲存、寫 Hxx.json、Close(False) |
| 3 | `Run(uiapp, "finish")` | 建立 LK-00 索引與 StartingView，檢查初版 19 平面／19 視埠／21 圖紙，寫 verification.json，儲存關閉 |
| 4 | 明確開啟並啟用新主檔，再 `Run(uiapp, "polish")` | 此階段走 ActiveUIDocument 並檢查目標路徑／GUID；放大中文、加十九個 3D 視埠、整理參考表及連結對照、儲存，隨後稽核 |
| 5 | `Run(uiapp, "audit")` 或已核對目標的 `AuditDocument` | 僅在原案例 sidecar 與模型仍一致時使用；模型只讀，輸出本次 native-audit.json，仍會寫外部 JSON |

`finish` 與 `polish` 並非可無條件重入：可能重建索引、重複文字或已存在視圖。不要以「有分階段」推定整個流程冪等。每步先確認目前狀態；`case` 所用背景目標不能同時當作用中文件開啟。

`AuditDocument` 逐讀 H01…H19.json 中舊的 `assemblyIds/sourceIds`，沒有重新發現目前案例集合。原元素已刪除時可能空參考失敗，使用者新增元素則完全不在報告內；成功輸出也不代表完整覆蓋現況。維護須以目前文件的受管紀錄、來源／群組關聯、類別與案例區域重新盤點，記錄本次 ID／UID 基線並明確列出新增、刪除及無法歸案元素。原 sidecar 僅作歷史定位，不用來重建已刪物件；需要涵蓋方向或其他未輸出資料時另加原生幾何讀回。

初版整理後為十九平面、十九 3D、三十八視埠、二十一圖紙（十九案例＋索引＋統計參考）、十個 LK 族群。這些是初版結構期望，並非所有插件的固定要求。

## 初版案例設計及現行差異

| 教材案例 | 穩定操作 | 設計重點 |
|---|---|---|
| H01／H02／H03 | autoHanger／manualHanger／hangerSettings | 自動四站、手動末端、設定另存與重載 |
| H04／H05 | createClamp／clampSettings | 既有共架補兩類管夾，各自套用／恢復；目前 UI 有效欄位須重新核對 |
| H06／H07／H08 | autoChannel／manualChannel／channelSettings | 自動四組雙層、手動末端雙層、共架設定 |
| H09／H10／H11 | deleteHanger／deleteChannel／deleteAll | 個別／整組／全文件刪除；最後一項跨所有 A 區 |
| H12／H13 | updateHanger／updateChannel | H12 來源先改外徑75／中心1100，舊吊架仍60.3／1000；H13 以 M5=125 加寬 |
| H14／H15 | moveHanger／moveChannel | 相對來源方向 +100 mm，保留來源與關聯 |
| H16 | smartView | 參考樣板色彩／透明度，核對實際視圖與樣板恢復 |
| H17／H18 | copyLink／deleteLink | 兩個專用連結、不同牆型別；來源與主模型複本可辨，複製世界位置正確 |
| H19 | schedule | 初版僅選左側12實例，四套單吊架＋四套主共架＝八套；材料從原生參數另算 |

此表是**教材原案例映射**。現行 UI 文件有不同的 Hxx 排序，必須把「圖紙案號 → 操作 ID → 目前 UI 工具名稱」寫清楚。

幾何基準可從初版 README 推導：每側管長6000、外徑60.3、中心1000／1500、三管 Y=0／600／1200、樓板頂3000／底2700 mm；自動站點150／2050／3950／5850，手動6000；右側平移10000，每案以22000×15000的網格排列。上述皆局部座標／特定前置，不是全模型固定世界位置。

後續版本已修過族群方向、上下橫桿等長、管夾附加來源、分層及統計表結構。初版 `ChannelLength2`、統計材料長度及實例數可能不再成立。至少閱讀本次相關報告：

- 方向：`docs/test-reports/all-hanger-case-orientation-20261025.md`、`orientation-lifecycle-20261025.md`。
- 等長與材料公式：`docs/test-reports/equal-channel-length-20261025.md`、`docs/material-parameter-contracts.md`。
- 共架／新管夾關聯：`docs/test-reports/h04-frame-discovery-20261025.md`。
- 原生統計表可開啟：`docs/test-reports/schedule-grid-fix-20261025.md`。只檢查 Header 文字會漏掉 Body 結構問題；要實際雙擊表格並確認重開後可用。

## 中文族群轉換的可重用部分

新建直接用現行中文目錄。只有維護明確指定的舊教材時才參考 `Localize-ManualFamilies.vb`：

1. 驗目標路徑、CreationGUID、專案類型、非唯讀／非工作共用，以及族群來源管理紀錄與名稱衝突。
2. 頂層族群精確舊名→中文名；只有類型名完全等於原族群全名時才改同名類型。保留其他類型、巢狀族群、幾何、公式與參數。
3. 同步本次教材文字、原生表文字與受管記錄中的族群別名，確認 JSON 可解析。
4. 同交易內比較 ID／UID、型別、位置／旋轉、包圍盒、數值參數及釘選快照；差異超出命名則拒絕提交，儲存後再稽核。

原腳本固定舊 GUID、十族群及 1.0.3→1.0.4 文字，不可在現用已中文化主檔再次直接執行。原製作查證 Revit 名稱拒絕 `~`，中文名使用 `-`；應呼叫目標版本 `NamingUtils.IsValidName`，不要憑字串看似合理就改名。

## 檢查與交付邊界

原檢查腳本只有在本次重取的 native-audit.json 與**未改變的初版數值契約**相符時才適用：

```powershell
pwsh -File tests/Verify-ManualValidation.ps1 -ArtifactRoot '本次已核對的交付絕對路徑'
```

先查參數與原始碼再執行。它會寫 checks.json，固定數量／長度不適合直接拿來驗目前已編輯主檔。歷史 850 PASS 只說明當時初版，不是現在模型重新通過。變更契約需依本次需求重算並更新相應檢查，不得把 Actual 改成 Expected。

原製作曾因複製系統類型顯示更名提示而逾時，後續確認 operationSnapshot 已完成才續作；也曾因獨立短管水力計算出現78警告，查本機2025 API XML後只對教材專用系統設 CalculationLevel=None。這些是診斷線索，不能成為重播未知寫入或全面關閉計算的理由。

舊報告包含 ZIP、搬移副本與可攜性測試，`tests/Package-ManualValidation.ps1`、`tests/Verify-LocalizedManualPortability.vb` 仍在專案；依目前 AGENTS.md 偏好，**不自動執行這些流程**。交付只維持現用主檔、必要 Links、使用說明／人工紀錄與本次證據。驗相對路徑和原位置重開即可；不能把這寫成已完成另地搬移驗證。
