## 第七章：其他通訊平台

Telegram 和 Discord 是最常用的平台，但 OpenClaw 還支持很多其他通訊工具。本章教你如何連接它們。

### 7.1 iMessage

iMessage 是 Apple 的原生通訊工具，OpenClaw 可以在 macOS 上連接 iMessage。

**前提條件：**
- macOS 設備（Mac 電腦）
- 已登入 Apple ID
- 已啟用 iMessage

**設定步驟：**

```bash
# 在 macOS 上安裝 OpenClaw
npm install -g openclaw

# 啟動 OpenClaw
openclaw gateway start
```

在 `openclaw.json` 加入 iMessage 配置：

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
- iMessage channel **只支援 macOS**，Linux 和 Windows 無法使用
- 需要 macOS 的 Messages app 正常運作
- 支援群組 chat 和一對一 DM

> 💡 **小貼士：** 如果你正在用 Mac 做伺服器，iMessage 是一個很方便的選擇，不需要額外安裝任何第三方工具。

**小結：** iMessage 設定簡單，但限制是必須用 macOS。如果你已經有 Mac 在運行，加幾行 config 就能搞定。

---

### 7.2 Signal

Signal 是一個注重私隱的通訊工具，OpenClaw 透過 `signal-cli` 連接 Signal。

**前提條件：**
- 安裝 `signal-cli`
- 一個已註冊的電話號碼

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
# 在另一部電話 send message 到這個號碼
# 在 OpenClaw log 裡面 check 有沒有收到
openclaw gateway status
```

**小結：** Signal 設定需要多幾步，但優點在於私隱度高。適合注重安全的用家。記得 signal-cli 需要 Java runtime。

---

### 7.3 Slack

Slack 是團隊協作的首選工具，OpenClaw 可以作為 Slack Bot 加入 workspace。

**第一步：建立 Slack App**

1. 去 [api.slack.com/apps](https://api.slack.com/apps)
2. Click「Create New App」→「From scratch」
3. 輸入 App 名稱，選擇你的 workspace

**第二步：設定 Bot 權限**

在「OAuth & Permissions」加入以下 scopes：

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

1. 在「OAuth & Permissions」頁面 click「Install to Workspace」
2. 按「Allow」授權
3. Copy「Bot User OAuth Token」（`xoxb-` 開頭）

**第四步：設定 Event Subscriptions**

1. 在「Event Subscriptions」啟用
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

**小結：** Slack Bot 設定步驟較多，但一旦搞好，團隊協作非常方便。記住要設定正確的 Event Subscriptions，否則收不到訊息。

---

### 7.4 Google Chat

Google Chat 適合使用 Google Workspace 的團隊。

**第一步：建立 Service Account**

1. 去 [Google Cloud Console](https://console.cloud.google.com/)
2. 建立新 project 或選擇現有的
3. 啟用 Google Chat API
4. 建立 Service Account，下載 JSON key file

**第二步：設定 Chat App**

在 Google Chat API 設定頁面：
- App name：填你想要的名稱
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

**小結：** Google Chat 適合已經在使用 Google Workspace 的公司，Service Account 設定有些繁瑣，但 Google 生態整合度高。

---

### 7.5 其他平台簡介

OpenClaw 還透過 plugin 系統支持更多平台：

| 平台 | 連接方式 | 適用場景 |
|------|---------|---------|
| **Matrix** | Matrix bridge | 開源去中心化通訊 |
| **Mattermost** | Mattermost plugin | 自架 Slack 替代品 |
| **MS Teams** | Teams plugin | 微軟生態 |
| **IRC** | IRC plugin | 老牌聊天室 |
| **Nostr** | Nostr plugin | Web3 去中心化 |

**安裝 plugin 的一般步驟：**

```bash
# 安裝對應 plugin
openclaw plugins install <plugin-name>

# 設定 config（每個 plugin 不同）
# 在 openclaw.json 的 plugins 裡面配置
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

**小結：** OpenClaw 的 plugin 架構令它能夠連接幾乎任何通訊平台。如果你使用的平台不在預設支持列表，可以考慮自己寫 plugin 或者在社群尋找現成的。

