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
