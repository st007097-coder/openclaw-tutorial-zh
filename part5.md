## 第十六章：Web 控制台

### 16.1 Control UI 介紹

OpenClaw 內置一個 Web 控制台（Control UI），你可以用瀏覽器管理一切。

#### 打開控制台

喺瀏覽器入面輸入：
```
http://127.0.0.1:18789/
```

如果你有設定密碼，會要求登入。

#### 控制台 Tabs

| Tab | 功能 |
|-----|------|
| **Chat** | 同 Agent 聊天 |
| **Config** | 編輯設定 |
| **Sessions** | 管理 Session |
| **Nodes** | 管理 Node 裝置 |

### 16.2 聊天介面

**Chat Tab** 係最基本嘅功能，你可以直接喺瀏覽器度同 Agent 對話。

功能：
- 即時對話（支援 streaming）
- 上傳檔案/圖片
- 查看對話歷史
- 切換不同 Session

### 16.3 設定管理

**Config Tab** 提供兩種編輯模式：

#### 表單模式（Form）

適合新手，用 GUI 表單改設定，唔使懂 JSON。

#### Raw JSON Editor

適合進階用戶，直接編輯 `openclaw.json`：

```json
{
  "gateway": {
    "port": 18789,
    "auth": {
      "token": "my-token"
    }
  },
  "agents": {
    "list": [
      {
        "name": "default",
        "model": "gpt-4o"
      }
    ]
  }
}
```

改完之後按 **Save**，Config 會自動 hot reload。

### 16.4 Session 管理

**Sessions Tab** 可以管理所有活躍嘅 Session：

- 查看 Session 列表
- 查看每個 Session 嘅對話歷史
- 刪除 Session
- 查看 Session 狀態（活躍/閒置）

### 📝 第十六章小結

- **Control UI** 喺 `http://127.0.0.1:18789/`
- 四個主要 Tab：Chat、Config、Sessions、Nodes
- Config 支援表單模式同 Raw JSON 編輯
- 可以喺 Web 入面管理 Session 同 Node

---

## 第十七章：進階配置

### 17.1 多 Gateway 配置

同一部機器可以跑多個 Gateway 實例，例如分開測試同生產環境。

#### 方法：用唔同 port 同 config

```bash
# 第一個實例（默認）
openclaw gateway start --config ~/.openclaw/openclaw.json --port 18789

# 第二個實例
openclaw gateway start --config ~/.openclaw/openclaw-staging.json --port 18790
```

#### 用環境變數區分

```bash
OPENCLAW_CONFIG=~/.openclaw/openclaw-staging.json OPENCLAW_PORT=18790 openclaw gateway start
```

### 17.2 遠端存取（Tailscale、SSH）

如果你部 Gateway 喺 VPS 或者屋企部機度跑，你需要遠端存取。

#### 方法一：Tailscale（推薦）

Tailscale 係最簡單最安全嘅方法：

```bash
# 安裝 Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# 登入
sudo tailscale up

# 之後用 Tailscale IP 存取
# http://100.x.x.x:18789/
```

**好處：**
- 唔使開 port 喺公網
- 加密連接
- 免費 plan 夠用

#### 方法二：SSH Tunnel

如果唔想用 Tailscale，可以用 SSH tunnel：

```bash
# 喺你本地電腦執行
ssh -N -L 18789:127.0.0.1:18789 user@your-server-ip

# 之後喺瀏覽器入面開
# http://127.0.0.1:18789/
```

`-N` = 唔執行遠端命令
`-L 18789:127.0.0.1:18789` = 將本地 18789 port 轉發到遠端 18789

### 17.3 自訂 System Prompt

你可以為每個 Agent 自訂 System Prompt：

```json
{
  "agents": {
    "list": [
      {
        "name": "default",
        "model": "gpt-4o",
        "systemPrompt": "你係一個專業嘅香港 QS 助手。用廣東話回答問題，熟悉建築工程同工料測量。"
      },
      {
        "name": "coder",
        "model": "claude-sonnet-4-20250514",
        "systemPrompt": "你係一個全棧開發專家。寫 clean code，注重 best practices。"
      }
    ]
  }
}
```

#### System Prompt 最佳實踐

- 保持簡潔，唔好太長
- 明確指定語言同風格
- 加入具體領域知識
- 定期根據使用效果調整

