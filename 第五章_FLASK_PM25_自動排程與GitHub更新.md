# 第五章：自動排程與 GitHub Actions 自動更新

本章將教你如何透過 GitHub Actions 自動執行 PM2.5 資料更新腳本，實現每日定時抓取與寫入資料庫。

---

## 🎯 教學目標

- 建立更新腳本 `update_pm25.py`
- 撰寫 GitHub Actions 工作流程
- 設定每日自動執行排程

---

## 1️⃣ 確保專案結構正確

專案資料夾應包含以下檔案結構：

```
pm25-flask/
├── .github/
│   └── workflows/
│       └── pm25-update.yml
├── update_pm25.py
├── pm25_utils.py
├── db_utils.py
├── requirements.txt
└── ...
```

---

## 2️⃣ 建立 GitHub Actions Workflow

建立 `.github/workflows/pm25-update.yml` 檔案，內容如下：

```yaml
name: Update PM2.5 DB

on:
  schedule:
    - cron: '30 15 * * *'   # 每天 23:30 台灣時間（UTC+8）
  workflow_dispatch:        # 允許手動觸發

jobs:
  update-db:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt

      - name: Run update script
        env:
          API_KEY: ${{ secrets.PM25_API_KEY }}
        run: |
          python update_pm25.py
```

---

## 3️⃣ 修改更新腳本支援環境變數

請將 `update_pm25.py` 改為以下格式：

```python
import os
from pm25_utils import fetch_pm25_data, transform_pm25_data, save_to_mysql

api_key = os.environ.get("API_KEY")

if not api_key:
    raise Exception("未提供 API_KEY 環境變數")

records = fetch_pm25_data(api_key)
data_list = transform_pm25_data(records)
save_to_mysql(data_list)
```

---

## 4️⃣ 設定 GitHub Secrets（機密資料）

1. 打開你的 GitHub 專案
2. 點選「Settings」→「Secrets and variables」→「Actions」
3. 新增一筆 Secret：
   - Name：`PM25_API_KEY`
   - Value：你的 API 金鑰

這樣 GitHub 就可以安全地讀取你的金鑰並自動執行更新腳本。

---

## 5️⃣ 手動執行與排程確認

- 可進入 GitHub → Actions → `Update PM2.5 DB` → 手動執行測試
- 確認每日排程會在 UTC 時間 `15:30`（台灣時間 `23:30`）自動執行

---

## ✅ 本章結束

你已成功將 PM2.5 更新腳本自動化，未來每天會自動更新資料。  
接下來你可以考慮部署到線上（Render、PythonAnywhere、Railway）讓別人也能瀏覽圖表。
