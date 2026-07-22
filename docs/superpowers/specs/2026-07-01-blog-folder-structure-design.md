---
title: 技術部落格資料夾結構設計
date: 2026-07-01
status: approved
---

# 技術部落格資料夾結構設計

## 背景

公司技術部落格文件放在此資料夾（`D/Blog`），目前是空的，發布平台尚未決定，先當作純文字/Markdown 草稿庫使用。理想產出頻率是每月一篇，但先求產出第一篇。內容主要是「以前公司做過的展覽」的技術細節整理，一個展覽可能拆成多篇文章。

## 資料夾結構

```
Blog/
├── posts/
│   └── YYYY-MM-標題/            每篇文章一個資料夾
│       ├── index.md             主文章（含 YAML frontmatter）
│       ├── assets/              圖片、影片等媒體素材
│       └── notes/               草稿、額外資訊、參考資料
├── templates/
│   └── post-template.md         新文章起手式
```

- 資料夾命名統一使用 `YYYY-MM-標題`（標題為文章英文 slug 或簡短中文標題），不論是否為當月唯一文章都套用此規則，避免之後衝突要改名。
- 同一展覽拆成多篇文章時，各篇各自建立資料夾，透過 frontmatter 的 `exhibition` 欄位關聯，不依賴資料夾名稱編碼。

## Frontmatter 規範

`index.md` 開頭統一包含：

```yaml
---
title: ""
date: YYYY-MM-DD
status: draft   # draft | published
tags: []
exhibition: ""   # 展覽名稱，方便同展覽多篇串連
---
```

- `status`：追蹤草稿 / 已發布，方便日後搜尋進度。
- `exhibition`：同一展覽多篇文章共用同一個值，方便未來查詢關聯文章。

## 素材與草稿

- `assets/`：存放該篇文章對應的圖片、影片，文章內以相對路徑引用（如 `./assets/xxx.png`），未來搬遷到任何部落格系統/CMS 都容易轉移。
- `notes/`：存放寫作過程中的草稿、參考資料、額外整理，不會混入正式發布內容。

## 新文章流程

1. 複製 `templates/post-template.md` 到 `posts/YYYY-MM-標題/index.md`
2. 建立同層 `assets/` 與 `notes/` 子資料夾
3. 素材放入 `assets/`，草稿/參考資料放入 `notes/`
4. 完稿後將 frontmatter 的 `status` 改為 `published`

## 未決定事項（非本次範圍）

- 最終發布平台（Hexo/Hugo/自架系統等）尚未決定；本結構刻意保持平台無關，之後可依需要撰寫轉換腳本。
- tags 分類方式尚未制定，先留空自由填寫，累積幾篇後再regularize。