### 17.4 Provider Failover

設定備用模型，主模型 fail 咗就自動切換：

```json
{
  "model": {
    "primary": "gpt-4o",
    "fallbacks": ["claude-sonnet-4-20250514", "gemini-2.5-pro"]
  }
}
```

#### 運作原理

```
請求 → gpt-4o
         ↓ （fail）
       claude-sonnet-4-20250514
         ↓ （fail）
       gemini-2.5-pro
         ↓ （全部 fail）
       返回錯誤
```

### 17.5 Queue 與 Rate Limit

#### Queue 模式

當多個訊息同時到達時，控制處理方式：

```json
{
  "queue": {
    "mode": "steer"
  }
}
```

| 模式 | 說明 |
|------|------|
| `steer` | 新訊息可以「插隊」，打斷目前處理（默認） |
| `followup` | 新訊息排隊，等目前處理完成 |
| `collect` | 收集一段時間嘅訊息，合併處理 |

#### Block Streaming

分段發送長回覆，避免一次性發送太長內容：

```json
{
  "blockStreaming": {
    "enabled": true,
    "maxChars": 2000
  }
}
```

### 17.6 Config Hot Reload

控制配置變更後嘅 reload 方式：

```json
{
  "configReload": "hybrid"
}
```

| 模式 | 說明 |
|------|------|
| `hybrid` | 能 hot reload 嘅就 hot reload，唔得就 restart（默認） |
| `hot` | 全部 hot reload（可能有風險） |
| `restart` | 全部 restart（最安全但會斷連） |
| `off` | 唔自動 reload |

#### $include：拆分 Config

大型 config 可以用 `$include` 拆分成多個文件：

**主文件 `openclaw.json`：**
```json
{
  "$include": ["./config/gateway.json", "./config/agents.json", "./config/channels.json"]
}
```

**`config/gateway.json`：**
```json
{
  "gateway": {
    "port": 18789,
    "auth": {
      "token": "my-token"
    }
  }
}
```

#### Config RPC

程式化修改配置：

```json
// config.apply — 完整替換配置
{
  "method": "config.apply",
  "params": { "config": { ... } }
}

// config.patch — 部分更新配置
{
  "method": "config.patch",
  "params": { "patch": { "gateway": { "port": 18790 } } }
}
```

### 📝 第十七章小結

- **多 Gateway**：用唔同 port + config 跑多個實例
- **遠端存取**：Tailscale（推薦）或 SSH Tunnel
- **自訂 System Prompt**：為每個 Agent 定制個性
- **Provider Failover**：主模型 fail 自動切換備用
- **Queue modes**：控制多訊息處理方式
- **Config Hot Reload**：hybrid 模式最平衡
- **$include**：大型 config 拆分管理

---

## 第十八章：部署到伺服器

### 18.1 VPS 部署

將 OpenClaw 部署到 VPS（Virtual Private Server），24/7 運行。

#### 推薦 VPS 供應商

| 供應商 | 最低配置 | 價格（月） | 推薦度 |
|--------|----------|-----------|--------|
| DigitalOcean | 1 vCPU / 1GB | ~$6 | ⭐⭐⭐⭐ |
| Hetzner | 2 vCPU / 4GB | ~€4 | ⭐⭐⭐⭐⭐ |
| Oracle Cloud Free | 4 vCPU / 24GB | 免費 | ⭐⭐⭐⭐ |
| AWS Lightsail | 1 vCPU / 1GB | ~$3.5 | ⭐⭐⭐ |
| Vultr | 1 vCPU / 1GB | ~$5 | ⭐⭐⭐ |

#### 安裝步驟

```bash
# 1. SSH 入去部 VPS
ssh root@your-vps-ip

# 2. 安裝 Node.js（如果未有）
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo bash -
sudo apt install -y nodejs

# 3. 安裝 OpenClaw
sudo npm install -g openclaw

# 4. 初始化
openclaw init

# 5. 編輯設定
nano ~/.openclaw/openclaw.json

# 6. 啟動 Gateway
openclaw gateway start
```

#### 用 systemd 管理服務

建立 service file：

