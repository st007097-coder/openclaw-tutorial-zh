## 第十二章：Tools（工具）使用

### 12.1 exec — 執行 Shell 命令

`exec` 係最強大嘅工具之一，可以喺伺服器上執行任何 shell 命令。

**基本用法：**

```bash
# 列出檔案
ls -la

# 檢查系統狀態
df -h
free -m

# 執行 Python 腳本
python3 script.py
```

**背景執行：**

```bash
# 長時間運行嘅任務，用 yieldMs 自動背景化
# 設定 yieldMs: 5000 表示超過 5 秒就轉到背景
```

**安全模式：**

| 模式 | 說明 |
|------|------|
| `sandbox` | 最嚴格，隔離環境 |
| `gateway` | 中等，有限權限 |
| `node` | 完整權限（要小心） |

**Exec Approvals：**

有啲危險命令需要你批准先會執行：
- `rm`、`mv` 大量檔案
- 安裝軟件
- 修改系統設定

```bash
# 如果收到 approval request
/approve allow-once   # 單次允許
/approve allow-always  # 永久允許
/deny                  # 拒絕
```

**小結：** exec 令 agent 可以做幾乎任何嘢，但安全第一——用 sandbox 模式、啟用 approvals。

---

### 12.2 browser — 瀏覽器自動化

`browser` 工具可以控制瀏覽器，做網頁自動化。

**兩種瀏覽器模式：**

| 模式 | 說明 | 用途 |
|------|------|------|
| `openclaw` profile | 隔離嘅 agent 專用瀏覽器 | 自動化任務 |
| `user` profile | 連接用戶真實 Chrome | 需要用戶登入狀態 |

**基本操作流程：**

```
1. Open page → 2. Snapshot → 3. 搵到 ref → 4. 動作（click/type）
```

**Snapshot 兩種模式：**

- **AI Snapshot**：適合複雜頁面，AI 理解頁面結構
- **Role Snapshot**：標準 accessibility tree，速度快

**範例：自動填表**

```
1. browser open "https://example.com/form"
2. browser snapshot（取得頁面結構）
3. browser click ref="e12"（click 某個元素）
4. browser type ref="e15" text="Hello World"
5. browser click ref="submit-btn"
```

**遙控 CDP：**

支援 Browserless、Browserbase 等遠端瀏覽器服務，唔使喺本地跑瀏覽器。

**小結：** browser 自動化令 agent 可以上網做嘢，填表、查資料、截圖，用途廣泛。

---

### 12.3 web_search / web_fetch — 網絡搜索

**web_search — 搜索引擎：**

OpenClaw 支援多種搜索引擎：
- Brave Search
- Perplexity
- Gemini（Google Search）
- Grok

```bash
# 基本搜索
web_search "OpenClaw 教程"

# 限定結果數量
web_search "香港天氣" count=3
```

**web_fetch — 抓取網頁：**

```bash
# 抓取網頁內容，轉成 Markdown
web_fetch "https://example.com/article"

# 只抓文字（唔要格式）
web_fetch "https://example.com/article" extractMode=text

# 限制字數
web_fetch "https://example.com" maxChars=5000
```

**實際使用場景：**

```
用戶：「今日有咩新聞？」
→ web_search "香港今日新聞"
→ 搵到結果
→ 回覆摘要
```

```
用戶：「幫我睇吓呢個網頁講咩」
→ web_fetch "https://example.com/long-article"
→ 讀取內容
→ 總結重點
```

**小結：** web_search 同 web_fetch 係 agent 上網嘅兩大工具，一個用嚟搵嘢，一個用嚟讀嘢。

---

### 12.4 文件操作

OpenClaw 支援基本嘅文件操作：

**read — 讀取文件：**

```bash
# 讀取文件
read "/path/to/file.txt"

# 讀取部分內容（大文件）
read "/path/to/large-file.txt" offset=100 limit=50
```

**write — 寫入文件：**

```bash
# 建立或覆蓋文件
write "/path/to/new-file.txt" content="Hello World"
```

**edit — 編輯文件：**

```bash
# 精確替換
edit "/path/to/file.txt" oldText="舊內容" newText="新內容"
```

**實際範例：**

```
用戶：「幫我喺 config.json 入面將 port 改做 8080」
→ read config.json
→ 找到 "port": 3000
→ edit config.json oldText='"port": 3000' newText='"port": 8080'
```

**小結：** 文件操作簡單直接，read / write / edit 三個命令就搞定大部分需求。

---

