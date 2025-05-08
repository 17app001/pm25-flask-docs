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
