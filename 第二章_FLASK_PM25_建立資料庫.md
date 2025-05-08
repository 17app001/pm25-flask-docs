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