### 12.5 TTS — 語音合成

TTS（Text-to-Speech）可以將文字轉成語音。

**基本用法：**

```
tts "你好，我係阿星，有咩可以幫到你？"
```

**使用場景：**

- 講故仔、總結新聞
- 回覆長篇內容時，用語音更方便
- 視障用戶友好

**注意事項：**
- TTS 音訊會自動送出到當前 channel
- 成功後回覆 `NO_REPLY` 避免重複訊息

**小結：** TTS 令 agent 可以「開聲」講嘢，增加互動體驗。

---

### 12.6 其他工具

**image_generate — 圖片生成：**

```
image_generate "一隻戴住安全帽嘅卡通貓喺工地度"
```

支援 OpenAI、Google 等多種圖片生成模型。

**pdf — PDF 分析：**

```
pdf "/path/to/document.pdf" prompt="總結呢份文件嘅重點"
```

可以分析 PDF 嘅文字同圖片內容。

**canvas — UI 呈現：**

```
canvas present url="https://example.com"
```

將網頁或 HTML 呈現喺 canvas 上，可以截圖同互動。

**小結：** OpenClaw 嘅工具箱非常豐富，幾乎任何任務都可以搵到合適嘅工具。

---

## 第十三章：Automation（自動化）

### 13.1 Heartbeat — 心跳輪詢

**Heartbeat** 係 OpenClaw 嘅定期「心跳檢查」，令 agent 可以主動做嘢，唔使等人叫。

**運作原理：**

```
設定間隔 → 定時觸發 → 讀 HEARTBEAT.md → 執行檢查清單 → 回報結果
```

**設定：**

```json
{
  "agents": {
    "defaults": {
      "heartbeat": {
        "enabled": true,
        "intervalMs": 1800000
      }
    }
  }
}
```

上面設定咗每 30 分鐘（1,800,000 ms）觸發一次。

**HEARTBEAT.md 範例：**

```markdown
# HEARTBEAT.md - 心跳檢查清單

## 每次檢查（3-4 次/日）

- [ ] 有冇新 email？
- [ ] 未來 24 小時有冇 calendar 事件？
- [ ] Twitter 有冇新 mention？

## 注意事項

- 深夜（23:00-08:00）除非有緊急事，唔好打擾用戶
- 如果冇新嘢，回覆 HEARTBEAT_OK
- 每次檢查唔超過 2-3 項，慳 token
```

**狀態追蹤：**

用 `memory/heartbeat-state.json` 追蹤上次檢查時間：

```json
{
  "lastChecks": {
    "email": 1743460800,
    "calendar": 1743457200,
    "weather": null
  }
}
```

**幾時打擾用戶、幾時靜雞雞：**

| 情況 | 行動 |
|------|------|
| 重要 email 到咗 | send message 畀用戶 |
| Calendar 事件 < 2 小時 | 提醒用戶 |
| 冇新嘢 | HEARTBEAT_OK（唔好嘈） |
| 深夜 23:00-08:00 | 除非緊急，否則 HEARTBEAT_OK |

**小結：** Heartbeat 令 agent 由「被動回應」變做「主動出擊」，可以幫你巡邏 email、calendar 等等。

---

### 13.2 Cron Jobs — 定時任務

**Cron Jobs** 係精確定時執行嘅任務，同 heartbeat 唔同——cron 要求準時，heartbeat 可以漂移。

**設定方式：**

喺 `openclaw.json` 入面：

```json
{
  "cron": {
    "jobs": [
      {
        "name": "morning-briefing",
        "schedule": "0 9 * * *",
        "agent": "main",
        "prompt": "做一份今日嘅 morning briefing：天氣、calendar、email 摘要"
      },
      {
        "name": "weekly-report",
        "schedule": "0 17 * * 5",
        "agent": "main",
        "prompt": "整理今週嘅工作記錄，做一份 weekly report"
      }
    ]
  }
}
```

**Cron 語法速查：**

```
分 時 日 月 星期

# 範例：
0 9 * * *     # 每日早上 9 點
30 14 * * 1   # 每逢星期一 2:30 PM
0 0 1 * *     # 每月 1 號 midnight
*/30 * * * *  # 每 30 分鐘
```

**Cron vs Heartbeat：**

| 特性 | Heartbeat | Cron |
|------|-----------|------|
| 時間精度 | 可以漂移 | 精確 |
| 執行環境 | 主 session | 獨立 session |
| 適合用途 | 批量巡邏檢查 | 精確定時任務 |
| 消耗 | 較多（定期觸發） | 較少（按需觸發） |