---

## 第八章：Agent（代理）深入理解

### 8.1 Agent 是什麼？

簡單講，**Agent 是一個完整的 AI「大腦」**。它有自己的：

- **Workspace**（工作空間）— 優先記憶和操作指令
- **Sessions**（會話）— 和不同人的對話
- **Auth**（認證）— API key 和權限

預設情況下，OpenClaw 只有一個 agent，叫做 `"main"`。這就是你平時聊天的那一個。

**為什麼需要多個 Agent？**

- 一個做私人助手，處理你的日程和郵件
- 一個做客服，回答客人問題
- 一個做翻譯，專門處理多語言

每個 agent 的 workspace 互相隔離，不會搞亂對方的記憶。

---

### 8.2 AGENTS.md — 操作指令

`AGENTS.md` 是 agent 的「員工手冊」，定義了它應該如何工作。

**位置：** `<workspace>/AGENTS.md`

**範例 AGENTS.md：**

```markdown
# AGENTS.md - Your Workspace

## Session Startup

每次開新 session 的時候：
1. 讀 `SOUL.md` — 了解自己是誰
2. 讀 `USER.md` — 了解服務對象
3. 讀 `memory/今日日期.md` — 看最近發生什麼事
4. 如果是主 session，讀 `MEMORY.md`

## Memory 規則

- 重要決定 → 寫入 MEMORY.md
- 日常瑣事 → 寫入 memory/YYYY-MM-DD.md
- 不記得的東西 → 用 memory_search 尋找

## Red Lines

- 不可以洩露私隱資料
- 刪除東西之前要問
- 對外操作（寄 email、出 post）要確認
```

**小結：** AGENTS.md 是最重要的設定文件，所有行為規則都在這裡定義。寫得越詳細，agent 表現越好。

---

### 8.3 SOUL.md — 定義性格

`SOUL.md` 定義 agent 的性格、語氣和邊界。好像一個人的靈魂一樣。

**範例 SOUL.md：**

```markdown
# SOUL.md - Who You Are

## 核心原則

- 真誠有用，不講廢話
- 有意見，不是別人講什麼就跟什麼
- 先自己試，不行再問
- 用行動贏取信任

## 語氣

- 簡潔直接
- 有需要時詳細解釋
- 不做 corporate drone
- 不拍馬屁

## 邊界

- 私隱的東西不講出去
- 對外操作要問過先做
- 群組裡面不代表用戶發言
```

**小結：** SOUL.md 令你的 agent 有「性格」，不是一個死板的機械人。花些時間寫好它，效果差很遠。

---

### 8.4 USER.md — 用戶資料

`USER.md` 記錄關於你（用戶）的資料，讓 agent 知道如何和你溝通。

**範例 USER.md：**

```markdown
# USER.md - About Your Human

- **Name:** 期哥
- **Timezone:** Asia/Hong_Kong (UTC+8)
- **語言：** 廣東話為主

## Context

- 住在香港
- 正職：工料測量師
- 用 Telegram 溝通
```

**小結：** USER.md 幫助 agent 了解服務對象，包括偏好語言、時區和背景資料。

---

### 8.5 IDENTITY.md — 身份設定

`IDENTITY.md` 定義 agent 自己的身份。

**範例 IDENTITY.md：**

```markdown
# IDENTITY.md - Who Am I?

- **Name:** 阿星
- **Creature:** AI 助手
- **Vibe:** 親切、務實、有些少幽默感
- **Emoji:** ⭐
```

**小結：** IDENTITY.md 簡單但重要，令 agent 有自己的名字和形象。

---

### 8.6 BOOTSTRAP.md — 初始化引導

`BOOTSTRAP.md` 只在第一次運行時使用，用來引導 agent 初始化自己的設定。

**運作流程：**
1. 第一次 session 開始，agent 收到 BOOTSTRAP.md
2. 跟著裡面的指示做（例如問用戶問題、建立文件）
3. 完成後**刪除** BOOTSTRAP.md
4. 之後的 session 就不會再見到它

