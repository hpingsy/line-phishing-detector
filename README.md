# 🔍 LINE 偽冒下載網站偵測工具

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YOUR_USERNAME/line-phishing-detector/blob/main/LINE_釣魚黑名單_Colab.ipynb)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Python 3.8+](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)

> 自動模擬使用者在 Google 搜尋「LINE 下載」，偵測搜尋結果中的偽冒官網，輸出可直接匯入防火牆的黑名單。

---

## 📋 目錄

- [功能特色](#-功能特色)
- [運作原理](#-運作原理)
- [快速開始](#-快速開始)
- [輸出結果](#-輸出結果)
- [執行成果範例](#-執行成果範例)
- [部署黑名單](#-部署黑名單)
- [自訂設定](#-自訂設定)
- [注意事項](#-注意事項)

---

## ✨ 功能特色

| 功能 | 說明 |
|------|------|
| 🤖 真實瀏覽器模擬 | 使用 Selenium Chrome，完整模擬人類搜尋行為，避免被 Google 限速 |
| 🔍 多組關鍵字掃描 | 7 組搜尋關鍵字，涵蓋不同使用者搜尋習慣 |
| 🎯 智慧過濾 | 自動排除 line.me、App Store、Google Play 等官方合法來源 |
| 🈶 簡中偵測 | 自動識別頁面是否含簡體中文（高風險偽冒特徵） |
| 📊 多格式輸出 | 同時產生 TXT / CSV / HTML 報告，可直接匯入各類防火牆 |
| 🚩 風險標記 | 自動標記 `.cn 網域`、`仿冒登入頁`、`假下載連結`等高風險特徵 |

---

## 🔄 運作原理

```
模擬 Google 搜尋「LINE 下載」等 7 組關鍵字
            ↓
  擷取每組前 20 筆搜尋結果（共約 90+ 筆）
            ↓
  白名單過濾：排除官方網域 & 合法平台
            ↓
  偽冒特徵偵測：網域含「line」字眼 → 可疑
            ↓
  實際訪問網站：抓取頁面標題 & 內文
            ↓
  分析特徵：簡中文字 / .cn網域 / 仿冒結構
            ↓
  輸出黑名單（TXT / CSV / HTML 報告）
```

---

## 🚀 快速開始

### 方法一：Google Colab（推薦，零安裝）

1. 點擊上方 [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YOUR_USERNAME/line-phishing-detector/blob/main/LINE_釣魚黑名單_Colab.ipynb) 徽章
2. 點選「執行階段」→「全部執行」
3. 等待完成後，Cell 6 會自動下載 ZIP 壓縮包

### 方法二：本機執行

```bash
# 1. Clone 專案
git clone https://github.com/YOUR_USERNAME/line-phishing-detector.git
cd line-phishing-detector

# 2. 安裝依賴
pip install -r requirements.txt

# 3. 安裝 Chrome（Linux）
wget -q https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install ./google-chrome-stable_current_amd64.deb

# 4. 執行
python line_phishing_detector.py
```

---

## 📁 輸出結果

執行完成後，在 `/content/line_phishing_output/`（Colab）或 `./output/`（本機）產生以下檔案：

```
line_phishing_output/
├── blacklist_domains.txt   # 純網域清單，可匯入 Pi-hole / pfSense
├── blacklist_urls.txt      # 完整 URL 清單，可匯入 Proxy / WAF
├── blacklist_report.csv    # 詳細報告，可用 Excel 開啟
└── blacklist_report.html   # 視覺化 HTML 報告（深色主題）
```

### HTML 報告預覽

報告包含以下欄位：

| 欄位 | 說明 |
|------|------|
| 偽冒網域 | 偵測到的可疑網域 |
| 頁面標題 | 偽冒站顯示的標題 |
| URL | 完整網址 |
| 搜尋關鍵字 | 透過哪個關鍵字找到 |
| 排名 | Google 搜尋排名位置 |
| 簡中命中字 | 命中的簡體中文特有字 |
| 特徵標記 | `.cn網域` / `仿冒登入` / `假下載` 等 |

---

## 📊 執行成果範例

> 實際執行結果（2026年3月）：

| 統計項目 | 數量 |
|---------|------|
| Google 搜尋結果總計 | 91 筆 |
| 確認官方正版網域 | 16 個 |
| 非官方網域（已審查） | 59 個 |
| **偵測到的偽冒站** | **16 個** |

**偵測到的部分偽冒站：**

| 偽冒網域 | Google 排名 | 風險特徵 |
|---------|------------|---------|
| `linebbn.com` | #6 | 仿冒官網結構 |
| `line-webs.com` | #9 | 含簡中、仿冒下載頁 |
| `line-zh.net` | #12 | 含簡中、仿冒登入 |
| `line.softonic.cn` | #17 | .cn 網域、含簡中 |
| `line-me.us` | #20 | 含簡中、仿冒官網 |

---

## 🛡 部署黑名單

### Pi-hole（DNS 封鎖）
```bash
cp blacklist_domains.txt /etc/pihole/custom.list
pihole restartdns
```

### Squid Proxy
```
acl line_phishing dstdomain "/etc/squid/blacklist_domains.txt"
http_access deny line_phishing
```

### pfSense / OPNsense
`Firewall` → `Aliases` → `URLs` → 貼入 `blacklist_domains.txt` 網址

### Windows Defender ATP
`Security Center` → `Indicators` → `URLs/Domains` → 批次匯入 CSV

---

## ⚙ 自訂設定

在 **Cell 2** 修改以下變數：

```python
# 搜尋關鍵字（可自行增加）
SEARCH_QUERIES = [
    'LINE 下載',
    'LINE 電腦版 下載',
    # 加入更多關鍵字...
]

# 每組關鍵字取幾筆結果
RESULTS_PER_QUERY = 20

# 官方白名單（可新增信任網域）
OFFICIAL_DOMAINS = {
    'line.me', 'linecorp.com',
    # 新增...
}
```

---

## ⚠ 注意事項

- 本工具**僅收集公開搜尋結果**，不主動掃描或攻擊任何網站
- 建議每週執行一次，保持黑名單更新
- Google 可能偶爾限速，遇到時稍等後重試即可
- 輸出結果建議人工審閱後再匯入防火牆
- 如發現詐騙網站，請同步通報 [165 全民防騙網](https://165.npa.gov.tw) 或 [TWCERT/CC](https://www.twcert.org.tw)

---

## 📦 依賴套件

```
requests
beautifulsoup4
lxml
selenium
webdriver-manager
```

---

## 📄 License

MIT License © 2026

---

## 🙏 資料來源

- [PhishTank](https://www.phishtank.com/) - 全球釣魚資料庫
- [OpenPhish](https://openphish.com/) - 即時釣魚 Feed
- [urlscan.io](https://urlscan.io/) - 網站掃描結果
- [165 全民防騙網](https://165.npa.gov.tw) - 台灣警政署
- [TWCERT/CC](https://www.twcert.org.tw) - 台灣 CERT

> ⚠ **免責聲明**：本工具僅供企業內部資安防護研究使用。如遭詐騙請撥打 **165 反詐騙專線**。