**實際範例：每日安全檢查**

```json
{
  "name": "security-audit",
  "schedule": "0 3 * * *",
  "agent": "main",
  "prompt": "執行主機安全檢查：檢查 SSH 設定、防火牆規則、系統更新狀態，將結果寫入 memory/security-audit.md"
}
```

**小結：** Cron jobs 適合需要精確時間嘅自動化任務。同 heartbeat 配合使用，可以覆蓋大部分自動化需求。

---

### 13.3 Hooks — 鉤子

**Hooks** 係事件驅動嘅自動化——當某啲事件發生時，自動執行指定動作。

**Hook 類型：**

| Hook | 觸發時機 |
|------|---------|
| `onSessionStart` | 新 session 開始時 |
| `onSessionEnd` | session 結束時 |
| `onMessage` | 收到訊息時 |
| `onError` | 發生錯誤時 |

**設定範例：**

```json
{
  "hooks": {
    "onSessionStart": [
      {
        "run": "echo 'Session started at $(date)' >> /tmp/sessions.log"
      }
    ],
    "onError": [
      {
        "run": "echo 'Error occurred' >> /tmp/errors.log",
        "notify": true
      }
    ]
  }
}
```

**實用場景：**

1. **Session 開始時自動讀取最新資料**
2. **發生錯誤時自動通知管理員**
3. **收到特定類型訊息時自動路由到對應 agent**

**小結：** Hooks 提供事件驅動嘅自動化能力，令 OpenClaw 可以對各種事件做出即時反應。

---

### 13.4 Webhook 接收

**Webhook** 令外部服務可以直接通知 OpenClaw agent。

**運作原理：**

```
外部服務（GitHub, Stripe, etc.）
    ↓ HTTP POST
OpenClaw Webhook Endpoint
    ↓
Agent 處理
    ↓
執行對應操作
```

**設定 Webhook：**

```json
{
  "webhooks": {
    "github": {
      "path": "/webhooks/github",
      "secret": "your-webhook-secret",
      "agent": "main",
      "handler": "GitHub push 嘅 {{event}} 事件：{{payload.commits[0].message}}"
    }
  }
}
```

**實際範例：GitHub Webhook**

1. 喺 GitHub repo 設定 → Webhooks
2. Payload URL：`https://你的域名/webhooks/github`
3. Content type：`application/json`
4. Secret：同上面 config 一樣
5. 選擇事件：Push, PR, Issues

當有新 push 時，OpenClaw agent 會收到通知，可以自動：
- 更新項目狀態
- 通知團隊成員
- 觸發 CI/CD 流程

**Webhook 安全：**

- 設定 `secret` 驗證來源
- 用 HTTPS
- 限制 IP 來源（如果可能）

**小結：** Webhook 令 OpenClaw 可以接收外部世界嘅通知，係整合第三方服務嘅關鍵。

---

## 🎉 教程總結

恭喜你睇晒呢份教程！由 Telegram 基本設定，到多平台連接、Agent 概念、Session 管理、記憶系統、技能擴展、工具使用，以至自動化，你已經掌握咗 OpenClaw 嘅核心知識。

**快速回顧：**

- **第七章：** iMessage、Signal、Slack、Google Chat 等多平台連接
- **第八章：** Agent 係乜嘢、workspace 文件點用
- **第九章：** Session 生命週期、DM Scope、Compaction
- **第十章：** 記憶系統、語義搜索、自動管理
- **第十一章：** ClawHub 技能市場、自訂技能
- **第十二章：** exec、browser、web_search 等工具
- **第十三章：** Heartbeat、Cron、Hooks、Webhook 自動化

**下一步建議：**

1. 裝幾個常用 skills（weather、healthcheck）
2. 寫好你自己嘅 SOUL.md 同 AGENTS.md
3. 設定 heartbeat 或 cron 做自動巡邏
4. 喺 ClawHub 睇吓有冇你需要嘅技能

OpenClaw 嘅世界好大，慢慢探索啦！🚀
---

## 第十四章：Mobile Nodes（手機節點）

### 14.1 Node 係乜嘢？

**Node**（節點）就係你部手機、平板、或者 headless 裝置（例如 Raspberry Pi），佢通過 **WebSocket** 連接返去 Gateway。

簡單講：
- **Gateway** = 大腦（喺伺服器度跑）
- **Node** = 手腳（喺你部手機度跑）

