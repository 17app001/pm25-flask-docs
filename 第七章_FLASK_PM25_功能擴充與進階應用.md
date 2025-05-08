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
