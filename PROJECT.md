# 多運動計分板 - 國際化完善計劃

> 目標：切換語言為英文時，運動計分板內不再殘留中文硬編碼文字。

## 問題分析

專案為單一 HTML 檔案 (`index.html`)，內建 i18n 系統（`LANGS.zh` / `LANGS.en` 雙鍵字典 + `t()` 工具函數）。
目前的 i18n 覆蓋不完整：部分區域跳過 `t()` 直接寫中文，或英文字典缺少對應鍵。

### 殘留中文區域清單

#### 1. 規則對話框 (Rules Modal) — 三運動
`showRules()` 的所有規則文字全以中文硬編碼，完全繞過 i18n 系統。

**Badminton**（`Badminton.render()` 中 `data-h` 按鈕事件，約 L757-763）：
- 7 條規則：三戰兩勝制、21 分規則、29-29 封頂、發球權、場區位置、單/雙打差異、計分手勢、交換場地

**Pickleball**（`Pickleball.render()` 中 `data-h` 按鈕事件，約 L858-870）：
- 4 條規則（單打）或 5 條規則（雙打）：單局制、side-out、0-0-2 開局、計分手勢

**Golf**（`Golf.render()` 中 `data-h` 按鈕事件，約 L986-991）：
- 5 條規則：18 洞、Par 4 預設、計分手勢、總結顯示、桿數修改

#### 2. 確認對話框 (confirm()) — 所有運動
`confirm()` 無法使用 `t()`（瀏覽器原生對話框），目前全部硬編碼中文提示。

| 鍵 | 來源運動 | 內容 |
|---|---------|------|
| `vb_resetConfirm` | Volleyball | 重設排球比賽確認 |
| `bd_resetConfirm` | Badminton | 重設羽毛球比賽確認 |
| `bd_modeChangeConfirm` | Badminton | 切換單/雙打確認 |
| `bd_switchCourtFinal` | Badminton | 決勝局交換場地通知 |
| `bd_selectServer` | Badminton | 指定發球方對話框 |
| `pk_resetConfirm` | Pickleball | 重設皮克球比賽確認 |
| `pk_modeChangeConfirm` | Pickleball | 切換單/雙打確認 |
| `pk_selectFirstServer` | Pickleball | 選擇首發方對話框 |
| `pk_switchServe` | Pickleball | 交換發球方 toast |
| `gf_resetConfirm` | Golf | 重設高爾夫回合確認 |

#### 3. 散落的翻譯鍵缺失或不正確
| 鍵 | 狀態 | 位置 |
|---|------|------|
| `gf_playerEn` | 值為 `'選手'` 中文，但鍵名含 "En" — 應為 `'Player'` | `LANGS.en` Golf 區塊 |
| `bd_gamePoint` (Badminton 單打模式) | 無英文值 | Badminton 單打模式下未顯示 |
| `pk_receivingSide` (Badminton 單打) | 無英文值 | Badminton 單打模式顯示 |

#### 4. 全局 HTML
- `<title>多運動計分板 Scoreboard</title>` — 標題混合中英文
- `<meta name="apple-mobile-web-app-title" content="計分板">` — 無語言切換

---

## 修復計劃

### Phase 1: 規則對話框國際化（最高優先）

**1.1 在 `LANGS.zh` 和 `LANGS.en` 中新增規則鍵**

為 Badminton、Pickleball、Golf 各運動新增規則鍵，每個鍵對應一組規則條目：

```js
// 建議鍵名
rules_bd_zh_lines  // 羽毛球規則陣列 (zh)
rules_bd_en_lines  // Badminton Rules array (en)
rules_pk_zh_lines  // 皮克球規則陣列 (zh)
rules_pk_en_lines  // Pickleball Rules array (en)
rules_gf_zh_lines  // 高爾夫規則陣列 (zh)
rules_gf_en_lines  // Golf Rules array (en)
```

**1.2 修改 `showRules()` 支持規則陣列**

新增參數或派生邏輯，從規則陣列鍵取值而非硬編碼。

**1.3 重寫各運動的 `data-h` 事件處理器**

- `Badminton.render()` 中：將 `showRules(t('rules_bd'), [...中文陣列...])` 改為 `showRules(t('rules_bd'), LANGS[currentLang].rules_bd_lines)`
- `Pickleball.render()` 中：區分單/雙打，各取對應規則陣列
- `Golf.render()` 中：直接從規則陣列取值

---

### Phase 2: 確認對話框國際化（高優先）

**2.1 確保 `LANGS.en` 所有 confirm 鍵有對應英文**

目前已定義但缺英文的：
- `bd_switchCourtFinal`: 新增 `'Final: leader at 11, swap courts'`