Node 可以幫你做啲 Gateway 做唔到嘅嘢，例如影相、攞 GPS 位置、螢幕錄影等。

**Node 嘅類型：**
| 類型 | 說明 |
|------|------|
| iOS | iPhone / iPad，用 OpenClaw iOS App |
| Android | Android 手機，用 OpenClaw Android App |
| Headless | 無 UI 裝置，例如 Raspberry Pi、伺服器 |

### 14.2 配對流程

Node 同 Gateway 之間要**配對**（pair）先可以通訊。配對係 device-based，即每部裝置都要單獨 approve。

#### Step 1：喺 Node App 入面發起配對

打開 OpenClaw App，揀「配對新 Gateway」，輸入 Gateway 嘅 URL。

#### Step 2：Gateway 收到配對請求

Gateway 會收到一個配對請求，你可以喺 Telegram 或者 Web 控制台 approve：

```
/pair
```

Gateway 會顯示一個**配對碼**（pairing code）。

#### Step 3：喺 Node 輸入配對碼

將配對碼輸入到 Node App 入面。

#### Step 4：Approve 配對

喺 Gateway 端 approve：

```
/nodes approve <node-id>
```

#### 管理已配對裝置

```bash
# 列出所有已配對 Node
openclaw nodes list

# 撤銷配對
openclaw nodes revoke <node-id>

# 查看 Node 狀態
openclaw nodes status <node-id>
```

> **💡 提示：** 配對資訊存在 Gateway 嘅 device pairing store 入面，係持久化嘅，唔使每次重新配對。

### 14.3 相機、位置、語音功能

配對好之後，你就可以透過 Gateway 遠端控制 Node 嘅硬件功能：

#### 相機（Camera）

```bash
# 影相
camera.capture --facing back    # 後置鏡頭
camera.capture --facing front   # 前置鏡頭

# 開始錄影
camera.record start
camera.record stop
```

#### 位置（Location）

```bash
# 取得目前 GPS 位置
location.get

# 開始追蹤
location.track start --interval 60

# 停止追蹤
location.track stop
```

#### 螢幕錄影

```bash
# 開始螢幕錄影
screen.record start

# 停止並儲存
screen.record stop
```

#### 語音

```bash
# 開始錄音
voice.record start

# 停止
voice.record stop
```

#### 實際使用例子

你喺 Telegram 入面同阿星講：

> **你：** 影張相畀我睇吓
> **阿星：** 好！（自動調用 Node 嘅相機影相，然後 send 張相畀你）

> **你：** 我喺邊度？
> **阿星：** （取得 GPS 位置）你喺香港尖沙咀彌敦道附近。

### 14.4 Canvas

**Canvas** 係 Node 嘅一個強大功能，佢可以喺 Node 裝置上展示一個互動式介面。

#### Canvas 命令

```bash
# 展示一個 URL
canvas.present --url https://example.com

# 展示 HTML 內容
canvas.present --html "<h1>Hello World</h1>"

# 取得 Canvas 截圖
canvas.snapshot

# 隱藏 Canvas
canvas.hide

# 在 Canvas 中執行 JavaScript
canvas.eval --script "document.title = 'Updated'"
```

#### 實際應用

想像你喺屋企用 Node 裝置（例如平板）展示一個 Dashboard：

```bash
# 展示天氣 Dashboard
canvas.present --url https://weather-widget.example.com

# 展示日程表
canvas.present --html "<iframe src='https://calendar.google.com'></iframe>"
```

### 📝 第十四章小結

- **Node** 係連接 Gateway 嘅手機/平板/頭部裝置
- 配對流程：發起請求 → Gateway approve → 完成
- Node 提供相機、GPS、螢幕錄影、語音等功能
- **Canvas** 可以喺 Node 上展示互動式介面
- 所有操作都通過 Gateway 遠端控制

---

## 第十五章：安全與權限

安全係重中之重。OpenClaw 提供多層安全機制保護你嘅系統。

### 15.1 Gateway 安全

Gateway 係你嘅中央控制點，保護好佢就係保護你成個系統。

#### 認證方式

```json
{
  "gateway": {
    "auth": {
      "token": "your-secret-token-here",
      "password": "optional-password"
    }
  }
}
```

- **Token**：API 認證用，所有外部請求都要帶 token
- **Password**：Web 控制台登入用

> **⚠️ 重要：** 永遠唔好將 token 直接寫喺公開嘅地方（例如 GitHub）！

### 15.2 Token 管理

#### 設定 Token

