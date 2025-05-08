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