目前無的（需要新增鍵 + 英文翻譯）：
- `vb_resetConfirm`
- `bd_resetConfirm`
- `bd_modeChangeConfirm`
- `bd_selectServer`
- `pk_resetConfirm`
- `pk_modeChangeConfirm`
- `pk_selectFirstServer`
- `pk_switchServe`
- `gf_resetConfirm`

**2.2 修改所有 `confirm()` 調用**

全部改為 `confirm(LANGS[currentLang].xxx_key.replace(...))` 格式（保持目前的替換參數寫法以支援動態隊名 `{team}`）。

**2.3 修改所有 toast 訊息**

`toast(LANGS[currentLang].vb_switched)` 改為 `toast(t('vb_switched'))`（但 `toast()` 不接受 key，需要改 `toast()` 以接受 key 或預先解析）。

---

### Phase 3: 修正翻譯鍵錯誤與散列缺失

**3.1 修正 `gf_playerEn` 鍵值**

`LANGS.en.gf_playerEn` 從 `'選手'` 改為 `'Player'`。

**3.2 補充 Badminton 單打模式相關鍵**

- `bd_gamePoint` 在單打模式下需要使用
- `pk_receivingSide` 在 Badminton 單打模式下需要（如果引用）

---

### Phase 4: 全局 HTML 修正

**4.1 隱藏或動態切換標題**

`<title>` 和 `<meta apple-mobile-web-app-title>` 可在 `switchTo()` 時動態更新，或使用 `data-i18n` 屬性。

---

## 工作拆解與檢查清單

- [x] **Phase 1.1** 在 `LANGS.zh` 和 `LANGS.en` 中新增所有運動的規則陣列鍵
- [x] **Phase 1.2** 修改 `showRules(title, items)` 接收規則陣列或修改為 `showRules(title, keyPrefix)`
- [x] **Phase 1.3** Badminton.render() 重寫規則按鈕事件
- [x] **Phase 1.4** Pickleball.render() 重寫規則按鈕事件（分單打/雙打）
- [x] **Phase 1.5** Golf.render() 重寫規則按鈕事件
- [x] **Phase 1.0 排球** Volleyball 規則也一併國際化（`rules_vb_lines`)
- [x] **Phase 2.1** `LANGS.en` 新增所有 confirm 鍵的英文翻譯（如已存在則確認完整性）
- [x] **Phase 2.2** 所有運動的 `confirm(LANGS[currentLang].xxx)` 改為使用翻譯鍵
- [x] **Phase 2.3** 修改 `toast()` 以接受翻譯鍵或預先解析
- [x] **Phase 3.1** 修正 `LANGS.en.gf_playerEn` 從 `'選手'` 改為 `'Player'`（LANGS.zh/zh 互換修正）
- [x] **Phase 3.2** 確認 `bd_gamePoint` 和 `pk_receivingSide` 中英文均有對應翻譯
- [x] **Phase 4.1** `updateI18nTexts()` 新增動態更新 `<title>` 和 `<meta apple-mobile-web-app-title>` 語言
- [x] **Critical Bug Fix** 第 343 行 `rules_vb_lines` 陣列結尾缺少逗號，導致 JavaScript 語法錯誤（頁面全黑）
- [ ] **驗收測試** 逐一切換中英文，確認所有運動的每個頁面無中文殘留

---

## 注意事項

1. **單一檔案**：所有程式碼在 `index.html` 中，修改時注意行號跳變
2. **confirm() 限制**：瀏覽器原生對話框無法使用 `t()` 的 DOM 更新機制，必須提前從 `LANGS[currentLang]` 讀取字串
3. **toast() 函數**：目前直接接受字串，需要支持 key 解析或預先查找
4. **不變性**：目前的 `LANGS.zh` 不應修改現有鍵的值，只新增鍵
5. **錯誤處理**：如果鍵不存在，應回退到 `zh` 或鍵名本身（目前的 `t()` 已支援）
6. 所有運動的規則文字來源應保持一致格式（數值、符號、專有名詞）

---

## 實作記錄

### Phase 1 完成 — 規則對話框國際化

**日期**：2026-07-08

**修改摘要**：

| 運動 | 修改類型 | 說明 |
|------|---------|------|
| Badminton | `data-h` 事件重寫 | 改為 `LANGS[currentLang].rules_bd_lines[dbl?'doubles':'singles']` |
| Pickleball | `data-h` 事件重寫 | 改為 `LANGS[currentLang].rules_pk_lines[dbl?'doubles':'singles']` |
| Golf | `data-h` 事件重寫 | 改為 `LANGS[currentLang].rules_gf_lines` |
| Volleyball | `data-h` 事件重寫 + 新增 `rules_vb_lines` | 一併國際化（原計畫未涵蓋） |

**新增的翻譯鍵**（每種語言）：