```json
{
  "gateway": {
    "auth": {
      "token": "gw_abc123def456ghi789"
    }
  }
}
```

#### 更換 Token

```bash
# 用 openclaw CLI 更換
openclaw gateway token rotate

# 或者直接改 config
# 編輯 openclaw.json 入面嘅 gateway.auth.token
openclaw gateway restart
```

#### Token 最佳實踐

- 用強密碼生成器生成（至少 32 字元）
- 定期更換 token
- 唔同環境用唔同 token
- 用環境變數引用 token，唔好硬編碼

### 15.3 沙箱（Sandbox）

沙箱用 Docker 容器隔離 Agent 嘅操作，防止破壞性命令影響主系統。

#### 啟用沙箱

```json
{
  "sandbox": {
    "enabled": true,
    "mode": "non-main",
    "scope": "session"
  }
}
```

#### 沙箱模式

| 模式 | 說明 |
|------|------|
| `off` | 唔用沙箱（危險！） |
| `non-main` | 只喺非主 session 用沙箱（推薦） |
| `all` | 所有 session 都用沙箱 |

#### 沙箱範圍

| 範圍 | 說明 |
|------|------|
| `session` | 每個 session 獨立容器 |
| `agent` | 同一個 agent 共享容器 |
| `shared` | 所有操作共享一個容器 |

#### 沙箱工作原理

```
Agent 命令 → 沙箱檢查 → Docker 容器執行 → 結果返回
                ↓
         （隔離環境，唔影響主機）
```

### 15.4 Tool Policy

Tool Policy 控制 Agent 可以用邊啲工具，唔可以用邊啲。

#### 白名單模式（Allow）

```json
{
  "tools": {
    "allow": ["read", "write", "exec", "web_search"],
    "deny": []
  }
}
```

#### 黑名單模式（Deny）

```json
{
  "tools": {
    "allow": [],
    "deny": ["exec", "browser"]
  }
}
```

#### 常用工具列表

| 工具 | 功能 | 風險等級 |
|------|------|----------|
| `read` | 讀檔案 | 🟢 低 |
| `write` | 寫檔案 | 🟡 中 |
| `exec` | 執行命令 | 🔴 高 |
| `browser` | 瀏覽器自動化 | 🟡 中 |
| `web_search` | 網絡搜尋 | 🟢 低 |
| `message` | 發送訊息 | 🟡 中 |

### 15.5 allowFrom 白名單

`allowFrom` 控制邊啲人可以同 Agent 互動。

```json
{
  "allowFrom": [
    "telegram:123456789",
    "discord:user:987654321",
    "whatsapp:+85291234567"
  ]
}
```

#### dmPolicy

控制 Direct Message（私訊）策略：

```json
{
  "dmPolicy": "pairing"
}
```

| 值 | 說明 |
|----|------|
| `pairing` | 需要配對先可以私訊（最安全） |
| `allowlist` | 只允許白名單入面嘅人 |
| `open` | 任何人都可以私訊 |
| `disabled` | 唔接受私訊 |

### 15.6 Gateway Lock

Gateway Lock 防止未經授權嘅配置修改。

```json
{
  "gateway": {
    "lock": true
  }
}
```

啟用之後：
- 所有 config 修改都要經過認證
- RPC 調用需要授權
- 保護你嘅設定唔俾人改

### 15.7 安全檢查

OpenClaw 內置安全審計工具：

```bash
# 執行安全審計
openclaw security audit
```

審計會檢查：
- ✅ Token 強度
- ✅ 沙箱配置
- ✅ Tool Policy 設定
- ✅ allowFrom 白名單
- ✅ dmPolicy 配置
- ✅ SSRF policy for browser
- ✅ exec approvals 設定

#### 安全最佳實踐清單

- [ ] Gateway 有設定認證 token
- [ ] Token 至少 32 字元
- [ ] 沙箱已啟用（至少 non-main 模式）
- [ ] exec 工具喺 Tool Policy 入面受控
- [ ] allowFrom 白名單已設定
- [ ] dmPolicy 唔係 `open`
- [ ] Gateway Lock 已啟用
- [ ] 定期執行 `openclaw security audit`

### 📝 第十五章小結

- **Gateway 安全**：用 token 認證保護 API
- **沙箱**：Docker 隔離，防止破壞性操作
- **Tool Policy**：白名單/黑名單控制可用工具
- **allowFrom**：控制邊啲人可以同 Agent 互動
- **Gateway Lock**：防止未授權配置修改
- **定期審計**：`openclaw security audit`

---