```bash
sudo tee /etc/systemd/system/openclaw.service << 'EOF'
[Unit]
Description=OpenClaw Gateway
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/.openclaw
ExecStart=/usr/bin/openclaw gateway start
Restart=always
RestartSec=5
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
EOF

# 啟用並啟動
sudo systemctl daemon-reload
sudo systemctl enable openclaw
sudo systemctl start openclaw

# 查看狀態
sudo systemctl status openclaw

# 查看日誌
sudo journalctl -u openclaw -f
```

### 18.2 Docker 部署

用 Docker 部署最簡單，環境一致性最好。

#### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  openclaw:
    image: openclaw/openclaw:latest
    container_name: openclaw-gateway
    restart: unless-stopped
    ports:
      - "18789:18789"
    volumes:
      - openclaw-data:/root/.openclaw
    environment:
      - NODE_ENV=production
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:18789/health"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  openclaw-data:
```

#### 啟動

```bash
# 啟動
docker-compose up -d

# 查看日誌
docker-compose logs -f

# 停止
docker-compose down

# 更新
docker-compose pull
docker-compose up -d
```

#### 單容器方式

```bash
docker run -d \
  --name openclaw \
  -p 18789:18789 \
  -v openclaw-data:/root/.openclaw \
  --restart unless-stopped \
  openclaw/openclaw:latest
```

### 18.3 雲端部署

#### 部署到雲端平台

**Railway：**
```bash
# 安裝 Railway CLI
npm install -g @railway/cli

# 登入
railway login

# 部署
railway init
railway up
```

**Fly.io：**
```bash
# 安裝 flyctl
curl -L https://fly.io/install.sh | sh

# 登入
fly auth launch

# 部署
fly launch
fly deploy
```

### 18.4 Raspberry Pi

OpenClaw 支援 ARM 架構，可以喺 Raspberry Pi 上運行。

#### 安裝步驟

```bash
# 1. 確保系統更新
sudo apt update && sudo apt upgrade -y

# 2. 安裝 Node.js（ARM 版本）
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo bash -
sudo apt install -y nodejs

# 3. 驗證
node -v
npm -v

# 4. 安裝 OpenClaw
sudo npm install -g openclaw

# 5. 初始化同設定
openclaw init
nano ~/.openclaw/openclaw.json

# 6. 啟動
openclaw gateway start
```

#### Pi 專用建議

- 用 **Raspberry Pi 4** 或以上（2GB+ RAM）
- 用 SSD 而唔係 SD 卡（性能差好遠）
- 設定 swap（至少 2GB）：`sudo dphys-swapfile swapoff && sudo nano /etc/dphys-swapfile && sudo dphys-swapfile setup && sudo dphys-swapfile swapon`
- 用 Tailscale 遠端存取

### 📝 第十八章小結

- **VPS**：DigitalOcean / Hetzner / Oracle Cloud Free 推薦
- **systemd**：Linux 服務管理，自動重啟
- **Docker**：環境一致性最佳，docker-compose 管理
- **Raspberry Pi**：支援 ARM，用 SSD + 足夠 RAM
- **雲端平台**：Railway / Fly.io 一鍵部署

---

## 第十九章：疑難排解

### 19.1 常見問題 FAQ

#### ❌ Gateway 起唔到

**症狀：** `openclaw gateway start` 報錯或無反應

**解決步驟：**
```bash
# 1. 先跑 doctor 診斷
openclaw doctor

# 2. 檢查 config 語法
openclaw config validate

# 3. 查看錯誤日誌
openclaw logs --lines 50

# 4. 試吓自動修復
openclaw doctor --fix

# 5. 如果都唔得，重新初始化
openclaw init --force
```

#### ❌ WhatsApp 斷線

**症狀：** WhatsApp 消息收唔到

**解決步驟：**
```bash
# 重新連接
openclaw channels login whatsapp

# 檢查狀態
openclaw channels status --probe

# 如果 QR code 過期，重新生成
openclaw channels login whatsapp --refresh
```

#### ❌ Telegram poll 失敗

**症狀：** Telegram 收唔到消息

**解決步驟：**
```bash
# 1. 檢查 DNS
nslookup api.telegram.org

# 2. 如果用緊 IPv6，切去 IPv4
# 喺 openclaw.json 度加：
# "telegram": { "ipv6": false }

# 3. 如果喺大陸或者受限網絡，設置 proxy
# "telegram": { "proxy": "socks5://127.0.0.1:1080" }