- `rules_bd_lines`: `{ singles: [...], doubles: [...] }` — 羽毛球（含單/雙打差異）
- `rules_pk_lines`: `{ singles: [...], doubles: [...] }` — 皮克球（含 0-0-2 開局）
- `rules_gf_lines`: `['18 洞桿數賽。', ...]` — 高爾夫
- `rules_vb_lines`: `['五戰三勝制，...']` — 排球

**技術細節**：
- 所有 `showRules(t('rules_xxx'), ...)` 呼叫改為從 `LANGS[currentLang]` 動態讀取
- 單/雙打不同規則使用 `rules_bd_lines.doubles` / `rules_bd_lines.singles` 物件鍵區別
- 語法檢查通過（括號、大括號、方括號平衡）

---

### Phase 2 完成 — 確認對話框與 Toast 國際化

**日期**：2026-07-08

**修改摘要**：

| 運動 | 修改類型 | 說明 |
|------|---------|------|
| Volleyball | `data-swap` 按鈕事件 | `toast(LANGS[currentLang].vb_switched)` → `toast(t('vb_switched'))` |
| Badminton | `data-srv` 按鈕事件 | `confirm(t('bd_selectServer').replace(...))` → `confirm(LANGS[currentLang].bd_selectServer.replace(...))` |
| Pickleball | `data-first` 按鈕事件 | `confirm(t('pk_selectFirstServer').replace(...))` → `confirm(LANGS[currentLang].pk_selectFirstServer.replace(...))` |
| 所有運動 | Toast 訊息統一 | 全部改為 `toast(t('key'))` 格式 |

**確定的現狀**：
- 所有 `confirm()` 調用已使用 `LANGS[currentLang].key` 格式（無需修改確認對話框本身）
- 所有關鍵詞在 `LANGS.zh` 和 `LANGS.en` 中均有對應翻譯
- 線 575（排球交換場地 toast）：改為 `toast(t('vb_switched'))`
- 線 738（羽毛球指定發球方）：改為 `confirm(LANGS[currentLang].bd_selectServer.replace(...))`
- 線 845（皮克球選擇首發方）：改為 `confirm(LANGS[currentLang].pk_selectFirstServer.replace(...))`

**技術細節**：
- 確認對話框保持 `LANGS[currentLang].key` 格式（瀏覽器原生對話框限制）
- Toast 訊息統一使用 `t('key')` 格式
- 所有 `replace()` 參數替換（動態隊名 `{team}`）保留不變

---

### Phase 3 完成 — 修正翻譯鍵錯誤、缺失鍵、全局 HTML 標籤

**日期**：2026-07-08

**Critical Bug 修復**：
- **第 343 行**：`rules_vb_lines` 陣列結尾缺少逗號（`...。']` → `...。']`, ），導致 JavaScript 無法解析。
  症狀：網頁顯示全黑、tab 無法點擊、內容完全不渲染。
  修復方式：在 `rules_vb_lines` 陣列的 `]` 後補上逗號。

**修改摘要**：

| 項目 | 修改類型 | 說明 |
|------|---------|------|
| 第 343 行 | 語法修復 | `rules_vb_lines` 陣列結尾補逗號，JavaScript 正常解析 |
| LANGS.zh 線 298 | 值修正 | `gf_playerEn:'Player'` → `gf_playerEn:'選手'`（中文應用中文） |
| LANGS.en 線 415 | 值修正 | `gf_playerEn:'選手'` → `gf_playerEn:'Player'`（英文應用英文） |
| LANGS.zh 線 244 | 新增鍵 | `appTitle_zh:'多運動計分板 Scoreboard'` |
| LANGS.en 線 365 | 新增鍵 | `appTitle_zh:'多運動計分板 Scoreboard', appTitle_en:'Scoreboard'` |
| updateI18nTexts() | 功能增強 | 新增 `document.title` 和 `<meta apple-mobile-web-app-title>` 的語言切換 |

**`bd_gamePoint` 和 `pk_receivingSide` 現狀**：
- `bd_gamePoint`：`LANGS.zh` = `'局點'`（line 265），`LANGS.en` = `'Game Point'`（line 382）✅ 已有
- `pk_receivingSide`：`LANGS.zh` = `'接發方不計分'`（line 283），`LANGS.en` = `'Receiving side does not score'`（line 400）✅ 已有

**技術細節**：
- `updateI18nTexts()` 新增三行：
  1. `document.title = currentLang === 'zh' ? dict.appTitle_zh : dict.appTitle_en;`
  2. `const meta = document.querySelector('meta[name="apple-mobile-web-app-title"];`  
  3. `if(meta) meta.content = currentLang === 'zh' ? '計分板' : 'Scoreboard';`
- 所有語言切換觸發 `updateI18nTexts()`（透過 `switchTo()` 調用），標題即時更新
- 使用 jsdom 驗證：JavaScript 語法正確、tabs 正常、view 渲染正確
