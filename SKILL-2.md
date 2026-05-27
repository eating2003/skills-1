# 🏋️ 體能測驗計算器 Skill

## 一、工作流程總覽

這個 Skill 代表「輸入參數 → 標準比對 → 視覺化分析結論」互動網頁工具的完整工作流程，分為六個階段：

```
1. 目標設定 → 2. 標準資料建立 → 3. 功能拆解 → 4. 畫面設計 → 5. 程式實作 → 6. GitHub 發布
```

---

## 二、目標設定（Goal Setting）

開始前先回答三個問題：

| 問題 | 本次答案 |
|------|---------|
| 這個工具解決什麼問題？ | 役男不知道自己體能是否達到國軍鑑測標準 |
| 目標使用者是誰？ | 役男、現役軍人、體能訓練愛好者 |
| 成功的定義是什麼？ | 輸入成績後，立即看到合格判定、達標率進度條、分析結論與訓練建議 |

---

## 三、標準資料庫格式（Standards Database）

所有標準數值以 JavaScript 物件儲存，索引對應 [青男, 青女, 壯男, 壯女, 中男, 中女]：

```javascript
const STD = {
  upper: {
    pushup:     { vals:[40,21,30,15,20,8],  unit:'下', high:true, name:'伏地挺身' },
    pullup:     { vals:[5,20,3,14,2,8],     unit:'下/秒', high:true, name:'引體向上/曲臂懸垂' },
    kettlebell: { vals:[40,30,30,20,20,10], unit:'下', high:true, name:'壺鈴平舉' },
  },
  core: {
    situp:  { vals:[42,31,35,23,20,16], unit:'下', high:true, name:'仰臥起坐' },
    plank:  { vals:[85,85,70,70,70,70], unit:'秒', high:true, name:'平板撐體' },
    crunch: { vals:[31,27,24,21,20,17], unit:'下', high:true, name:'仰臥捲腹' },
  },
  cardio: {
    run3k:   { vals:[885,1055,990,1140,1080,1260],   unit:'秒', high:false, name:'3000公尺跑步' },
    walk5k:  { vals:[2420,2660,2500,2750,2700,2940], unit:'秒', high:false, name:'5公里健走' },
    swim800: { vals:[1530,1710,1620,1800,1710,1890], unit:'秒', high:false, name:'800公尺游走' },
    shuttle: { vals:[50,40,35,25,25,15], unit:'趟', high:true, name:'20公尺折返跑' },
    rope:    { vals:[530,430,499,399,462,362], unit:'下', high:true, name:'5分鐘跳繩' },
  }
};
```

- `high: true`：成績越高越好（數值型）
- `high: false`：時間越短越好（計時型）
- 計時型數值統一換算為秒數儲存

**資料來源**：依據網路上國軍基本體能鑑測合格標準資料整理，僅供參考。

---

## 四、功能拆解（Input / Process / Output）

| 階段 | 內容 |
|------|------|
| **Input** | 性別（男/女）、年齡層（青年/壯年/中年）、各項體能成績（下數或分:秒） |
| **Process** | 標準索引計算 → 成績比對 → 達標率計算 → 評級判斷 → 訓練建議選擇 |
| **Output** | 合格/未達標標籤、達標率進度條、S/A/B/C 評級、差距說明、訓練建議卡片、雷達圖 |

### 分頁規劃
- Tab 1：📊 測驗試算（核心功能）
- Tab 2：⚖️ BMI 計算（輔助功能）
- Tab 3：📋 標準一覽（資料查詢）

---

## 五、核心計算邏輯（Core Logic）

### 標準索引計算
```javascript
function getStdIdx() {
  const g = gender === 'male' ? 0 : 1;
  const a = age === 'young' ? 0 : age === 'mid' ? 1 : 2;
  return a * 2 + g;
  // 0=青男, 1=青女, 2=壯男, 3=壯女, 4=中男, 5=中女
}
```

### 合格判定
```javascript
// 數值型（高分較好）
const pass = score >= standard;

// 計時型（低分較好，統一換算秒數）
const totalSec = minutes * 60 + seconds;
const pass = totalSec <= standard;
```

### 達標率計算
```javascript
// 數值型：成績 / 標準 × 100%（上限 100%）
const pct = Math.min(100, score / standard * 100);

// 計時型：標準 / 成績 × 100%（越快達標率越高）
const pct = Math.min(100, standard / totalSec * 100);
```

