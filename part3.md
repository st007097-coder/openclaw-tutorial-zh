## 第七章：其他通訊平台

Telegram 同 Discord 係最常用嘅平台，但 OpenClaw 仲支持好多其他通訊工具。呢章教你點樣連接佢哋。

### 7.1 iMessage

iMessage 係 Apple 嘅原生通訊工具，OpenClaw 可以喺 macOS 上連接 iMessage。

**前提條件：**
- macOS 設備（Mac 電腦）
- 已登入 Apple ID
- 已啟用 iMessage

**設定步驟：**

```bash
# 喺 macOS 上安裝 OpenClaw
npm install -g openclaw

# 啟動 OpenClaw
openclaw gateway start
```

喺 `openclaw.json` 加入 iMessage 配置：

```json
{
  "channels": {
    "imessage": {
      "enabled": true
    }
  }
}
```

**注意事項：**
- iMessage channel **只支援 macOS**，Linux 同 Windows 用唔到
- 需要 macOS 嘅 Messages app 正常運作
- 支援群組 chat 同一對一 DM

> 💡 **小貼士：** 如果你用緊 Mac 做伺服器，iMessage 係一個好方便嘅選擇，唔使額外安裝任何第三方工具。

**小結：** iMessage 設定簡單，但限制係必須用 macOS。如果你已經有 Mac 喺度跑，加幾行 config 就搞得掂。

---

### 7.2 Signal

Signal 係一個注重私隱嘅通訊工具，OpenClaw 透過 `signal-cli` 連接 Signal。

**前提條件：**
- 安裝 `signal-cli`
- 一個已註冊嘅電話號碼

**安裝 signal-cli：**

```bash
# 下載 signal-cli
wget https://github.com/AsamK/signal-cli/releases/download/v0.13.9/signal-cli-0.13.9.tar.gz
tar xzf signal-cli-0.13.9.tar.gz
sudo mv signal-cli-0.13.9 /opt/signal-cli
sudo ln -s /opt/signal-cli/bin/signal-cli /usr/local/bin/

# 註冊電話號碼（需要接收 SMS 驗證碼）
signal-cli -a +852XXXXXXXX register

# 驗證
signal-cli -a +852XXXXXXXX verify <CODE>
```

**OpenClaw 配置：**

```json
{
  "channels": {
    "signal": {
      "enabled": true,
      "config": {
        "account": "+852XXXXXXXX",
        "signalCliPath": "/usr/local/bin/signal-cli"
      }
    }
  }
}
```

**測試連接：**

```bash
# 喺另一部電話 send message 畀呢個號碼
# 喺 OpenClaw log 入面 check 有冇收到
openclaw gateway status
```

**小結：** Signal 設定需要多幾步，但勝在私隱度高。適合注重安全嘅用家。記得 signal-cli 需要 Java runtime。

---

### 7.3 Slack

Slack 係團隊協作嘅首選工具，OpenClaw 可以作為 Slack Bot 加入 workspace。

**第一步：建立 Slack App**

