# 文字讀心術 Lab — 設定教學

| 檔案 | 用途 |
|---|---|
| `index.html` | 學生用的上機網頁，放到 GitHub Pages |
| `apps-script/Code.gs` | 後端方案 A：貼到 Google Apps Script，資料存進 Google 試算表 |
| `supabase/setup.sql` | 後端方案 B：貼到 Supabase，資料存進 Supabase 資料庫 |

**後端二選一就好**，都是免費的：

| | 方案 A：Google 試算表 | 方案 B：Supabase |
|---|---|---|
| 看資料的地方 | Google 試算表 | Supabase 後台的表格（Table Editor），可匯出 CSV |
| 需要的帳號 | Google | Supabase（可用 Email 註冊） |
| 隱藏不當句子 | 在「隱藏」欄打勾 | 把 `hidden` 改成 `true` |
| 注意 | 第一次部署要授權 | 免費專案太久沒使用會自動暫停，上課前要登入確認是開著的 |

建議照順序做：**先做後端，拿到網址，再填進網頁，最後放上 GitHub**。

---

## 方案 B：Supabase（不用 Google）

1. 到 supabase.com 用 Email 註冊並登入。
2. 按 **New project**，取個名字，設定資料庫密碼（自己記好，網頁用不到），地區選離台灣近的，例如 Tokyo 或 Singapore，按建立，等一兩分鐘。
3. 左邊選單點 **SQL Editor → New query**，把 `supabase/setup.sql` 全部貼上，按 **Run**。看到 Success 就代表兩張表（`board` 排行榜、`feedback` 回饋）都建好了。
4. 拿兩個值：
   - **Project URL**：在專案首頁或 **Project Settings → Data API** 可以找到，長得像 `https://abcdefgh.supabase.co`
   - **Publishable key**：在 **Project Settings → API Keys**，開頭是 `sb_publishable_`（舊專案叫 **anon public key**，是一長串 `eyJ...`，也可以用）
   - ⚠️ **不要用 secret key / service_role key**，那把放進網頁別人就能改你所有資料。
5. 打開 `index.html`，找到 `const CONFIG = {`，把兩個值填進去：

```js
  SUPABASE_URL: 'https://abcdefgh.supabase.co',
  SUPABASE_KEY: 'sb_publishable_xxxx',
```

6. 接著跳到下面的「第三部分：放上 GitHub Pages」。

**之後怎麼看資料**：Supabase 左邊選單 **Table Editor**，點 `feedback` 就是所有回饋，右上角可以 **Export → CSV**。
**隱藏不當句子**：Table Editor 點 `board`，找到那一列，把 `hidden` 改成 `true`。

> 為什麼 publishable key 可以放在網頁裡？因為 `setup.sql` 已經設好規則：網頁只能「新增句子」「讀沒被隱藏的句子」「新增回饋」，不能修改、刪除，也讀不到任何回饋內容。

---

## 方案 A 第一部分：建立後端（Google 試算表 + Apps Script）

1. 打開 Google 試算表，建立一份新的空白試算表，取名例如「文字讀心術 Lab 資料」。
2. 上方選單點 **擴充功能 → Apps Script**，會開一個新分頁。
3. 把編輯器裡原本的程式碼全部刪掉，貼上 `apps-script/Code.gs` 的完整內容，按儲存（磁碟圖示）。
4. 右上角點 **部署 → 新增部署作業**。
5. 左邊齒輪「選取類型」選 **網頁應用程式**，然後設定：
   - 說明：隨便填，例如 `v1`
   - 執行身分：**我**
   - 誰可以存取：**所有人**（這樣學生不用登入就能送資料）
6. 按 **部署**。第一次會要求授權：選你的 Google 帳號 →「進階」→「前往（不安全）」→ 允許。這是因為程式是你自己寫的、沒有經過 Google 審查，屬於正常流程。
7. 部署完成後會顯示一串**網頁應用程式網址**，長得像：
   `https://script.google.com/macros/s/xxxxxxxx/exec`
   把它複製下來。

**測試一下**：把網址貼到瀏覽器，後面加上 `?type=board`，看到 `{"ok":true,"items":[]}` 就代表成功了。

> ⚠️ 之後如果修改了 `Code.gs`，要到 **部署 → 管理部署作業 → 編輯（鉛筆）→ 版本選「新版本」→ 部署**，改動才會生效。網址不會變。

---

## 方案 A 第二部分：把網址填進網頁

用任何文字編輯器（記事本、VS Code 都可以）打開 `index.html`，找到這一段（大約在 `<script>` 開頭）：

```js
const CONFIG = {
  API_URL: '',
```

把剛剛複製的網址貼進引號裡：

```js
  API_URL: 'https://script.google.com/macros/s/xxxxxxxx/exec',
```

存檔。

---

## 第三部分：放上 GitHub Pages

1. 登入 GitHub，右上角 **＋ → New repository**。
   - Repository name：例如 `text-mind-lab`
   - 選 **Public**（免費帳號的 GitHub Pages 需要公開）
   - 按 **Create repository**
