# 泰時刻備課系統 - 泰文歌曲備課 MVP

最簡單可跑通的第一版：純 HTML 前端（GitHub Pages）＋ Cloudflare Worker 後端（存放 API 金鑰、呼叫 Claude API）＋ 資料存瀏覽器 localStorage。

## 目錄結構

```
thai-lesson-mvp/
├── worker/
│   ├── worker.js         Cloudflare Worker 程式碼
│   └── wrangler.toml     Worker 部署設定
└── frontend/
    ├── index.html        主程式（歌曲列表、批次審核、下載）
    └── assets/
        └── song-template.html   最終學習頁面樣板（沿用你原本的泰時刻歌詞頁模板）
```

## 一、部署 Cloudflare Worker

1. 本機安裝 wrangler（如果還沒裝）：
   ```
   npm install -g wrangler
   wrangler login
   ```
2. 進入 `worker` 資料夾部署：
   ```
   cd worker
   wrangler deploy
   ```
3. 部署完成後設定兩組密鑰（系統會提示輸入值，直接貼上即可）：
   ```
   wrangler secret put ANTHROPIC_API_KEY
   wrangler secret put APP_TOKEN
   ```
   - `ANTHROPIC_API_KEY`：你的 Anthropic API 金鑰（[console.anthropic.com](https://console.anthropic.com) 取得）
   - `APP_TOKEN`：自己隨便設一組密碼字串（例如一長串亂碼），前端會用這組密碼呼叫 Worker，避免網址被別人盜用
4. 部署完成後你會拿到一個網址，長得像：
   ```
   https://thai-lesson-prep-worker.你的帳號.workers.dev
   ```
   這組網址等一下要填進前端設定。
5. （建議）部署後把 `wrangler.toml` 裡的 `ALLOWED_ORIGIN` 改成你的 GitHub Pages 網址（例如 `https://你的帳號.github.io`），改完重新 `wrangler deploy` 一次，這樣只有你的前端頁面能呼叫這個 Worker。

## 二、部署前端到 GitHub Pages

1. 建一個新的 GitHub repository（例如 `thai-lesson-prep`）
2. 把 `frontend` 資料夾裡的內容（`index.html` 和 `assets/`）上傳到 repository 的根目錄
3. 到 repository 的 Settings → Pages，Source 選擇你上傳的分支（通常是 `main`）與根目錄 `/`
4. 存檔後幾分鐘，會拿到網址：`https://你的帳號.github.io/thai-lesson-prep/`

## 三、第一次使用

1. 打開上面的網址
2. 第一次會要求輸入：
   - **Cloudflare Worker 網址**：貼上步驟一拿到的網址
   - **存取密碼（APP_TOKEN）**：貼上你自己設定的那組密碼
3. 存檔後就會進到歌曲列表畫面，可以開始「＋ 新增歌曲」

## 使用流程

1. 新增歌曲：填歌名、歌手、YouTube ID（選填）、貼上整理好的完整歌詞（一行一句）
2. 系統會自動每 5 句分一批，逐批呼叫 Claude 生成：中文翻譯、單字斷句、每字音標、每字例句（泰文＋音標＋中文）
3. 每批生成後可以看畫面確認，覺得不對可以輸入調整意見重新生成該批，滿意後按「確認，下一批」
4. 全部批次跑完後，系統會自動套用學習頁面樣板，產生一個可下載的 `.html` 檔案
5. 回到列表可以看到這首歌狀態變成「已完成」，可以隨時再次下載；也可以按「刪除」清除這首歌在瀏覽器裡的所有資料

## 目前 MVP 的已知限制（之後可以再優化）

- **資料只存在目前這台電腦、這個瀏覽器**：換裝置、換瀏覽器、或清除瀏覽器資料就會不見。之後如果要多裝置同步，可以把 Worker 加上 Cloudflare KV 或 D1 資料庫。
- **沒有批次內單句編輯功能**：目前只能整批重新生成，還不能只改某一句或某個單字。
- **簡單權杖驗證**：APP_TOKEN 是前端明碼呼叫，足以擋掉隨機掃描濫用，但不是嚴謹的身份驗證，僅適合個人自用。
- **模型**：目前用的是 `claude-sonnet-5`，如果音標或例句品質不夠精準，可以在 `worker.js` 裡把 `MODEL` 改成 `claude-opus-5`（費用較高，品質較好）。

## 費用提醒

每批次呼叫一次 Claude API，依歌詞長度與單字量計費（輸出 token 越多費用越高），可以在 [Anthropic Console](https://console.anthropic.com) 的用量頁面追蹤花費。