**範例 BOOTSTRAP.md：**

```markdown
# BOOTSTRAP.md - First Run

你是第一次運行！請跟隨以下步驟：

1. 問用戶你叫什麼名字
2. 將答案寫入 IDENTITY.md
3. 問用戶他們的時區
4. 更新 USER.md
5. 建立 memory/ 目錄
6. 完成後刪除這個文件
```

**小結：** BOOTSTRAP.md 用完即棄，是 agent 出生時的「接生指南」。

---

### 8.7 TOOLS.md — 工具筆記

`TOOLS.md` 記錄你環境裡面的特殊工具配置。

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

**小結：** TOOLS.md 是 agent 的「環境 cheat sheet」，記錄了所有本地工具的細節。

---

### 8.8 Workspace 文件管理

OpenClaw 在 session 開始時會自動注入 workspace 文件到 context。但有大小限制：

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
6. `MEMORY.md` — 只在主 session 注入
7. `HEARTBEAT.md` — 收到心跳時注入
8. `memory/YYYY-MM-DD.md` — 今日和昨日的日誌

**管理技巧：**

- 大文件會自動截斷，所以不要在 workspace 裡面放太多東西
- 用 `memory_search` 尋找舊資料，不需要將所有東西塞進 context
- 定期整理 MEMORY.md，刪除過時的內容

**小結：** Workspace 文件是 agent 的記憶基礎，但要精打細算——context 有限，不要浪費在不重要的東西上面。

---

## 第九章：Session（會話）管理

### 9.1 Session 是什麼？

一個 **Session** 就是一次對話。每次你和 agent 聊天，背後就是一個 session 在運作。

Session 包含：
- **Message history**（訊息歷史）— 對話記錄
- **Context**（上下文）— Agent 的 workspace 文件 + 對話記憶
- **Auth**（認證）— 誰在聊天

---

### 9.2 DM Scope 設定

DM Scope 決定了如何分配私訊 session：

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

如果用 `main` 模式，A 君和 B 君 send 到同一個 bot，他們會共享同一個 session——即是可以互相看到對方的對話。如果你的 bot 供多個人使用，一定要用 `per-peer` 或以上。

**小結：** DM Scope 是多人使用時必看的設定。個人用選 `main`，多人用選 `per-peer` 或 `per-channel-peer`。

---

### 9.3 群組 Session

群組裡面，session ID 的格式是：

```
agent:<agentId>:<channel>:group:<id>
```

例如 Telegram 群組的 session 可能是：

```
agent:main:telegram:group:-100123456789
```

**Telegram Topics：**

如果群組啟用了 Topics（forum mode），session ID 會再細分：

```
agent:main:telegram:group:-100123456789:topic:42
```

這樣每個 topic 就有獨立的上下文，不會搞亂。

**小結：** 群組 session 自動隔離，每個群（甚至每個 topic）有獨立的對話上下文。

---

### 9.4 Session 生命週期

一個 session 的生命是這樣的：

```
建立 → 對話 → 壓縮 → 重置 → 歸檔 → 清理
```

**Session 重置：**

默認每日凌晨 4:00 AM 自動重置。你也可以手動觸發：

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

**小結：** Session 有完整的生命週期管理，自動清理舊 session，不需要擔心 storage 爆掉。

---

### 9.5 Session 壓縮（Compaction）

當對話太長、context 接近上限時，OpenClaw 會自動觸發 **Compaction**（壓縮）。

**Compaction 做什麼：**
1. 將舊的對話摘要化
2. 保留重要上下文
3. 丟棄詳細但不重要的內容

**Pre-compaction Memory Flush：**

壓縮之前，系統會自動提醒 agent：
> 「Session 即將壓縮，請將重要資訊寫入記憶文件。」

這樣 agent 就會在壓縮之前，將重要的東西 save 到 `memory/` 文件裡。

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

**小結：** Compaction 是 OpenClaw 的「自動瘦身」機制，確保 context 不會爆掉，同時用 memory flush 保護重要資訊不會流失。

---

