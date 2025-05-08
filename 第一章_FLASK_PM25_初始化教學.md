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