### 評級系統（對照標準表判定）
```javascript
function grade(pct) {
  if (pct >= 100) return { tag:'S 達標',  label:'達標' };
  if (pct >= 85)  return { tag:'A 接近',  label:'接近標準' };
  if (pct >= 70)  return { tag:'B 尚可',  label:'尚需加強' };
  return           { tag:'C 待加強', label:'明顯不足' };
}
```

### 訓練建議邏輯
```javascript
const TIPS = {
  upper: {
    pass:  '已達標，維持每週 2–3 次肌力訓練，嘗試增加負重或變化動作挑戰更高強度。',
    close: '差距不大，每日 3 組訓練，每組比現有成績多做 2–3 下，1–2 週內可望達標。',
    far:   '從基礎動作紮實練起，每天早晚各做 3 組，可從膝式伏地挺身開始循序漸進。'
  },
  // core, cardio 同樣結構...
};

function tipLevel(pct) {
  return pct >= 100 ? 'pass' : pct >= 80 ? 'close' : 'far';
}
```

---

## 六、UI 設計規範（Design Spec）

### 色彩系統
```css
--dark:     #1b3a1f;   /* 深軍綠，Header、主按鈕 */
--mid:      #2d5a35;   /* 中軍綠，表格標題 */
--khaki:    #c8a44a;   /* 卡其金，強調色、badge */
--bg:       #f0ede6;   /* 米白，頁面背景 */
--pass:     #1a7a34;   /* 合格綠 */
--fail:     #b52b1e;   /* 未達標紅 */
--warn:     #b87000;   /* 警示橙 */
```

### 評級顏色對應
```css
S 達標（>=100%）→ 綠色進度條 #1a7a34
A 接近（>=85%） → 青色進度條 #0c7a8a
B 尚可（>=70%） → 橙色進度條 #b87000
C 待加強（<70%）→ 紅色進度條 #b52b1e
```

### 字型
- 主字型：`Noto Sans TC`（Google Fonts）
- 數字字型：`Share Tech Mono`（Google Fonts）

### 版面
- 桌機：測驗項目 2×2 格、訓練建議 2 欄
- 手機：全部單欄，RWD 自動調整（最小 320px）

### 分析結論區結構
```
[類別名稱] [進度條█████░░] 達標率 XX%
           [評級標籤] 項目名稱 | 超標/還差 X 下
```

---

## 七、檔案結構

產出單一 `index.html`（或 `fitness.html`），包含：

```
index.html
├── <head>    Google Fonts、Chart.js CDN
├── <style>   全部 CSS（CSS Variables、RWD）
├── <body>    Header、三分頁（測驗/BMI/標準）
└── <script>  STD 資料庫、計算邏輯、分析結論、Chart.js 初始化
```

**允許的外部資源（CDN）：**
- `chart.js`（雷達圖）
- `Google Fonts: Noto Sans TC + Share Tech Mono`

---

## 八、GitHub 發布流程（Deployment Checklist）

```
□ 1. 建立 Public Repository（建議命名：fitness-calculator）
□ 2. 將 fitness.html 改名為 index.html
□ 3. 上傳至 Repository 根目錄
□ 4. Settings → Pages → Branch: main / Folder: / (root) → Save
□ 5. 等 1~2 分鐘，網址：https://<帳號>.github.io/<repo>/
□ 6. 手機瀏覽器開啟確認 RWD 正常
```

---

## 九、品質檢查清單（QA Checklist）

```
□ 男/女切換後，標準數值與合格標準正確更新
□ 三種年齡層切換後，標準正確更新
□ 計時型項目（分:秒）換算為秒數後比對正確
□ 達標率進度條寬度與顏色正確對應評級
□ 雷達圖在重新判定後正確刷新，無殘影
□ 訓練建議依達標率顯示 pass/close/far 正確版本
□ BMI 計算公式正確（體重 ÷ 身高m²）
□ 手機版不破版（320px 以上）
□ 無 JavaScript console 錯誤
□ 無需後端、無需 API Key，純前端可運作
```

---

## 十、可延伸應用（Reusability）

此 Skill 架構適用所有「標準比對型工具」，只需替換 STD 資料庫與 TIPS 訓練建議：

| 替換資料 | 可延伸的工具 |
|---------|------------|
| 消防員體能標準 | 消防特考體能計算器 |
| 警察特考標準 | 警察特考體能計算器 |
| 學生體適能常模 | 學生體適能自評工具 |
| 企業健康檢查指標 | 員工健康達標追蹤器 |