### 9.6 多 Agent Session 路由

如果你有多個 agent，可以設定路由規則，將不同類型的訊息送到不同 agent：

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
- 在群組裡面用 @mention 指定 agent
- 用 Hooks 自動路由（見第十三章）

**小結：** 多 Agent 路由適合需要不同專業領域的場景，每個 agent 有獨立的記憶和性格。

---

## 第十章：Memory（記憶）系統

### 10.1 Agent 如何記東西

OpenClaw 的 agent 每次 session 都是「全新」的——它不會自動記住上一次的東西。但透過 **Memory 系統**，它可以像人類一樣有記憶。

**記憶的兩個層次：**

1. **Short-term（短期記憶）** — `memory/YYYY-MM-DD.md`，每日一個文件
2. **Long-term（長期記憶）** — `MEMORY.md`，經過策劃的重要資訊

---

### 10.2 Workspace Files

**記憶相關的 workspace 文件：**

```
workspace/
├── MEMORY.md              # 長期記憶（只在主 session 加載）
├── memory/
│   ├── 2026-04-01.md      # 今日日誌
│   ├── 2026-03-31.md      # 昨日日誌
│   └── ...                # 歷史日誌
```

**每日日誌格式範例：**

```markdown
# 2026-04-01

## 時間線

- 09:00 期哥問關於 QS 報價的問題
- 10:30 幫忙整理了 Notion VO Tracker
- 14:00 找到了關於合約條款的資料
- 16:00 和期哥討論了項目進度

## 重要事項

- VO 報價 deadline 是 4月15日
- 期哥下星期要出差去深圳
```

**長期記憶格式範例：**

```markdown
# MEMORY.md - Long-Term Memory

## 關於期哥

- 正職：工料測量師（QS）
- 在香港做工程項目
- 偏好用廣東話溝通

## 決定記錄

- 2026-03-15：決定用 OpenClaw 管理所有通訊
- 2026-03-20：Notion 做項目管理主要工具

## 常用知識

- 合約 VO 流程：收到變更通知 → 報價 → 審批 → 落單
```

**小結：** 每日日誌是 raw log，長期記憶是精華。兩者配合使用，agent 就不會「失憶」。

---

### 10.3 記憶搜索

不需要將所有記憶都塞入 context，OpenClaw 提供了語義搜索工具：

**memory_search — 語義搜索：**

```
用 memory_search 搜尋：「上次期哥講的 VO 金額是多少？」
→ 返回相關的記憶片段
```

**memory_get — 精確讀取：**

```
用 memory_get 讀取：memory/2026-03-15.md
→ 返回該文件的完整內容
```

**搜索技術：**

OpenClaw 使用 **semantic + BM25 hybrid search**：
- Semantic search：理解語意，不是逐字 match
- BM25：傳統關鍵詞搜索
- 兩者結合，準確度高很多

**小結：** 記憶搜索令 agent 可以在大量歷史資料裡面快速找到需要的資訊，不需要全部讀完。

---

### 10.4 Long-term vs Short-term 記憶

| 特性 | Short-term（每日日誌） | Long-term（MEMORY.md） |
|------|----------------------|----------------------|
| 格式 | `memory/YYYY-MM-DD.md` | `MEMORY.md` |
| 內容 | Raw 日誌 | 策劃過的精華 |
| 加載 | 在 session 時按需讀取 | 只在主 session 注入 |
| 維護 | 自動 append | 手動策劃更新 |
| 保存期 | 永久（但舊的少用） | 持續更新 |

**何時用哪個：**

- **即時記錄** → 寫入今日的 `memory/YYYY-MM-DD.md`
- **重要決定** → 同時更新 `MEMORY.md`
- **尋找舊資料** → 用 `memory_search` 搜
- **長期知識** → 放在 `MEMORY.md`

**小結：** 短期記憶像日記，長期記憶像百科全書。兩者各有用途，配合使用才是最佳實踐。

---

### 10.5 自動記憶管理

OpenClaw 有幾個自動機制幫助管理記憶：

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
- 讀取今日和昨日的 `memory/` 日誌
- 在主 session 讀取 `MEMORY.md`

