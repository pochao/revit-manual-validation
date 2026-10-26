# Revit 人工驗證 RVT 製作技能

將 Revit 插件的功能整理成可手動操作、比較與復原的 RVT 教材。流程源自 `LKBIMHanger_1.0.4_人工驗證_中文族群.rvt`，也適用於其他 Revit 插件。

包含：

- A 試作區與 B 完成對照區的模型設計。
- 繁體中文步驟、預期結果、平面／3D 視埠與索引圖紙。
- 原生尺寸、幾何方向、管理關聯、連結與統計核對。
- 正式 UI、原生 Undo 與待使用者人工驗收的證據區分。
- 保留使用者既有編輯、維護單一現用主檔的流程。

## 使用

將儲存庫安裝到 Codex 的個人 skills 目錄；例如 Windows：

```powershell
git clone https://github.com/pochao/revit-manual-validation.git "$env:USERPROFILE\.codex\skills\revit-manual-validation"
```

若已存在同名技能，先比較內容，保留現有修改。

在對話中使用：

```text
使用 $revit-manual-validation，為這個 Revit 插件製作人工驗證 RVT，
包含可操作試作區、完成對照、中文步驟圖紙與人工驗證紀錄。
```

或更新既有教材：

```text
使用 $revit-manual-validation，依目前插件版本更新現用人工驗證 RVT，
保留我的模型編輯，重新核對本次案例與預期結果。
```

## 檔案

| 檔案 | 內容 |
|---|---|
| [SKILL.md](SKILL.md) | 技能入口與通用製作／維護流程 |
| [案例契約](references/case-contract.md) | 案例 manifest 範例、獨立數值期望與驗收證據 |
| [LKBIMHanger 配方](references/lkbimhanger-recipe.md) | 原產生器階段、硬編碼前提、中文族群轉換及維護限制 |
| [介面設定](agents/openai.yaml) | 技能顯示名稱與啟動提示 |

## 執行條件與範圍

這是供 Agent 使用的流程技能，實際建立／重開 RVT 需要受支援的 Revit、目標插件及有效 Revit API 入口。LKBIM 環境可搭配 `lkbim-revit` 技能與 Bridge；其他環境使用專案既有的受支援入口。

LKBIMHanger 配方中的專案路徑、族群與產生器是原環境參考，使用前需重新定位及核對；其他插件使用通用流程。儲存庫包含技能文件與案例範例，產品原始碼、族群檔及 RVT 模型由對應專案提供。

目前交付偏好為單一現用主 RVT 與必要 Links，ZIP 與另地搬移驗證僅在另有要求時製作。教材建立成功和原生檢查通過，須與使用者實際操作及 Undo 驗收分別記錄。