1. 去 [api.slack.com/apps](https://api.slack.com/apps)
2. Click「Create New App」→「From scratch」
3. 輸入 App 名稱，揀你嘅 workspace

**第二步：設定 Bot 權限**

喺「OAuth & Permissions」加入以下 scopes：

```
app_mentions:read
channels:history
channels:read
chat:write
groups:history
groups:read
im:history
im:read
im:write
reactions:read
reactions:write
users:read
```

**第三步：安裝到 Workspace**

1. 喺「OAuth & Permissions」頁面 click「Install to Workspace」
2. 撳「Allow」授權
3. Copy「Bot User OAuth Token」（`xoxb-` 開頭）

**第四步：設定 Event Subscriptions**

1. 喺「Event Subscriptions」啟用
2. Request URL 填入：`https://你的域名/slack/events`
3. Subscribe to bot events：
   - `app_mention`
   - `message.channels`
   - `message.groups`
   - `message.im`

**第五步：OpenClaw 配置**

```json
{
  "channels": {
    "slack": {
      "enabled": true,
      "config": {
        "botToken": "xoxb-你的-Bot-Token",
        "signingSecret": "你的-Signing-Secret",
        "appToken": "xapp-你的-App-Token"
      }
    }
  }
}
```

**小結：** Slack Bot 設定步驟較多，但一旦搞好，團隊協作非常方便。記住要設定正確嘅 Event Subscriptions，唔係收唔到訊息。

---

### 7.4 Google Chat

Google Chat 適合用 Google Workspace 嘅團隊。

**第一步：建立 Service Account**

1. 去 [Google Cloud Console](https://console.cloud.google.com/)
2. 建立新 project 或揀現有嘅
3. 啟用 Google Chat API
4. 建 Service Account，下載 JSON key file

**第二步：設定 Chat App**

喺 Google Chat API 設定頁面：
- App name：填你想要嘅名
- Connection settings：選「HTTP endpoint」
- URL：`https://你的域名/google-chat/events`

**第三步：OpenClaw 配置**

```json
{
  "channels": {
    "googlechat": {
      "enabled": true,
      "config": {
        "serviceAccountKeyFile": "/path/to/service-account-key.json",
        "spaceId": "spaces/XXXXX"
      }
    }
  }
}
```

**小結：** Google Chat 適合已經用緊 Google Workspace 嘅公司，Service Account 設定有啲繁瑣，但係 Google 生態整合度高。

---

### 7.5 其他平台簡介

OpenClaw 仲透過 plugin 系統支持更多平台：

| 平台 | 連接方式 | 適用場景 |
|------|---------|---------|
| **Matrix** | Matrix bridge | 開源去中心化通訊 |
| **Mattermost** | Mattermost plugin | 自架 Slack 替代品 |
| **MS Teams** | Teams plugin | 微軟生態 |
| **IRC** | IRC plugin | 老牌聊天室 |
| **Nostr** | Nostr plugin | Web3 去中心化 |

**安裝 plugin 嘅一般步驟：**

```bash
# 安裝對應 plugin
openclaw plugins install <plugin-name>

# 設定 config（每個 plugin 唔同）
# 喺 openclaw.json 嘅 plugins 入面配置
```

**Plugin 配置範例（以 Matrix 為例）：**

```json
{
  "plugins": {
    "matrix": {
      "enabled": true,
      "config": {
        "homeserver": "https://matrix.org",
        "accessToken": "syt_你的token",
        "userId": "@yourbot:matrix.org"
      }
    }
  }
}
```

**小結：** OpenClaw 嘅 plugin 架構令佢可以連接幾乎任何通訊平台。如果你用緊嘅平台唔喺預設支持列表，可以考慮自己寫 plugin 或者喺社群搵現成嘅。

---

## 第八章：Agent（代理）深入理解

### 8.1 Agent 係乜嘢？

簡單講，**Agent 係一個完整嘅 AI「大腦」**。佢有自己嘅：

- **Workspace**（工作空間）— 優先記憶同操作指令
- **Sessions**（會話）— 同唔同人嘅對話
- **Auth**（認證）— API key 同權限

預設情況下，OpenClaw 只有一個 agent，叫做 `"main"`。呢個就係你平時傾偈嗰個。

**點解需要多個 Agent？**

- 一個做私人助手，處理你嘅日程同郵件
- 一個做客服，答客人問題
- 一個做翻譯，專門處理多語言

每個 agent 嘅 workspace 互相隔離，唔會搞亂對方嘅記憶。

---

### 8.2 AGENTS.md — 操作指令

`AGENTS.md` 係 agent 嘅「員工手冊」，定義咗佢應該點做嘢。

**位置：** `<workspace>/AGENTS.md`

**範例 AGENTS.md：**

```markdown
# AGENTS.md - Your Workspace

## Session Startup

每次開新 session 嘅時候：
1. 讀 `SOUL.md` — 了解自己係邊個
2. 讀 `USER.md` — 了解服務對象
3. 讀 `memory/今日日期.md` — 睇最近發生咩事
4. 如果係主 session，讀 `MEMORY.md`

## Memory 規則

- 重要決定 → 寫入 MEMORY.md
- 日常瑣事 → 寫入 memory/YYYY-MM-DD.md
- 唔記得嘅嘢 → 用 memory_search 搵

## Red Lines

- 唔可以洩露私隱資料
- 刪嘢之前要問
- 對外操作（寄 email、出 post）要確認
```

**小結：** AGENTS.md 係最重要嘅設定文件，所有行為規則都喺度定義。寫得越詳細，agent 表現越好。

---

### 8.3 SOUL.md — 定義性格

`SOUL.md` 定義 agent 嘅性格、語氣同邊界。好似一個人嘅靈魂咁。

**範例 SOUL.md：**

```markdown
# SOUL.md - Who You Are

## 核心原則

- 真誠有用，唔講廢話
- 有意見，唔係人哋講乜就跟乜
- 先自己試，唔得先問
- 用行動贏取信任

## 語氣

- 簡潔直接
- 有需要時詳細解釋
- 唔做 corporate drone
- 唔拍馬屁

## 邊界

- 私隱嘢唔講出去
- 對外操作要問過先做
- 群組入面唔代表用戶發言
```

**小結：** SOUL.md 令你嘅 agent 有「性格」，唔係一個死板嘅機械人。花啲時間寫好佢，效果差好遠。

---

### 8.4 USER.md — 用戶資料

`USER.md` 記錄關於你（用戶）嘅資料，等 agent 識得點樣同你溝通。

**範例 USER.md：**

```markdown
# USER.md - About Your Human

- **Name:** 期哥
- **Timezone:** Asia/Hong_Kong (UTC+8)
- **語言：** 廣東話為主

## Context

- 住喺香港
- 正職：工料測量師
- 用 Telegram 溝通
```

**小結：** USER.md 幫 agent 了解服務對象，包括偏好語言、時區同背景資料。

---

### 8.5 IDENTITY.md — 身份設定

`IDENTITY.md` 定義 agent 自己嘅身份。

**範例 IDENTITY.md：**

```markdown
# IDENTITY.md - Who Am I?

- **Name:** 阿星
- **Creature:** AI 助手
- **Vibe:** 親切、務實、有少少幽默感
- **Emoji:** ⭐
```

**小結：** IDENTITY.md 簡單但重要，令 agent 有自己嘅名同形象。

---

### 8.6 BOOTSTRAP.md — 初始化引導

`BOOTSTRAP.md` 只喺第一次運行時使用，用嚟引導 agent 初始化自己嘅設定。

**運作流程：**
1. 第一次 session 開始，agent 收到 BOOTSTRAP.md
2. 跟住入面嘅指示做（例如問用戶問題、建立文件）
3. 完成後**刪除** BOOTSTRAP.md
4. 之後嘅 session 就唔會再見到佢

**範例 BOOTSTRAP.md：**

```markdown
# BOOTSTRAP.md - First Run

你係第一次運行！請跟以下步驟：

1. 問用戶你叫咩名
2. 將答案寫入 IDENTITY.md
3. 問用戶佢哋嘅時區
4. 更新 USER.md
5. 建立 memory/ 目錄
6. 完成後刪除呢個文件
```

**小結：** BOOTSTRAP.md 用完即棄，係 agent 出世時嘅「接生指南」。

---

### 8.7 TOOLS.md — 工具筆記

`TOOLS.md` 記錄你環境入面嘅特殊工具配置。

**範例 TOOLS.md：**

```markdown
# TOOLS.md - Local Notes

## Obsidian

- Vault 路徑：/home/user/obsidian-vault
- API URL：https://localhost:27124/
- Token：xxxxx

## 攝影機

- 大門：camera-01
- 後園：camera-02

## TTS 設定

- 偏好 voice：en-US-Neural2-J
```

**小結：** TOOLS.md 係 agent 嘅「環境 cheat sheet」，記錄晒所有本地工具嘅細節。

---

### 8.8 Workspace 文件管理

OpenClaw 喺 session 開始時會自動注入 workspace 文件到 context。但係有大小限制：

| 設定 | 默認值 | 說明 |
|------|-------|------|
| `bootstrapMaxChars` | 20,000 | 單個文件最大字符數 |
| `bootstrapTotalMaxChars` | 150,000 | 所有 bootstrap 文件總字符數 |

**文件注入優先次序：**

1. `SOUL.md` — 必定注入
2. `USER.md` — 必定注入
3. `IDENTITY.md` — 必定注入
4. `AGENTS.md` — 必定注入
5. `TOOLS.md` — 必定注入
6. `MEMORY.md` — 只喺主 session 注入
7. `HEARTBEAT.md` — 收到心跳時注入
8. `memory/YYYY-MM-DD.md` — 今日同昨日嘅日誌

**管理技巧：**

- 大文件會自動截斷，所以唔好喺 workspace 入面放太多嘢
- 用 `memory_search` 搵舊資料，唔使將所有嘢塞晒入 context
- 定期整理 MEMORY.md，刪除過時嘅內容

**小結：** Workspace 文件係 agent 嘅記憶基礎，但要精打細算——context 有限，唔好浪費喺唔重要嘅嘢上面。

---

## 第九章：Session（會話）管理

### 9.1 Session 係乜嘢？

一個 **Session** 就係一次對話。每次你同 agent 傾偈，背後就係一個 session 喺運作。

Session 包含：
- **Message history**（訊息歷史）— 對話記錄
- **Context**（上下文）— Agent 嘅 workspace 文件 + 對話記憶
- **Auth**（認證）— 誰喺度傾偈

---

### 9.2 DM Scope 設定

DM Scope 決定咗點樣分配私訊 session：

```json
{
  "agents": {
    "defaults": {
      "dmScope": "main"
    }
  }
}
```

**四種模式比較：**

| 模式 | 說明 | 適用場景 |
|------|------|---------|
| `main` | 所有 DM 共享一個 session | 單人使用 |
| `per-peer` | 按發送者隔離 | 多人但用同一個 bot |
| `per-channel-peer` | 按頻道+發送者隔離 | 推薦多人使用 |
| `per-account-channel-peer` | 按帳號+頻道+發送者隔離 | 最嚴格隔離 |

**設定範例（per-channel-peer）：**

```json
{
  "agents": {
    "defaults": {
      "dmScope": "per-channel-peer"
    }
  }
}
```

**實際影響：**

如果用 `main` 模式，A 君同 B 君 send 畀同一個 bot，佢哋會共享同一個 session——即係互相睇到對方嘅對話。如果你個 bot 俾多個人用，一定要用 `per-peer` 或以上。

**小結：** DM Scope 係多人使用時必睇嘅設定。個人用揀 `main`，多人用揀 `per-peer` 或 `per-channel-peer`。

---

### 9.3 群組 Session

群組入面，session ID 嘅格式係：

```
agent:<agentId>:<channel>:group:<id>
```

例如 Telegram 群組嘅 session 可能係：

```
agent:main:telegram:group:-100123456789
```

**Telegram Topics：**

如果群組啟用咗 Topics（forum mode），session ID 會再細分：

```
agent:main:telegram:group:-100123456789:topic:42
```

咁每個 topic 就有獨立嘅上下文，唔會搞亂。

**小結：** 群組 session 自動隔離，每個群（甚至每個 topic）有獨立嘅對話上下文。

---

### 9.4 Session 生命週期

一個 session 嘅生命係咁樣嘅：

```
建立 → 對話 → 壓縮 → 重置 → 歸檔 → 清理
```

**Session 重置：**

默認每日凌晨 4:00 AM 自動重置。你都可以手動觸發：

```
/new    # 啟動新 session
/reset  # 重置當前 session
```

**Session 維護設定：**

```json
{
  "agents": {
    "defaults": {
      "session": {
        "pruneAfter": "30d",
        "maxEntries": 500,
        "rotateBytes": "10mb"
      }
    }
  }
}
```

| 設定 | 說明 |
|------|------|
| `pruneAfter` | 30 日無活動就清理 |
| `maxEntries` | 最多保留 500 條訊息 |
| `rotateBytes` | 超過 10MB 就輪轉 |

**小結：** Session 有完整嘅生命週期管理，自動清理舊 session，唔使擔心 storage 爆煲。

---

### 9.5 Session 壓縮（Compaction）

當對話太長、context 接近上限時，OpenClaw 會自動觸發 **Compaction**（壓縮）。

**Compaction 做咩：**
1. 將舊嘅對話摘要化
2. 保留重要上下文
3. 丟棄詳細但唔重要嘅內容

**Pre-compaction Memory Flush：**

壓縮之前，系統會自動提醒 agent：
> 「Session 即將壓縮，請將重要資訊寫入記憶文件。」

咁 agent 就會喺壓縮之前，將重要嘅嘢 save 到 `memory/` 文件度。

**相關配置：**

```json
{
  "agents": {
    "defaults": {
      "compaction": {
        "memoryFlush": true,
        "thresholdPercent": 80
      }
    }
  }
}
```

**小結：** Compaction 係 OpenClaw 嘅「自動減肥」機制，確保 context 唔會爆煲，同時用 memory flush 保護重要資訊唔會流失。

---

### 9.6 多 Agent Session 路由

如果你有多個 agent，可以設定路由規則，將唔同類型嘅訊息送去唔同 agent：

```json
{
  "agents": {
    "entries": {
      "main": {
        "model": "openrouter/anthropic/claude-sonnet-4"
      },
      "support": {
        "model": "openrouter/google/gemini-2.5-flash",
        "workspace": "/home/user/support-workspace"
      }
    }
  }
}
```

**路由方式：**
- 用 `/agent support` 指令切換
- 喺群組入面用 @mention 指定 agent
- 用 Hooks 自動路由（見第十三章）

**小結：** 多 Agent 路由適合需要唔同專業領域嘅場景，每個 agent 有獨立嘅記憶同性格。

---

## 第十章：Memory（記憶）系統

### 10.1 Agent 點樣記嘢

OpenClaw 嘅 agent 每次 session 都係「全新」嘅——佢唔會自動記住上一次嘅嘢。但透過 **Memory 系統**，佢可以好似人類咁有記憶。

**記憶嘅兩個層次：**

1. **Short-term（短期記憶）** — `memory/YYYY-MM-DD.md`，每日一個文件
2. **Long-term（長期記憶）** — `MEMORY.md`，經過策劃嘅重要資訊

---

### 10.2 Workspace Files

**記憶相關嘅 workspace 文件：**

```
workspace/
├── MEMORY.md              # 長期記憶（只喺主 session 加載）
├── memory/
│   ├── 2026-04-01.md      # 今日日誌
│   ├── 2026-03-31.md      # 昨日日誌
│   └── ...                # 歷史日誌
```

**每日日誌格式範例：**

```markdown
# 2026-04-01

## 時間線

- 09:00 期哥問關於 QS 報價嘅問題
- 10:30 幫手整理咗 Notion VO Tracker
- 14:00 搵咗關於合約條款嘅資料
- 16:00 同期哥討論咗項目進度

## 重要事項

- VO 報價 deadline 係 4月15日
- 期哥下星期要出差去深圳
```

**長期記憶格式範例：**

```markdown
# MEMORY.md - Long-Term Memory

## 關於期哥

- 正職：工料測量師（QS）
- 喺香港做工程項目
- 偏好用廣東話溝通

## 決定記錄

- 2026-03-15：決定用 OpenClaw 管理所有通訊
- 2026-03-20：Notion 做項目管理主要工具

## 常用知識

- 合約 VO 流程：收到變更通知 → 報價 → 審批 → 落單
```

**小結：** 每日日誌係 raw log，長期記憶係精華。兩者配合使用，agent 就唔會「失憶」。

---

### 10.3 記憶搜索

唔使將所有記憶都塞入 context，OpenClaw 提供咗語義搜索工具：

**memory_search — 語義搜索：**

```
用 memory_search 搜尋：「上次期哥講嘅 VO 金額係幾多？」
→ 返回相關嘅記憶片段
```

**memory_get — 精確讀取：**

```
用 memory_get 讀取：memory/2026-03-15.md
→ 返回該文件嘅完整內容
```

**搜索技術：**

OpenClaw 用 **semantic + BM25 hybrid search**：
- Semantic search：理解語意，唔係逐字 match
- BM25：傳統關鍵詞搜索
- 兩者結合，準確度高好多

**小結：** 記憶搜索令 agent 可以喺大量歷史資料入面快速搵到需要嘅資訊，唔使全部讀晒。

---

### 10.4 Long-term vs Short-term 記憶

| 特性 | Short-term（每日日誌） | Long-term（MEMORY.md） |
|------|----------------------|----------------------|
| 格式 | `memory/YYYY-MM-DD.md` | `MEMORY.md` |
| 內容 | Raw 日誌 | 策劃過嘅精華 |
| 加載 | 喺 session 時按需讀取 | 只喺主 session 注入 |
| 維護 | 自動 append | 手動策劃更新 |
| 保存期 | 永久（但舊嘅少用） | 持續更新 |

**幾時用邊個：**

- **即時記錄** → 寫入今日嘅 `memory/YYYY-MM-DD.md`
- **重要決定** → 同時更新 `MEMORY.md`
- **搵舊資料** → 用 `memory_search` 搜
- **長期知識** → 放喺 `MEMORY.md`

**小結：** 短期記憶好似日記，長期記憶好似百科全書。兩者各有用途，配合使用先係最佳實踐。

---

### 10.5 自動記憶管理

OpenClaw 有幾個自動機制幫手管理記憶：

**1. Auto Memory Flush（自動記憶儲存）**

當 session 接近 compaction 時，系統會自動提醒 agent 寫入記憶：

```json
{
  "agents": {
    "defaults": {
      "compaction": {
        "memoryFlush": true
      }
    }
  }
}
```

**2. Session 啟動記憶加載**

每次新 session 開始時，agent 會自動：
- 讀取今日同昨日嘅 `memory/` 日誌
- 喺主 session 讀取 `MEMORY.md`

**3. 定期記憶整理（透過 Heartbeat）**

喺 HEARTBEAT.md 入面加入：

```markdown
## Memory Maintenance

每幾日做一次：
1. 睇吓近期嘅 memory/ 日誌
2. 將重要資訊策劃到 MEMORY.md
3. 刪除 MEMORY.md 入面過時嘅嘢
```

**小結：** 自動記憶管理減輕咗 agent 嘅負擔，令記憶系統可以長期運作而唔需要人手干預。

---

## 第十一章：Skills（技能）系統

### 11.1 Skills 係乜嘢？

一個 **Skill** 就係一個包含 `SKILL.md` 嘅目錄，入面教 agent 點樣做某樣嘢。

想像 skills 好似「擴充包」——你可以加新功能俾 agent，而唔使改核心程式。

**Skills 嘅來源（優先次序）：**

1. `<workspace>/skills/` — 你自己寫嘅（最高優先）
2. `~/.openclaw/skills/` — 全局安裝嘅
3. `~/.agents/skills/` — Agent 專用嘅
4. Bundled — OpenClaw 內置嘅

---

### 11.2 ClawHub — 技能市場

**ClawHub** (https://clawhub.com) 係 OpenClaw 嘅公開技能市場，好似 App Store 咁，有大量現成嘅 skill 可以下載。

**瀏覽技能：**

```bash
# 搜尋技能
openclaw skills search weather

# 睇技能詳情
openclaw skills info weather
```

---

### 11.3 安裝技能

**安裝：**

```bash
# 安裝技能
openclaw skills install weather

# 更新所有已安裝嘅技能
openclaw skills update --all

# 列出已安裝嘅技能
openclaw skills list
```

**配置已安裝嘅技能：**

```json
{
  "skills": {
    "entries": {
      "weather": {
        "enabled": true,
        "config": {
          "defaultCity": "Hong Kong"
        }
      },
      "obsidian": {
        "enabled": true,
        "env": {
          "OBSIDIAN_API_URL": "https://localhost:27124/"
        }
      }
    }
  }
}
```

**Gating（門檻檢查）：**

有啲 skill 需要特定條件先可以運行：
- `bins`：需要某啲 command-line 工具
- `env`：需要環境變數
- `config`：需要特定配置

如果唔滿足條件，skill 會顯示安裝失敗嘅原因。

**Token 影響：**

每個 skill 大約用 97 個字符 + 名稱同描述嘅長度。裝太多 skill 會佔用 context，所以只裝需要嘅。

**小結：** ClawHub 令你可以快速擴展 agent 嘅能力，好似砌積木咁加功能。

---

### 11.4 創建自訂技能

**Step 1：建立目錄結構**

```bash
mkdir -p ~/workspace/skills/my-skill
```

**Step 2：撰寫 SKILL.md**

```markdown
---
name: my-skill
description: 我嘅自訂技能，用嚟做 XYZ
version: 1.0.0
author: 你的名
openclaw:
  requires:
    bins:
      - curl
    env:
      - API_KEY
---

# My Skill

## 用途

呢個技能用嚟做 XYZ。

## 使用方法

當用戶要求做 XYZ 時：

1. 搵到相關資料
2. 執行 ABC
3. 回傳結果

## 範例

用戶：「幫我做 XYZ」
→ 用 curl 呼叫 API → 整理結果 → 回覆用戶

## 錯誤處理

- 如果 API 唔回應，等 5 秒後重試
- 重試 3 次都失敗就報告錯誤
```

**Step 3：測試技能**

```bash
# 重新載入 skills
openclaw gateway restart

# 測試
# 喺對話入面觸發你嘅 skill
```

**AgentSkills 格式細節：**

- **YAML frontmatter**：定義 metadata（name, description, requires 等）
- **Markdown body**：教 agent 點樣用嘅指令
- 支援引用相對路徑嘅文件

**小結：** 寫自訂 skill 並唔難，核心就係一個 `SKILL.md` 文件。寫得好嘅 skill 文件等於教識 agent 一樣新技能。

---

### 11.5 常用技能推薦

以下係一啲好實用嘅 skills，強烈建議安裝：

| 技能 | 用途 | 安裝命令 |
|------|------|---------|
| `weather` | 查天氣 | `openclaw skills install weather` |
| `healthcheck` | 主機安全檢查 | `openclaw skills install healthcheck` |
| `skill-creator` | 建立新技能 | `openclaw skills install skill-creator` |
| `obsidian` | 管理 Obsidian 筆記 | `openclaw skills install obsidian` |
| `nano-banana-pro` | AI 圖片生成 | `openclaw skills install nano-banana-pro` |

**小結：** 喺 ClawHub 入面有好多好用嘅現成技能，善用佢哋可以慳好多時間。

---