**3. 定期記憶整理（透過 Heartbeat）**

在 HEARTBEAT.md 裡面加入：

```markdown
## Memory Maintenance

每隔幾日做一次：
1. 看看近期的 memory/ 日誌
2. 將重要資訊策劃到 MEMORY.md
3. 刪除 MEMORY.md 裡面過時的東西
```

**小結：** 自動記憶管理減輕了 agent 的負擔，令記憶系統可以長期運作而不需要人手干預。

---

## 第十一章：Skills（技能）系統

### 11.1 Skills 是什麼？

一個 **Skill** 就是一個包含 `SKILL.md` 的目錄，裡面教 agent 如何做某件事情。

想像 skills 像「擴充包」——你可以加新功能給 agent，而不需要修改核心程式。

**Skills 的來源（優先次序）：**

1. `<workspace>/skills/` — 你自己寫的（最高優先）
2. `~/.openclaw/skills/` — 全局安裝的
3. `~/.agents/skills/` — Agent 專用的
4. Bundled — OpenClaw 內置的

---

### 11.2 ClawHub — 技能市場

**ClawHub** (https://clawhub.com) 是 OpenClaw 的公開技能市場，像 App Store 一樣，有大量現成的 skill 可以下載。

**瀏覽技能：**

```bash
# 搜尋技能
openclaw skills search weather

# 看技能詳情
openclaw skills info weather
```

---

### 11.3 安裝技能

**安裝：**

```bash
# 安裝技能
openclaw skills install weather

# 更新所有已安裝的技能
openclaw skills update --all

# 列出已安裝的技能
openclaw skills list
```

**配置已安裝的技能：**

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

有些 skill 需要特定條件才可以運行：
- `bins`：需要某些 command-line 工具
- `env`：需要環境變數
- `config`：需要特定配置

如果不滿足條件，skill 會顯示安裝失敗的原因。

**Token 影響：**

每個 skill 大約使用 97 個字符 + 名稱和描述的長度。裝太多 skill 會佔用 context，所以只裝需要的。

**小結：** ClawHub 令你可以快速擴展 agent 的能力，像砌積木一樣加功能。

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
description: 我的自訂技能，用來做 XYZ
version: 1.0.0
author: 你的名字
openclaw:
  requires:
    bins:
      - curl
    env:
      - API_KEY
---

# My Skill

## 用途

這個技能用來做 XYZ。

## 使用方法

當用戶要求做 XYZ 時：

1. 找到相關資料
2. 執行 ABC
3. 回傳結果

## 範例

用戶：「幫我做 XYZ」
→ 用 curl 呼叫 API → 整理結果 → 回覆用戶

## 錯誤處理

- 如果 API 不回應，等 5 秒後重試
- 重試 3 次都失敗就報告錯誤
```

**Step 3：測試技能**

```bash
# 重新載入 skills
openclaw gateway restart

# 測試
# 在對話裡面觸發你的 skill
```

**AgentSkills 格式細節：**

- **YAML frontmatter**：定義 metadata（name, description, requires 等）
- **Markdown body**：教 agent 如何使用的指令
- 支援引用相對路徑的文件

**小結：** 寫自訂 skill 並不難，核心就是一個 `SKILL.md` 文件。寫得好的 skill 文件等於教會 agent 一樣新技能。

---

### 11.5 常用技能推薦

以下是一些很實用的 skills，強烈建議安裝：

| 技能 | 用途 | 安裝命令 |
|------|------|---------|
| `weather` | 查天氣 | `openclaw skills install weather` |
| `healthcheck` | 主機安全檢查 | `openclaw skills install healthcheck` |
| `skill-creator` | 建立新技能 | `openclaw skills install skill-creator` |
| `obsidian` | 管理 Obsidian 筆記 | `openclaw skills install obsidian` |
| `nano-banana-pro` | AI 圖片生成 | `openclaw skills install nano-banana-pro` |

**小結：** 在 ClawHub 裡面有很多好用的現成技能，善用它們可以節省很多時間。

---
