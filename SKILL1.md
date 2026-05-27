
# 🏋️ 體能測驗計算器 Skill

## 一、工作流程總覽

這個 Skill 代表我個人設計「標準比對型互動工具」的完整工作流程，分為六個階段：

```
1. 目標設定 → 2. 資料蒐集 → 3. 功能拆解 → 4. 畫面設計 → 5. 程式實作 → 6. GitHub 發布
```

---

## 二、目標設定（Goal Setting）

在開始前，先回答三個問題：

| 問題 | 本次答案 |
|------|---------|
| 這個工具解決什麼問題？ | 役男不知道自己體能是否達到國軍鑑測標準 |
| 目標使用者是誰？ | 役男、現役軍人、體能訓練愛好者 |
| 成功的定義是什麼？ | 使用者輸入成績後，能立即知道合格與否，並獲得 AI 訓練建議 |

---

## 三、資料蒐集（Data Collection）

### 標準資料庫格式
所有標準數值以 JavaScript 物件儲存，索引對應 [青男, 青女, 壯男, 壯女, 中男, 中女]：

```javascript
const STD = {
  upper: {
    pushup:     { vals:[40,21,30,15,20,8],  unit:'下', high:true },
    pullup:     { vals:[5,20,3,14,2,8],     unit:'下/秒', high:true },
    kettlebell: { vals:[40,30,30,20,20,10], unit:'下', high:true },
  },
  core: {
    situp:  { vals:[42,31,35,23,20,16], unit:'下', high:true },
    plank:  { vals:[85,85,70,70,70,70], unit:'秒', high:true },
    crunch: { vals:[31,27,24,21,20,17], unit:'下', high:true },
  },
  cardio: {
    run3k:   { vals:[885,1055,990,1140,1080,1260],   unit:'秒', high:false },
    walk5k:  { vals:[2420,2660,2500,2750,2700,2940], unit:'秒', high:false },
    swim800: { vals:[1530,1710,1620,1800,1710,1890], unit:'秒', high:false },
    shuttle: { vals:[50,40,35,25,25,15], unit:'趟', high:true },
    rope:    { vals:[530,430,499,399,462,362], unit:'下', high:true },
  }
};
```

**資料來源**：依據網路上國軍基本體能鑑測合格標準資料整理，僅供參考。

---

## 四、功能拆解（Feature Breakdown）

### Input / Process / Output 架構

| 階段 | 內容 |
|------|------|
| **Input** | 性別、年齡層、各項體能成績（下數或分:秒）、可選 AI 分析 |
| **Process** | 標準比對 → 差距計算 → 合格判定 → Chart.js 雷達圖資料生成 → Claude API 呼叫 |
| **Output** | 合格/不合格標籤、差距數值、雷達圖、AI 個人化訓練建議 |

### 分頁規劃
- Tab 1：📊 測驗試算（主功能）
- Tab 2：⚖️ BMI 計算（輔助功能）
- Tab 3：📋 標準一覽（資料查詢）

---

## 五、核心計算邏輯（Core Logic）

### 標準索引計算
```javascript
function getStdIdx() {
  const gIdx = gender === 'male' ? 0 : 1;
  const aIdx = ageGroup === 'young' ? 0 : ageGroup === 'mid' ? 1 : 2;
  return aIdx * 2 + gIdx; // 0=青男,1=青女,2=壯男,3=壯女,4=中男,5=中女
}
```

### 合格判定
- **高分型**（伏地挺身、仰臥起坐等）：`成績 >= 標準` 為合格
- **低分型**（跑步、健走計時）：`成績(秒) <= 標準(秒)` 為合格
- **計時輸入**：拆分分鐘欄 + 秒鐘欄，內部統一轉換為秒數比較

### AI 分析（Claude API）
```javascript
// 將使用者成績整理為結構化 prompt，呼叫 Claude API
const prompt = `
你是一位專業體能訓練教練。
使用者資料：${gender==='male'?'男':'女'}性，${ageLabel}（${ageRange}歲）
測驗結果：${resultsText}
請提供：1.整體評估 2.最需加強項目 3.具體訓練計畫（週訓練建議）
以繁體中文回覆，語氣像教練鼓勵學員。
`;
```

---

## 六、UI 設計規範（Design Spec）

### 色彩系統
```css
--army-dark:  #1b3a1f;  /* 深軍綠，用於 Header、按鈕 */
--army-mid:   #2d5a35;  /* 中軍綠，用於標題 */
--khaki:      #c8a44a;  /* 卡其金，用於強調色 */
--bg:         #f0ede6;  /* 米白，頁面背景 */
--pass:       #22863a;  /* 合格綠 */
--fail:       #b52b1e;  /* 未達標紅 */
```

### 字型
- 主字型：`Noto Sans TC`（Google Fonts）
- 數字字型：`Share Tech Mono`（Google Fonts）

### 版面
- 桌機：雙欄（左輸入、右結果）
- 手機：單欄，RWD 自動調整
- 最小支援寬度：320px

---

## 七、GitHub 發布流程（Deployment Checklist）

```
□ 1. 建立 Public Repository（命名建議：fitness-calculator）
□ 2. 上傳 index.html 至根目錄
□ 3. Settings → Pages → Branch: main / Folder: / (root) → Save
□ 4. 等待 1~2 分鐘，取得網址：https://<帳號>.github.io/<repo>/
□ 5. 測試：手機瀏覽器開啟確認 RWD 正常
□ 6. 同資料夾上傳本 SKILL.md 作為開發紀錄
```

---

## 八、品質檢查清單（QA Checklist）

```
□ 男/女切換後，標準數值正確更新
□ 三種年齡層切換後，標準數值正確更新
□ 計時型項目（分:秒）計算正確
□ 雷達圖在重新判定後正確刷新，無殘影
□ AI 分析按鈕：loading 狀態顯示、結果正確呈現
□ 手機版不破版（320px 以上）
□ 無 JavaScript console 錯誤
```

---

## 九、可延伸應用（Reusability）

本 Skill 的架構適用於任何「標準比對型工具」，替換資料庫即可快速產出新工具：

| 替換資料 | 可延伸的工具 |
|---------|------------|
| 消防員體能標準 | 消防特考體能計算器 |
| 警察特考標準 | 警察特考體能計算器 |
| 學生體適能常模 | 學生體適能自評工具 |
| 各項運動訓練指標 | 個人運動達標追蹤器 |
