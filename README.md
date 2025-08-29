# RSVP 表單系統部署指南

這是一個完整的 RSVP 表單系統，包含前端網頁、Google Apps Script 後端和 Google Sheets 資料儲存。

## 系統架構

- **前端**: 靜態 HTML 網頁 (GitHub Pages)
- **後端**: Google Apps Script Web App
- **資料庫**: Google Sheets
- **相簿**: Google Photos + publicalbum 嵌入

## 功能特色

- ✅ 響應式設計，支援手機和桌面
- ✅ 表單驗證和錯誤處理
- ✅ 即時統計資訊顯示
- ✅ Google Photos 相簿整合
- ✅ 安全的 token 驗證機制
- ✅ CORS 支援，允許跨域請求

## 快速開始

### 1. 建立 Google Sheets

1. 開啟 [Google Sheets](https://sheets.google.com)
2. 建立新試算表，命名為 `RSVP Responses`
3. 在 A1-F1 填入標題列：
   ```
   A1: timestamp
   B1: name  
   C1: email
   D1: attend
   E1: num_people
   F1: note
   ```
4. 複製試算表 ID（URL 中的長字串）

### 2. 部署 Google Apps Script

1. 開啟 [Google Apps Script](https://script.google.com)
2. 建立新專案，命名為 `RSVP Backend`
3. 將 `Code.gs` 的內容貼入編輯器
4. 修改設定：
   ```javascript
   var SHEET_ID = "你的試算表ID";
   ```
5. 執行 `setSecret()` 函式設定密鑰：
   - 修改函式中的密鑰值（使用長隨機字串）
   - 點選「執行」按鈕
   - 授權存取權限
6. 部署為 Web App：
   - 點選「部署」→「新部署」
   - 選擇類型：Web app
   - 執行身分：我 (你的帳戶)
   - 存取權限：任何人
   - 點選「部署」
   - 複製 Web App URL

### 3. 設定前端網頁

1. 修改 `index.html` 中的設定：
   ```javascript
   const WEBAPP_URL = "你的Apps Script URL?token=你的密鑰";
   ```
2. 修改 Google Photos 相簿連結（可選）：
   ```html
   data-link="你的Google Photos分享連結"
   ```

### 4. 部署到 GitHub Pages

1. 在 GitHub 建立新 repository
2. 上傳 `index.html` 到 repository
3. 在 Settings → Pages 中啟用 GitHub Pages
4. 選擇 Source: main branch
5. 等待幾分鐘後即可存取網站

## 詳細設定指南

### Google Apps Script 設定

#### 必要設定
```javascript
// 在 Code.gs 中修改這些值
var SHEET_ID = "你的Google Sheets ID";

// 執行 setSecret() 函式時設定
function setSecret() {
  var secret = "至少32字符的隨機字串";
  PropertiesService.getScriptProperties().setProperty('WEBHOOK_SECRET', secret);
}
```

#### 權限設定
- 執行身分：我（使用你的權限存取試算表）
- 存取權限：任何人（允許匿名 POST 請求）

#### API Endpoints
- `POST /exec?token=密鑰` - 提交表單資料
- `GET /exec?action=list&token=密鑰` - 取得回應列表
- `GET /exec?action=health` - 健康檢查

### 前端設定

#### 必要修改
```javascript
// 在 index.html 中修改
const WEBAPP_URL = "https://script.google.com/macros/s/部署ID/exec?token=密鑰";
```

#### Google Photos 相簿設定
1. 在 Google Photos 建立相簿
2. 新增照片到相簿
3. 點選「分享」→「建立連結」
4. 複製分享連結
5. 在 HTML 中修改：
```html
<div class="pa-carousel-widget" 
     data-link="你的Google Photos分享連結"
     data-title="相簿標題">
</div>
```

### 安全性設定

#### Token 安全
- 使用至少 32 字符的隨機字串作為密鑰
- 不要在程式碼中硬編碼密鑰，使用 PropertiesService
- 定期更換密鑰

#### 資料驗證
- 後端會驗證所有必要欄位
- 限制人數範圍 (1-50)
- 防止 XSS 攻擊（資料清理）

## 測試指南

### 本機測試
1. 直接開啟 `index.html` 檔案
2. 填寫表單測試（需要已部署的 Apps Script）

### 線上測試
1. 開啟 GitHub Pages URL
2. 測試表單提交
3. 檢查 Google Sheets 是否新增資料
4. 測試相簿是否正常顯示

### API 測試
使用 curl 或 Postman 測試：
```bash
# 提交表單
curl -X POST "你的Apps Script URL?token=密鑰" \
  -H "Content-Type: application/json" \
  -d '{"name":"測試","email":"test@example.com","attend":"yes","num_people":2,"note":"測試留言"}'

# 取得回應列表
curl "你的Apps Script URL?action=list&token=密鑰"

# 健康檢查
curl "你的Apps Script URL?action=health"
```

## 常見問題

### Q: 表單提交後沒有反應
**A: 檢查以下項目**
- Chrome DevTools Console 是否有錯誤訊息
- Apps Script URL 和 token 是否正確
- Google Sheets ID 是否正確
- Apps Script 是否有執行權限

### Q: CORS 錯誤
**A: Apps Script 已設定 CORS headers，如果仍有問題：**
- 確認 Apps Script 部署時選擇「任何人」存取
- 檢查瀏覽器是否封鎖跨域請求
- 嘗試使用無痕模式測試

### Q: 相簿不顯示
**A: 檢查以下項目**
- Google Photos 相簿是否已設定分享連結
- 分享連結是否正確貼入 HTML
- 相簿是否有照片
- publicalbum script 是否載入成功

### Q: 統計資訊不顯示
**A: 統計功能是可選的，需要：**
- 在 Apps Script 中實作 `action=list` endpoint
- 前端正確呼叫統計 API
- 有足夠的資料可供統計

## 進階功能

### 自訂樣式
修改 `index.html` 中的 CSS 變數：
```css
:root {
  --pink-1: #ffd7e6;    /* 背景漸層色 */
  --pink-2: #f7c0d1;    /* 背景漸層色 */
  --accent: #e21f4d;    /* 主色調 */
  --card-bg: #fff;      /* 卡片背景 */
  --radius: 12px;       /* 圓角大小 */
}
```

### 新增欄位
1. 在 Google Sheets 新增欄位標題
2. 在 HTML 表單新增對應的 input 欄位
3. 在 Apps Script 的 `doPost` 函式中處理新欄位
4. 更新 `ensureHeaders` 函式中的預期標題

### 郵件通知
在 Apps Script 中新增 Gmail API 功能：
```javascript
function sendNotificationEmail(formData) {
  GmailApp.sendEmail(
    "管理員信箱",
    "新的 RSVP 回覆",
    `收到新的回覆：${formData.name} (${formData.attend})`
  );
}
```

### 資料匯出
可以透過 Google Sheets 的匯出功能：
- 檔案 → 下載 → CSV/Excel
- 或使用 Apps Script 建立自動匯出功能

## 支援與更新

如需技術支援或功能建議，請：
1. 檢查本文件的常見問題section
2. 查看 Google Apps Script 的執行記錄
3. 檢查瀏覽器開發者工具的錯誤訊息

## 版本歷史

- v1.0 - 初始版本，包含基本 RSVP 功能
- 包含響應式設計、相簿整合、統計功能

---

**注意事項：**
- 請確保 Google 帳戶有足夠權限建立 Sheets 和 Apps Script
- 建議使用 HTTPS 部署前端頁面以確保安全性
- 定期備份 Google Sheets 資料