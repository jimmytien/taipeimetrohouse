# 大都會物業房務管理系統 - 專案分析報告

這份文件旨在幫助您快速了解「大都會物業房務管理系統」的網頁架構、程式碼發佈流程，以及預設密碼與重要專案參數。

## 1. 專案架構與技術棧 (Web Architecture)

此專案屬於傳統的多頁面網頁應用程式 (Multi-Page Application, MPA)，並無使用 React/Vue 等前端框架，而是使用純 HTML/JS 搭配 Firebase 雲端服務組成。

### 前端 (Frontend - 位於 `public/` 目錄)
- **UI 框架**：使用 **Bootstrap 4**，套用開源的樣式模板 **[SB Admin 2](https://startbootstrap.com/theme/sb-admin-2)**。
- **核心套件**：依賴 **jQuery 3.4.1** 處理 DOM 操作、**Chart.js** 繪製圖表及原生的 **Firebase Web SDK (v7.11.0)** 處理資料互動。
- **程式結構**：
  - `.html`：包括 `index.html`, `login.html`, `tasks.html`, `room.html` 等，每個頁面都有獨立的 HTML 檔案。
  - `js/`：存放自定義的業務邏輯，例如 `login.js`, `cleanhouse.js` 等。
  - `css/` 與 `scss/`：樣式文件。開發時若有修改 `scss`，需要依賴 Gulp 進行編譯。
  - `package.json`：定義了前端用到的 npm 套件及 Gulp 編譯腳本。

### 後端 (Backend - 架構於 Firebase 上)
專案全面採用 Google Firebase 解決方案（專案 ID：`taipeimetrohouse`）：
- **Firebase Hosting**：用於代管及發佈 `public/` 底下的所有靜態前端網頁。
- **Firebase Realtime Database**：即時資料庫，用於儲存使用者、任務、房屋等數據。
- **Firebase Authentication**：負責處理使用者的登入與身份驗證。
- **Firebase Cloud Functions** (位於 `functions/` 目錄)：
  - 使用 Node.js 12 撰寫。
  - 主要利用 `Express` 建置 HTTP API 端點 (如 `/sendMail`, `/sendCleanHouseMail` 等)。
  - 利用 `Nodemailer` 自動寄發多種業務通知信（如電子簽約、清潔通知等）。

---

## 2. 如何發佈與上架 (Deployment Guide)

發佈本專案必須透過 **Firebase CLI** 來進行。請確保您的電腦已安裝 node.js，並且安裝了全域的 Firebase 命令行工具。

### Step 1: 環境準備
打開終端機 (命令提示字元)，安裝 Firebase CLI 並登入擁有此專案權限的 Google 帳號：
```bash
npm install -g firebase-tools
firebase login
```

### Step 2: 前端編譯 (若未修改樣式可跳過)
專案的前端有使用 Gulp 自動編譯 SCSS 為 CSS。若有修改設計，上架前必須重新編譯：
```bash
cd public
npm install
npm start   # 此指令會執行 gulp watch 並啟動開發伺服器
# 或只執行 gulp 進行編譯
```

### Step 3: 發佈專案至 Firebase
您可以選擇一次性發佈所有內容，或是分開發佈：

**A. 發佈全部 (網頁前端 + 雲端函數)**
```bash
# 在專案根目錄下 (即包含 firebase.json 的那一層)
firebase deploy
```

**B. 僅發佈前端網頁 (Hosting)**
```bash
firebase deploy --only hosting
```

**C. 僅發佈雲端函數 (Functions)**
```bash
firebase deploy --only functions
# 或使用 functions 目錄內準備好的指令：
cd functions && npm run deploy
```

---

## 3. 預設密碼與重要專案參數 (Passwords & Settings)

> [!WARNING]
> **資料庫權限已過期斷線 (極度重要)**
> 您專案根目錄的 `database.rules.json` 設定了讀寫權限的最後期限為 `1612627200000` (即 2021年2月7日)。**目前您的專案資料庫是處於全面封鎖讀寫的狀態。** 如果網頁無法讀取資料，是因為此規則已過期，必須延後 timestamp 或修改基於角色的驗證規則後重新發佈 (`firebase deploy --only database`)。

### a) 登入系統機制 (無預設密碼)
觀察 `public/login.html` 與 `public/js/login.js`，原本的信箱/密碼登入表單已被隱藏註解。**目前系統唯一的登入方式是「Login with Google」(Google 第三方登入)。**
因此，**系統沒有管理員的「預設帳號/密碼」**。管理員或員工都是使用自己私人的 Google 帳號授權登入後，再透過 Firebase Database 判定權限。

### b) 系統自動寄信密碼 (SMTP)
在 `functions/index.js` 中，為了讓系統能自動寄發通知信，開發者寫死綁定了一組 GMail 帳號以及一組**應用程式專用密碼**：
- **寄件人信箱**：`metrohousetaipei@gmail.com`
- **密碼**：`lrwaxvvpmumhkwfw`

> [!TIP]
> 如果未來系統無法正常寄出信件，很可能是此 Google 帳號的這組密碼被撤銷，或是需要重新登入該信箱解除安全封鎖。

### c) Firebase 專案金鑰 (API Key)
前端連接 Firebase 的所有相關金鑰位於 `public/js/login.js`：
- **Project ID**: `taipeimetrohouse`
- **Database URL**: `https://taipeimetrohouse-default-rtdb.firebaseio.com`
這些是前端發出請求的公開識別碼，本身只要透過正常連線機制即可使用，不算是帳號密碼。