2. 在新 repo 頁面點 **uploading an existing file**，把這些檔案拖進去，按 **Commit changes**：
   - `index.html`
   - `ntusd-positive.txt`、`ntusd-negative.txt`（選用）
   - `NTUSD-LICENSE.txt`（有放 NTUSD 就要附）

   repo 是公開的，`Code.gs`、`setup.sql` 裡沒有密碼，要一起放也可以，但不放比較單純。
3. 到 repo 的 **Settings → Pages**：
   - Source 選 **Deploy from a branch**
   - Branch 選 **main**、資料夾選 **/(root)**，按 **Save**
4. 等 1–2 分鐘，重新整理頁面，上方會出現網址，例如：
   `https://你的帳號.github.io/text-mind-lab/`

這就是要給學生的網址，可以拿去產生 QR Code 放進簡報。

**之後修改網頁**：在 repo 裡點檔案 → 右上角鉛筆圖示直接改，或重新上傳同名檔案覆蓋，按 **Commit changes**，等 1–2 分鐘就會更新。

> 想直接打開某個分頁，可以在網址後面加：`#lab1`、`#lab2`、`#lab3`、`#feedback`。
> 例如回饋表單的 QR Code 可以用 `https://你的帳號.github.io/text-mind-lab/#feedback`。

---

## 上課當天

**上機一的排行榜投影**：在講者電腦打開網頁，按排行榜旁邊的 **投影模式**，畫面只剩排行榜、字放大，每 8 秒自動更新。

**有人寫了不適當的句子**：方案 A 在試算表「排行榜」那一列的「隱藏」欄打勾；方案 B 在 Supabase 把那一列的 `hidden` 改成 `true`。下一次更新就會從網頁消失。

**看回饋**：方案 A 打開試算表的「回饋」工作表；方案 B 打開 Supabase 的 `feedback` 表，都可以匯出成檔案。

**上機一結束後的 AI 示範**：從排行榜挑 3–5 句最精彩的，貼到 Claude 或其他 AI 聊天視窗問「這句是正面、中性還是負面？為什麼？」，全班一起看 AI 能不能看懂。

---

## 換成你自己的資料

- **上機二（20 則評論）**：在 `index.html` 搜尋 `const L2 = [`，每一筆是 `['評論', 'AI 主題']`。
- **上機三（完整流程）**：搜尋 `const L3 = {`，每一筆是 `['原始評論', 'AI 主題', 'pos/neu/neg']`。
- **字典**：搜尋 `const LEXICON`，可以自己加詞（`'詞': 1` 是正面、`'詞': -1` 是負面）。

### 加入 NTUSD（台大情緒詞典，選用）

1. 取得 NTUSD 的兩個檔案：正面詞和負面詞，各是一行一個詞的 txt。NTUSD 主要開放學術研究使用，使用前請確認授權。
2. **一定要用繁體中文版**。網路上流傳很多簡體版，學生打的是繁體字，用簡體版會幾乎查不到詞。
3. 把兩個檔案改名成 `ntusd-positive.txt` 和 `ntusd-negative.txt`，跟 `index.html` 一起上傳到 GitHub repo 的同一層。
5. 把 NTUSD 的 MIT 授權全文存成 `NTUSD-LICENSE.txt`，一起放進資料夾。MIT 授權要求散布時附上版權聲明。
6. 打開網址，上機一「字典的判斷」右上角會從「示範字典 · 約 40 個詞」變成「NTUSD ＋ 示範字典 · 幾千個詞」，就代表載入成功。

補充：
- 檔案是 UTF-8 或 UTF-16 都可以，網頁會自動判斷。
- 示範字典原本的詞（例如「好棒棒」+1）會保留，跟 NTUSD 衝突時以示範字典為準；同時出現在正面和負面清單的詞會略過。
- 不想用 NTUSD 時，把檔案從 repo 刪掉，或把 `CONFIG` 裡的 `NTUSD_POS`、`NTUSD_NEG` 改成 `''`。
- 換成 NTUSD 之後，記得自己先玩幾句。詞變多以後，有些判斷會變得很奇怪（例如某個單字被當成情緒詞），上課前先知道比較好解釋。
- 直接雙擊打開本機的 `index.html` 時，瀏覽器會擋住讀檔，所以 NTUSD 要放上 GitHub Pages 才會生效。想在自己電腦先測，可以在資料夾打開終端機，輸入 `python -m http.server 8000`（Mac 用 `python3`），再打開 `http://localhost:8000`。

AI 的主題和情感是事先跑好的，換資料時記得一起更新（可以把評論貼給 Claude，請它照格式產生）。

---

## 上課前檢查清單

- [ ] 方案 A：後端網址加上 `?type=board` 能看到 `{"ok":true,...}`
- [ ] 方案 B：上課前一兩天登入 Supabase，確認專案沒有被暫停
- [ ] 打開 GitHub Pages 網址，上機一的排行榜狀態顯示「全班共 0 句 · 自動更新」（不是「尚未連接後端」）
- [ ] 自己送一句測試句子，確認試算表有出現，再把它打勾隱藏
- [ ] 送一份測試回饋，確認「回饋」工作表有資料
- [ ] 用學校的電腦打開一次，確認網路沒有擋 `github.io`，以及 `script.google.com`（方案 A）或 `supabase.co`（方案 B）
