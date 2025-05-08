# Flask PM2.5 空氣品質監測系統  
### 完整教學手冊  
作者：Jerry（GPT 教學整理）  
版本：v1.0  
日期：2025 年  

---

## 📖 目錄

1. [第一章：環境與專案初始化](#第一章環境與專案初始化)
2. [第二章：建立資料庫](#第二章建立資料庫)
3. [第三章：取得 PM2.5 開放資料並寫入資料庫](#第三章取得-pm25-開放資料並寫入資料庫)
4. [第四章：建立前端網站與圖表展示](#第四章建立前端網站與圖表展示)
5. [第五章：自動排程與 GitHub Actions 自動更新](#第五章自動排程與-github-actions-自動更新)
6. [第六章：部署 Flask PM2.5 專案到線上](#第六章部署-flask-pm25-專案到線上)
7. [第七章：功能擴充與進階應用](#第七章功能擴充與進階應用)

---

# Flask PM2.5 空氣品質監測系統 - 完整教學手冊

作者：Jerry（GPT 教學整理）

---

# 第一章：環境與專案初始化

本章節將協助你建立一個乾淨的 Flask 專案環境，準備好日後開發 PM2.5 資料監控網站。

---

## 🎯 教學目標

- 安裝 Python 與虛擬環境
- 初始化 Flask 專案結構
- 安裝必要套件
- 建立 Git 專案並忽略不必要的檔案

---

## 1️⃣ 安裝 Python 與建立虛擬環境

請確認你已安裝好 Python 3.8 或以上版本。可用以下指令檢查版本：

```bash
python --version
```

接著建立一個新的虛擬環境 `pm25-env`：

```bash
python -m venv pm25-env
```

啟動虛擬環境：

- **Windows**：

  ```bash
  pm25-env\Scripts\activate
  ```

- **macOS / Linux**：

  ```bash
  source pm25-env/bin/activate
  ```

---

## 2️⃣ 安裝 Flask 與相關套件

在虛擬環境中安裝 Flask 以及其他所需套件：

```bash
pip install flask requests pymysql
```

📌 **補充說明：**

| 套件名稱 | 用途 |
|----------|------|
| `flask` | 建立 Web API 與前端網站 |
| `requests` | 抓取政府開放 PM2.5 資料 |
| `pymysql` | Python 連接 MySQL 資料庫 |

為了將目前安裝的套件記錄下來，執行以下指令：

```bash
pip freeze > requirements.txt
```

---

## 3️⃣ 建立專案結構

請在你的專案資料夾內建立以下結構：

```
pm25-flask/
├── app.py              # 主程式
├── requirements.txt    # 套件清單
├── templates/          # HTML 模板
│   └── index.html
├── static/             # 靜態資源（CSS/JS）
└── pm25_utils.py       # 資料處理工具
```

📌 **結構說明：**

| 檔案/資料夾       | 功能說明 |
|------------------|----------|
| `app.py`         | 主 Flask 程式入口 |
| `templates/`     | 存放 HTML 模板 |
| `static/`        | 存放靜態資源（CSS/JS 圖片等） |
| `pm25_utils.py`  | 放 API 抓取與資料轉換邏輯 |
| `requirements.txt` | 套件清單（可用於部署） |

---

## 4️⃣ 初始化 Git 專案

若你打算使用 Git 版本控制，在資料夾中執行：

```bash
git init
```

接著建立 `.gitignore` 檔案，加入以下內容：

```gitignore
__pycache__/
pm25-env/
*.pyc
.env
```

---

## 5️⃣ 測試 Flask 是否能正常啟動

請先建立 `app.py`，並寫入以下內容：

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return "Flask PM2.5 專案初始化成功！"

if __name__ == '__main__':
    app.run(debug=True)
```

執行伺服器：

```bash
python app.py
```

打開瀏覽器，前往 `http://127.0.0.1:5000/`  
✅ 若顯示「Flask PM2.5 專案初始化成功！」即代表成功！

---

## 📌 小提醒

- 離開虛擬環境可輸入：

  ```bash
  deactivate
  ```

---

## ✅ 本章結束

你已完成專案初始化與環境設定，下一章我們將建立 **MySQL 資料庫**，準備儲存 PM2.5 資料。


---

# 第二章：建立資料庫

本章節將協助你建立 MySQL 資料庫，設計 PM2.5 資料表，並透過 Python 成功連接資料庫，為後續儲存資料做準備。

---

## 🎯 教學目標

- 建立 MySQL 資料庫與表格
- 設計資料欄位結構
- 撰寫 Python 資料庫連線與測試程式

---

## 1️⃣ 建立 MySQL 資料庫與帳號

請先確認你已安裝 MySQL Server，並能使用以下指令進入命令列：

```bash
mysql -u root -p
```

登入後，建立一個資料庫：

```sql
CREATE DATABASE pm25_db DEFAULT CHARSET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

你也可以另外建立專用帳號（選擇性）：

```sql
CREATE USER 'pm25user'@'localhost' IDENTIFIED BY 'pm25pass';
GRANT ALL PRIVILEGES ON pm25_db.* TO 'pm25user'@'localhost';
FLUSH PRIVILEGES;
```

---

## 2️⃣ 建立資料表結構

選擇剛剛建立的資料庫，並建立資料表：

```sql
USE pm25_db;

CREATE TABLE pm25_data (
    id INT AUTO_INCREMENT PRIMARY KEY,
    county VARCHAR(20),
    site VARCHAR(30),
    pm25 FLOAT,
    datacreationdate DATETIME,
    unit VARCHAR(10),
    UNIQUE KEY uq_site_time (site, datacreationdate)
);
```

📌 補充說明：

| 欄位名稱         | 說明           |
|------------------|----------------|
| `county`         | 縣市           |
| `site`           | 測站名稱       |
| `pm25`           | PM2.5 數值     |
| `datacreationdate` | 資料時間     |
| `unit`           | 單位（如 µg/m³）|

---

## 3️⃣ 安裝並撰寫 Python 資料庫連線程式

我們使用 `pymysql` 套件來操作資料庫。請先確認已安裝：

```bash
pip install pymysql
```

建立一個 `db_utils.py` 檔案，加入以下連線函式：

```python
import pymysql

def open_db():
    return pymysql.connect(
        host="localhost",
        user="pm25user",       # 或 root
        password="pm25pass",   # 或 root 密碼
        database="pm25_db",
        charset="utf8mb4"
    )
```

---

## 4️⃣ 測試連線是否成功

新增 `test_db.py` 檔案，寫入以下內容：

```python
from db_utils import open_db

try:
    conn = open_db()
    print("✅ 資料庫連線成功")
except Exception as e:
    print("❌ 連線失敗:", e)
finally:
    if 'conn' in locals():
        conn.close()
```

執行測試：

```bash
python test_db.py
```

若顯示 `✅ 資料庫連線成功` 即表示完成本章操作 🎉

---

## ✅ 本章結束

你已成功建立 PM2.5 資料表，並能透過 Python 連接資料庫。  
下一章我們將從政府開放資料 API 抓取即時 PM2.5 資料並寫入資料庫。


---

# 第三章：取得 PM2.5 開放資料並寫入資料庫

本章節將指導你如何透過政府提供的 PM2.5 開放資料 API，抓取即時資料並寫入 MySQL 資料庫。

---

## 🎯 教學目標

- 了解 PM2.5 開放資料 API
- 撰寫資料抓取與轉換程式
- 將資料寫入資料庫，避免重複寫入

---

## 1️⃣ 資料來源介紹

資料來源：**環保署空氣品質監測網**

API 範例（JSON 格式）：

```
https://data.moenv.gov.tw/api/v2/aqx_p_432?api_key=你的KEY&format=json
```

你需要先註冊帳號並取得 API Key。

---

## 2️⃣ 撰寫資料抓取程式

在 `pm25_utils.py` 中加入：

```python
import requests

def fetch_pm25_data(api_key):
    url = f"https://data.moenv.gov.tw/api/v2/aqx_p_432?api_key={api_key}&format=json"
    response = requests.get(url)
    if response.status_code == 200:
        raw = response.json()
        return raw["records"]
    else:
        raise Exception("API 錯誤：" + str(response.status_code))
```

---

## 3️⃣ 整理資料格式

新增一個整理轉換函式：

```python
from datetime import datetime

def transform_pm25_data(records):
    data_list = []
    for r in records:
        try:
            data_list.append((
                r["county"],
                r["sitename"],
                float(r["pm2.5"]) if r["pm2.5"] else None,
                datetime.strptime(r["datacreationdate"], "%Y-%m-%d %H:%M"),
                r["unit"]
            ))
        except Exception as e:
            print("資料轉換錯誤", e)
    return data_list
```

---

## 4️⃣ 寫入 MySQL 資料庫

在 `pm25_utils.py` 中再加入以下函式：

```python
from db_utils import open_db

def save_to_mysql(data_list):
    conn = open_db()
    cur = conn.cursor()

    sql = """
    INSERT INTO pm25_data (county, site, pm25, datacreationdate, unit)
    VALUES (%s, %s, %s, %s, %s)
    ON DUPLICATE KEY UPDATE pm25=VALUES(pm25)
    """

    cur.executemany(sql, data_list)
    conn.commit()
    conn.close()
    print(f"✅ 寫入 {len(data_list)} 筆資料")
```

---

## 5️⃣ 實際執行測試

新增 `update_pm25.py` 測試整合功能：

```python
from pm25_utils import fetch_pm25_data, transform_pm25_data, save_to_mysql

API_KEY = "請填入你的API Key"

records = fetch_pm25_data(API_KEY)
data_list = transform_pm25_data(records)
save_to_mysql(data_list)
```

執行：

```bash
python update_pm25.py
```

若成功寫入資料，表示整合完成 ✅

---

## ✅ 本章結束

你已能自動抓取 PM2.5 資料並寫入 MySQL 資料庫。  
下一章我們將開始建立前端網站，顯示最新資料與圖表！


---

# 第四章：建立前端網站與圖表展示

本章將教你如何使用 Flask 模板搭配 Chart.js 顯示 PM2.5 資料，建立互動式圖表與基本查詢介面。

---

## 🎯 教學目標

- 使用 Flask 顯示 HTML 網頁
- 整合 Chart.js 繪製即時 PM2.5 圖表
- 加入站點選單與後端資料查詢功能

---

## 1️⃣ 設定 Flask 模板資料夾

Flask 預設會從 `templates/` 讀取 HTML 檔案，請建立一個 `templates/index.html`：

```html
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <title>PM2.5 即時監測</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
  <h2>PM2.5 即時監測圖表</h2>
  <form method="GET" action="/">
    <label>選擇站點：
      <select name="site">
        {% for s in site_list %}
          <option value="{{s}}" {% if s == selected_site %}selected{% endif %}>{{s}}</option>
        {% endfor %}
      </select>
    </label>
    <button type="submit">查詢</button>
  </form>

  <canvas id="pm25Chart" width="600" height="300"></canvas>

  <script>
    const ctx = document.getElementById('pm25Chart').getContext('2d');
    const chart = new Chart(ctx, {
      type: 'line',
      data: {
        labels: {{ labels | safe }},
        datasets: [{
          label: 'PM2.5',
          data: {{ values | safe }},
          borderWidth: 2,
          borderColor: 'rgba(75, 192, 192, 1)',
          fill: false
        }]
      },
      options: {
        scales: {
          x: { display: true },
          y: { beginAtZero: true }
        }
      }
    });
  </script>
</body>
</html>
```

---

## 2️⃣ 修改 app.py 提供資料與模板

在 `app.py` 中修改如下：

```python
from flask import Flask, render_template, request
from db_utils import open_db

app = Flask(__name__)

@app.route('/')
def index():
    site = request.args.get("site", default="中壢")

    conn = open_db()
    cur = conn.cursor()
    cur.execute("SELECT DISTINCT site FROM pm25_data ORDER BY site")
    site_list = [r[0] for r in cur.fetchall()]

    cur.execute("""
        SELECT datacreationdate, pm25
        FROM pm25_data
        WHERE site=%s
        ORDER BY datacreationdate DESC
        LIMIT 20
    """, (site,))
    rows = cur.fetchall()
    conn.close()

    labels = [r[0].strftime("%H:%M") for r in rows][::-1]
    values = [r[1] for r in rows][::-1]

    return render_template("index.html",
                           site_list=site_list,
                           selected_site=site,
                           labels=labels,
                           values=values)

if __name__ == '__main__':
    app.run(debug=True)
```

---

## 3️⃣ 啟動網站

執行：

```bash
python app.py
```

打開瀏覽器進入 `http://127.0.0.1:5000/`，選擇站點後即可查看 PM2.5 折線圖。

---

## ✅ 本章結束

你已建立互動式圖表並能查詢歷史 PM2.5 資料，下一章將進一步設計自動排程與 GitHub Actions 自動更新功能。


---

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


---

# 第六章：部署 Flask PM2.5 專案到線上

本章將教你如何將 PM2.5 Flask 專案部署到線上平台，例如 Render、PythonAnywhere 或 Railway，讓其他人能透過瀏覽器看到你的圖表。

---

## 🎯 教學目標

- 了解部署的基本觀念
- 建立 `Procfile` 與 `requirements.txt`
- 使用 Render 部署 Flask 專案
- 注意部署時的環境變數與資料庫連線設定

---

## 1️⃣ 準備部署檔案

在專案根目錄新增以下兩個檔案：

### 📄 `requirements.txt`

如果你之前沒有建立過，請重新產生一次：

```bash
pip freeze > requirements.txt
```

### 📄 `Procfile`

這是告訴 Render 如何啟動你的專案的檔案，內容如下（注意無副檔名）：

```
web: python app.py
```

---

## 2️⃣ 設定 `app.py` 可辨識埠號（Render 需要）

修改 `app.run(...)` 為以下格式：

```python
import os

if __name__ == '__main__':
    port = int(os.environ.get("PORT", 5000))
    app.run(host='0.0.0.0', port=port)
```

---

## 3️⃣ 將程式碼上傳 GitHub

Render 使用 GitHub 倉庫部署，請將你的專案 Push 上 GitHub：

```bash
git init
git add .
git commit -m "initial commit"
git remote add origin https://github.com/你的帳號/pm25-flask.git
git push -u origin master
```

---

## 4️⃣ 使用 Render 部署步驟

1. 前往 [https://render.com/](https://render.com/)
2. 點選「New」→「Web Service」
3. 選擇 GitHub 倉庫並授權
4. 設定如下：

   | 項目            | 設定值              |
   |-----------------|---------------------|
   | Environment     | Python              |
   | Build Command   | `pip install -r requirements.txt` |
   | Start Command   | `python app.py`     |
   | Branch          | `master`            |
   | Region          | 選 Taiwan 附近區域（美西或亞洲）|

5. 若你的專案中使用了 API_KEY 或資料庫帳密，可在「環境變數」中設定：

   - `API_KEY=你的PM2.5金鑰`
   - `DB_HOST`、`DB_USER`、`DB_PASSWORD` 等

---

## 5️⃣ 使用雲端資料庫（選用）

Render 可整合 PostgreSQL，如要繼續使用本地 MySQL，建議改用 Railway + ClearDB、PlanetScale 或其他支援 MySQL 的雲端平台。

---

## ✅ 本章結束

你已學會將專案部署到 Render，未來也可以嘗試部署到其他平台如 Railway 或 PythonAnywhere，甚至接入自己的網域名稱！

下一步可考慮：
- 美化前端 UI
- 增加圖表類型（趨勢分析、區間比較）
- 加入使用者登入／資料上傳功能


---

# 第七章：功能擴充與進階應用

恭喜你完成了 Flask PM2.5 專案的基礎開發與部署！  
本章將帶你思考與實作進階功能，讓這個專案更具互動性與應用價值。

---

## 🎯 教學目標

- 思考實用的擴充功能
- 加入歷史資料查詢與時間範圍篩選
- 支援多種感測數據（PM10、O3、CO2）
- 增加圖表類型與分析介面

---

## 1️⃣ 加入時間區間篩選功能

在 `app.py` 中接收 `start` 與 `end` GET 參數，修改查詢語法如下：

```python
from flask import request
from datetime import datetime

start = request.args.get("start")
end = request.args.get("end")

sql = "SELECT datacreationdate, pm25 FROM pm25_data WHERE site=%s"
params = [site]

if start and end:
    sql += " AND datacreationdate BETWEEN %s AND %s"
    params.extend([start, end])

sql += " ORDER BY datacreationdate DESC LIMIT 100"
cur.execute(sql, tuple(params))
```

---

## 2️⃣ 表單新增日期篩選欄位（HTML）

```html
<label>起始時間：<input type="datetime-local" name="start" value="{{start}}"></label>
<label>結束時間：<input type="datetime-local" name="end" value="{{end}}"></label>
```

搭配 Chart.js，讓使用者查詢特定區段的 PM2.5 變化趨勢。

---

## 3️⃣ 支援更多環境數據欄位

你可以修改資料表結構，加入 PM10、O3、CO 等欄位：

```sql
ALTER TABLE pm25_data ADD pm10 FLOAT, ADD o3 FLOAT, ADD co FLOAT;
```

並在抓取與儲存時同步更新這些欄位：

```python
r.get("pm10"), r.get("o3"), r.get("co")
```

---

## 4️⃣ 增加圖表類型與比較模式

- 加入 **條狀圖** 比較不同站點 PM2.5 平均值
- 加入 **週期圖** 呈現一天內的高峰與低谷時間
- 可考慮搭配前端框架（如 Vue.js）或資料視覺化工具（如 D3.js）

---

## 5️⃣ 可考慮的進階功能

| 功能                 | 說明 |
|----------------------|------|
| 使用者登入系統       | 建立個人儀表板 |
| 導入地圖顯示          | 使用 Leaflet.js 顯示地理位置 |
| 匯出報表             | 匯出 Excel 或 PDF 報表 |
| LINE Notify 警示     | 空氣品質惡化時自動發送提醒 |
| 後台資料管理介面     | 利用 Flask-Admin 提供資料查看與修改功能 |

---

## ✅ 本章結束

你已掌握了專案的擴充方向，這些功能可以依照實際需求自由組合與實作，進一步打造成一個實用的環境監測系統。

---

## 📚 延伸學習建議

- Flask Blueprint 模組化
- SQLAlchemy ORM 使用
- Docker 容器化部署
- 前後端分離（Vue / React）


---