# 4. 重啟
openclaw gateway restart
```

#### ❌ Discord 看唔到群組消息

**症狀：** Bot 收到 DM 但收唔到群組消息

**解決步驟：**
```bash
# 1. 檢查 Discord intents
# 喺 Discord Developer Portal 入面確認開啟：
# - SERVER MEMBERS INTENT
# - MESSAGE CONTENT INTENT

# 2. 檢查 groupPolicy
# openclaw.json 入面確認：
# "discord": { "groupPolicy": "allow" }

# 3. 重新邀請 bot（確保權限正確）
```

#### ❌ 高 token 消耗

**症狀：** API 費用好貴

**解決方法：**
```json
{
  "agents": {
    "list": [{
      "name": "default",
      "isolatedSession": true,
      "lightContext": true
    }]
  }
}
```

- `isolatedSession`：每個 session 獨立 context，唔會累積舊對話
- `lightContext`：減少 context 窗口大小

### 19.2 Doctor 診斷工具

OpenClaw 內置診斷工具，幫你快速搵出問題。

#### 基本診斷

```bash
openclaw doctor
```

輸出範例：
```
🔍 OpenClaw Doctor

✅ Node.js version: v22.22.2
✅ OpenClaw version: latest
✅ Config file: valid
✅ Gateway: running on port 18789
⚠️ WhatsApp: disconnected (re-link required)
✅ Telegram: connected
✅ Discord: connected
❌ Tailscale: not installed

💡 Suggestions:
  - Run `openclaw channels login whatsapp` to reconnect
  - Consider installing Tailscale for remote access
```

#### 自動修復

```bash
openclaw doctor --fix
```

會自動嘗試修復可以修復嘅問題。

### 19.3 日誌查看

#### 查看最新日誌

```bash
openclaw logs
```

#### 即時跟蹤日誌

```bash
openclaw logs --follow
# 或
openclaw logs -f
```

#### 查睇指定行數

```bash
openclaw logs --lines 100
```

#### 查看整體狀態

```bash
openclaw status
```

輸出範例：
```
📊 OpenClaw Status

Gateway: running (port 18789)
Uptime: 3d 5h 23m
Sessions: 12 active
Channels: 3 connected (Telegram, Discord, WhatsApp)
Nodes: 2 paired
Model: gpt-4o (primary) | claude-sonnet-4-20250514 (fallback)
```

#### 頻道狀態

```bash
openclaw channels status --probe
```

### 📝 第十九章小結

- **Doctor**：`openclaw doctor` 診斷 + `--fix` 自動修復
- **日誌**：`openclaw logs -f` 即時跟蹤
- **常見問題**：config 驗證、重新連接、DNS/IPv6 檢查
- **Token 爆煲**：用 `isolatedSession` + `lightContext` 減少消耗

---

## 第二十章：實戰案例

### 20.1 個人 AI 助手

最常見嘅用途——將 OpenClaw 變成你嘅私人 AI 助手。

#### 配置範例

```json
{
  "agents": {
    "list": [{
      "name": "personal",
      "model": "gpt-4o",
      "systemPrompt": "你係一個貼心嘅私人助手。用廣東話溝通。幫手管理日程、回覆郵件、搜尋資料。簡潔直接，唔好廢話。"
    }]
  },
  "channels": {
    "telegram": {
      "token": "YOUR_TELEGRAM_BOT_TOKEN"
    }
  }
}
```

#### 日常使用場景

- 📅 **日程管理：** 「聽日有咩會議？」
- 📧 **郵件摘要：** 「有冇重要郵件？」
- 🔍 **資料搜尋：** 「幫我查吓 XXX 嘅最新消息」
- 🌤️ **天氣預報：** 「聽日會唔會落雨？」
- 📝 **筆記整理：** 「幫我整理呢段會議記錄」

### 20.2 開發者工作流

開發者可以用 OpenClaw 做 code review、自動化測試等。

#### 配置範例

```json
{
  "agents": {
    "list": [{
      "name": "dev",
      "model": "claude-sonnet-4-20250514",
      "systemPrompt": "你係一個 senior full-stack developer。寫 clean code，注重 best practices。用繁體中文解釋技術概念。"
    }]
  }
}
```

#### 使用場景

- 🔍 **Code Review：** 直接貼 code，幫你 review
- 🐛 **Debug：** 「呢段 code 有 bug，幫我睇吓」
- 📖 **文檔生成：** 「幫呢個 function 寫 docstring」
- 🧪 **測試：** 「幫呢個 module 寫 unit test」
- 🔄 **Git 操作：** 「幫我建立一個 feature branch」

### 20.3 團隊協作

OpenClaw 可以喺 Discord / Slack 群組入面做團隊助手。

#### 配置範例

```json
{
  "channels": {
    "discord": {
      "token": "YOUR_DISCORD_BOT_TOKEN",
      "groupPolicy": "allow"
    }
  },
  "agents": {
    "list": [{
      "name": "team",
      "model": "gpt-4o",
      "systemPrompt": "你係團隊嘅 AI 助手。喺群組入面唔好太頻繁回應。只喺有人 @你 或者你覺得可以幫到手嘅時候先講嘢。"
    }]
  }
}
```

#### 使用場景

- 📋 **會議記錄：** 「阿星，幫手整理今日會議重點」
- ❓ **FAQ 回答：** 新成員問嘅常見問題
- 📊 **數據查詢：** 「呢個項目嘅進度點？」
- 🔔 **通知轉發：** CI/CD 失敗、PR 需要 review 等

### 20.4 自動化監控

用 OpenClaw 做定時檢查同監控。

#### 配置範例

```json
{
  "cron": [
    {
      "schedule": "0 */6 * * *",
      "prompt": "檢查伺服器狀態同磁碟使用量，如果有超過 80% 就 send 警告",
      "channel": "telegram",
      "target": "YOUR_CHAT_ID"
    },
    {
      "schedule": "0 9 * * 1",
      "prompt": "整理上週嘅 Git commits，生成週報",
      "channel": "telegram",
      "target": "YOUR_CHAT_ID"
    }
  ]
}
```

#### 使用場景

- 🖥️ **伺服器監控：** CPU、RAM、Disk 使用量
- 📈 **網站監控：** 網站係咪正常運行
- 💰 **費用追蹤：** AWS/GCP 月結單
- 📰 **新聞摘要：** 每日行業新聞摘要
- 🔄 **備份檢查：** 確認備份係咪成功

### 📝 第二十章小結

- **個人助手**：日程、郵件、搜尋、天氣
- **開發者工作流**：Code Review、Debug、文檔生成
- **團隊協作**：Discord/Slack 群組助手
- **自動化監控**：Cron 定時執行、伺服器監控

---

## 結語

恭喜你！睇到呢度，你已經掌握咗 OpenClaw 嘅核心功能。

### 🚀 進階學習資源

| 資源 | 連結 |
|------|------|
| 官方文檔 | https://docs.openclaw.ai |
| Discord 社群 | https://discord.com/invite/clawd |
| Skills 市場 | https://clawhub.com |
| GitHub | https://github.com/openclaw/openclaw |

### 🤝 社群支援

加入 **Discord 社群**（https://discord.com/invite/clawd），同其他用戶交流心得、分享 Skills、搵人幫手。

### 🛠️ 貢獻 OpenClaw

OpenClaw 係開源項目，歡迎貢獻：
- 🐛 回報 Bug
- 💡 提出新功能
- 📝 改善文檔
- 🧩 開發 Skills 分享到 ClawHub
- ⭐ GitHub 上加 Star

### 下一步

1. **完善你嘅設定**：根據自己需求調整 config
2. **探索 Skills**：喺 ClawHub 揾啱你用嘅 Skills
3. **加入社群**：Discord 入面有好多高手
4. **分享經驗**：將你嘅用法同心得分享畀其他人

> OpenClaw 嘅可能性近乎無限。發揮你嘅創意，打造屬於你自己嘅 AI 助手吧！ ⭐

---

## 附錄

### A. 常用 CLI 命令速查表

#### 基本操作

| 命令 | 說明 |
|------|------|
| `openclaw init` | 初始化 OpenClaw |
| `openclaw gateway start` | 啟動 Gateway |
| `openclaw gateway stop` | 停止 Gateway |
| `openclaw gateway restart` | 重啟 Gateway |
| `openclaw gateway status` | 查看 Gateway 狀態 |
| `openclaw status` | 整體狀態概覽 |

#### 診斷與日誌

| 命令 | 說明 |
|------|------|
| `openclaw doctor` | 診斷工具 |
| `openclaw doctor --fix` | 自動修復 |
| `openclaw logs` | 查看日誌 |
| `openclaw logs --follow` | 即時跟蹤日誌 |
| `openclaw logs --lines 100` | 查看最近 100 行 |

#### 頻道管理

| 命令 | 說明 |
|------|------|
| `openclaw channels list` | 列出頻道 |
| `openclaw channels login <channel>` | 登入頻道 |
| `openclaw channels status --probe` | 頻道狀態檢查 |

#### Node 管理

| 命令 | 說明 |
|------|------|
| `openclaw nodes list` | 列出已配對 Node |
| `openclaw nodes approve <id>` | 批准配對 |
| `openclaw nodes revoke <id>` | 撤銷配對 |

#### 安全

| 命令 | 說明 |
|------|------|
| `openclaw security audit` | 安全審計 |
| `openclaw gateway token rotate` | 更換 Token |
| `openclaw config validate` | 驗證配置 |

### B. openclaw.json 配置範例

#### 最小配置

```json
{
  "gateway": {
    "port": 18789,
    "auth": {
      "token": "your-secret-token"
    }
  },
  "agents": {
    "list": [{
      "name": "default",
      "model": "gpt-4o"
    }]
  },
  "channels": {
    "telegram": {
      "token": "YOUR_TELEGRAM_BOT_TOKEN"
    }
  }
}
```

#### 完整生產配置

```json
{
  "gateway": {
    "port": 18789,
    "auth": {
      "token": "gw_strong_token_here_32chars_min"
    },
    "lock": true
  },
  "agents": {
    "list": [{
      "name": "default",
      "model": "gpt-4o",
      "systemPrompt": "你係一個有幫助嘅 AI 助手。",
      "isolatedSession": true,
      "lightContext": false
    }]
  },
  "model": {
    "primary": "gpt-4o",
    "fallbacks": ["claude-sonnet-4-20250514", "gemini-2.5-pro"]
  },
  "channels": {
    "telegram": {
      "token": "YOUR_TELEGRAM_BOT_TOKEN"
    },
    "discord": {
      "token": "YOUR_DISCORD_BOT_TOKEN",
      "groupPolicy": "allow"
    }
  },
  "sandbox": {
    "enabled": true,
    "mode": "non-main",
    "scope": "session"
  },
  "tools": {
    "allow": ["read", "write", "exec", "web_search", "web_fetch", "browser", "message"],
    "deny": []
  },
  "allowFrom": [
    "telegram:YOUR_CHAT_ID"
  ],
  "dmPolicy": "allowlist",
  "configReload": "hybrid",
  "queue": {
    "mode": "steer"
  }
}
```

### C. 環境變數列表

| 環境變數 | 說明 | 默認值 |
|----------|------|--------|
| `OPENCLAW_CONFIG` | Config 文件路徑 | `~/.openclaw/openclaw.json` |
| `OPENCLAW_PORT` | Gateway port | `18789` |
| `OPENCLAW_HOME` | OpenClaw home 目錄 | `~/.openclaw` |
| `OPENAI_API_KEY` | OpenAI API Key | — |
| `ANTHROPIC_API_KEY` | Anthropic API Key | — |
| `GOOGLE_API_KEY` | Google AI API Key | — |
| `TELEGRAM_BOT_TOKEN` | Telegram Bot Token | — |
| `DISCORD_BOT_TOKEN` | Discord Bot Token | — |
| `NODE_ENV` | 運行環境 | `development` |
| `DEBUG` | Debug 模式 | — |

#### 使用環境變數

```bash
# 喺 shell profile 入面設定（~/.bashrc 或 ~/.zshrc）
export OPENAI_API_KEY="sk-..."
export TELEGRAM_BOT_TOKEN="123456:ABC..."

# 或者喺 .env 文件入面設定
echo 'OPENAI_API_KEY=sk-...' >> ~/.openclaw/.env
```

> **💡 提示：** 環境變數優先級高於 config 文件中嘅相同設定。

---

*教程完。多謝閱讀！有問題歡迎去 Discord 社群（https://discord.com/invite/clawd）交流。* ⭐
