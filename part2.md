## 第四章：連接 Telegram

Telegram 是最多人用的 OpenClaw channel 之一，設定亦相對簡單。

### 4.1 建立 Telegram Bot

首先，我們要在 Telegram 建立一個 Bot：

#### 步驟 1：找 BotFather

1. 打開 Telegram
2. 搜尋 `@BotFather`（有藍色 ✅ 的才是官方）
3. 按 `/start`

[截圖：Telegram BotFather 對話介面]

#### 步驟 2：建立新 Bot

輸入命令：

```
/newbot
```

BotFather 會問你：
1. **Bot 名稱** — 例如：`我的 AI 助手`
2. **Bot Username** — 必須 `bot` 結尾，例如：`my_ai_helper_bot`

#### 步驟 3：取得 Token

建立成功後，BotFather 會給你一個 **Bot Token**，格式類似：

```
123456789:ABCdefGHIjklMNOpqrSTUvwxYZ
```

> ⚠️ **這個 Token 就是你 Bot 的密碼，千萬不要公開！** 複製好它，我們下一節要用。

[截圖：BotFather 回傳 Bot Token 的訊息]

#### 步驟 4：設定 Bot Profile（可選）

你可以繼續在 BotFather 處設定：

```
/setdescription — Bot 簡介
/setabouttext — Bot 個人簡介
/setuserpic — Bot 頭像
/setcommands — Bot 命令列表
```

### 4.2 設定 OpenClaw 連接 Telegram

拿到 Token 之後，設定 OpenClaw：

#### 方法一：用 CLI（推薦）

```bash
openclaw onboard
# 選擇 Telegram → 輸入 Bot Token
```

#### 方法二：直接改配置文件

編輯 `~/.openclaw/openclaw.json`：

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123456789:ABCdefGHIjklMNOpqrSTUvwxYZ",
      dmPolicy: "pairing",
      groups: {
        "*": {
          requireMention: true,
        },
      },
    },
  },
}
```

#### 配置解釋

| 選項 | 說明 | 默認值 |
|------|------|--------|
| `enabled` | 是否啟用 Telegram | `true` |
| `botToken` | 在 BotFather 拿到的 Token | — |
| `dmPolicy` | 私訊策略 | `"pairing"` |
| `groups` | 群組設定 | — |
| `requireMention` | 群組中需要 @Bot 才回應 | `true` |

#### DM Policy 說明

`dmPolicy` 控制誰可以私訊你的 Bot：

| 值 | 說明 |
|----|------|
| `pairing` | 需要配對才可以用（推薦，最安全） |
| `allowlist` | 只有 `allowFrom` 列表中的人可以用 |
| `open` | 任何人都可以用 |
| `disabled` | 禁用私訊 |

#### 環境變數方式

你亦可以用環境變數代替在配置文件寫 Token：

```bash
export TELEGRAM_BOT_TOKEN="123456789:ABCdefGHIjklMNOpqrSTUvwxYZ"
```

```json5
{
  channels: {
    telegram: {
      enabled: true,
      // 如果用環境變數，botToken 可以不寫
      // OpenClaw 會自動讀取 TELEGRAM_BOT_TOKEN
    },
  },
}
```

#### 重啟 Gateway

設定完成後，重啟 Gateway 讓配置生效：

```bash
openclaw gateway restart
```

[截圖：openclaw gateway restart 的終端輸出]

### 4.3 測試收發訊息

Gateway 重啟之後，測試一下是否 work：

#### 步驟 1：私訊 Bot

1. 在 Telegram 搜尋你的 Bot username
2. 按 `/start`
3. 如果用 `pairing` 模式，你會收到一個配對碼

#### 步驟 2：配對（Pairing 模式）

如果用 `pairing` 模式，你需要在伺服器端 approve：

```bash
# 查看待配對的請求
openclaw pairing list telegram

# 批准配對（用收到的 CODE）
openclaw pairing approve telegram <CODE>
```

[截圖：openclaw pairing list 和 approve 的終端輸出]

#### 步驟 3：測試對話

配對成功之後，發個訊息給 Bot：

```
你好！
```

如果你收到 AI 回覆，恭喜！🎉 Telegram 連接成功！

[截圖：Telegram 同 AI Bot 的對話截圖]

#### 常見問題

| 問題 | 解決方法 |
|------|----------|
| Bot 沒有回應 | `openclaw gateway status` 檢查 Gateway 是否 running |
| 收到 error | `openclaw doctor` 診斷問題 |
| Token 錯誤 | 確認 Token 是否 copy 齊，有沒有多了空格 |
| 配對沒有反應 | `openclaw pairing list telegram` 查看有沒有 pending 請求 |

### 4.4 群組設定

OpenClaw 支援在 Telegram 群組使用。

#### 基本群組配置

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123456789:ABCdef...",
      dmPolicy: "pairing",
      groups: {
        // 所有群組的默認設定
        "*": {
          requireMention: true,  // 需要 @Bot 才回應
        },
      },
    },
  },
}
```

#### 特定群組配置

你可以為特定群組設定不同的規則：

```json5
{
  channels: {
    telegram: {
      groups: {
        // 默認：所有群組
        "*": {
          requireMention: true,
        },
        // 特定群組（用 Group ID）
        "-1001234567890": {
          requireMention: false,  // 不用 @ 就回應
          users: ["123456789"],   // 只回應這些用戶
        },
      },
    },
  },
}
```

> 💡 **怎樣找 Group ID？** 
> 1. 將 Bot 加入群組
> 2. 在群組發訊息
> 3. 去 `https://api.telegram.org/bot<TOKEN>/getUpdates`
> 4. 你會看到 `chat.id`，這個就是 Group ID（負數）

#### Forum Topics

如果你的群組是 Forum 模式（有 topic），OpenClaw 支援 topic 獨立設定：

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          requireMention: true,
          topics: {
            "123": {  // Topic ID
              requireMention: false,
            },
          },
        },
      },
    },
  },
}
```

### 4.5 進階設定

#### Streaming 模式

控制 AI 回覆的「打字效果」：

```json5
{
  channels: {
    telegram: {
      streaming: "partial",  // partial | block | off
    },
  },
}
```

| 值 | 效果 |
|----|------|
| `partial` | 逐段顯示（默認，體驗最好） |
| `block` | 等完整回覆才一次過顯示 |
| `off` | 關閉 streaming |

#### Allowlist 模式

如果你想只給特定人用 Bot：

```json5
{
  channels: {
    telegram: {
      dmPolicy: "allowlist",
      allowFrom: [
        "123456789",   // 用戶的 Telegram ID
        "987654321",
      ],
    },
  },
}
```

> 💡 **怎樣找自己的 Telegram ID？** 發訊息給 `@userinfobot` 或者 `@getmyid_bot`。

### 第四章小結

- 建立 Bot：找 @BotFather → `/newbot` → 拿 Token
- 設定 OpenClaw：配置文件加 `channels.telegram` 或用 CLI
- Pairing 模式最安全：`openclaw pairing approve telegram <CODE>`
- 群組默認要 @Bot 才回應，可以改 `requireMention`
- 建議用環境變數儲存 Token

---

## 第五章：連接 WhatsApp

WhatsApp 連接稍微複雜一些，因為它基於 WhatsApp Web 協議（Baileys）。

### 5.1 準備工作

#### 重要注意事項

> ⚠️ **WhatsApp 連接有風險：**
> - 基於逆向工程的 WhatsApp Web 協議
> - WhatsApp 可能會 ban 非官方客戶端的號碼
> - **強烈建議用一個獨立電話號碼**，不要用你日常用的號碼
> - 風險自負

#### 你需要準備：

| 項目 | 說明 |
|------|------|
| 📱 獨立電話號碼 | 建議用預付卡或新號碼 |
| 📱 一部手機 | 用來掃 QR Code |
| 🌐 穩定網絡 | 伺服器需要穩定連線 |

### 5.2 安裝 WhatsApp 插件

WhatsApp 是以插件形式提供，需要先安裝：

```bash
# 安裝 WhatsApp 插件
openclaw plugins install @openclaw/whatsapp
```

安裝完成後，確認插件已啟用：

```bash
# 查看已安裝的插件
openclaw plugins list
```

[截圖：openclaw plugins list 的輸出，顯示 whatsapp 插件]

### 5.3 掃碼配對

#### 步驟 1：開始登入

```bash
openclaw channels login --channel whatsapp
```

執行後，終端會顯示一個 QR Code。

[截圖：終端顯示 WhatsApp QR Code]

#### 步驟 2：用手機掃碼

1. 用你準備的獨立電話號碼的 WhatsApp
2. 去 **設定 → 已關聯裝置 → 關聯裝置**
3. 掃描終端上的 QR Code

[截圖：WhatsApp「已關聯裝置」介面]

#### 步驟 3：確認連接

掃碼成功後，終端會顯示類似：

```
✅ WhatsApp connected successfully!
```

#### 步驟 4：啟動 Gateway

```bash
openclaw gateway start
# 或者如果已經 running 著：
openclaw gateway restart
```

### 5.4 權限管理

WhatsApp 的權限管理比較重要，因為你不想讓所有人用你的 AI。

#### DM（私訊）配置

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",    // pairing | allowlist | open | disabled
      allowFrom: [
        "+85291234567",       // 用國際格式電話號碼
        "+85298765432",
      ],
    },
  },
}
```

#### Pairing 模式操作

同 Telegram 差不多：

```bash
# 查看待配對請求
openclaw pairing list whatsapp

# 批准配對
openclaw pairing approve whatsapp <CODE>
```

### 5.5 群組管理

#### 群組配置

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+85291234567"],
      groupPolicy: "allowlist",      // allowlist | open | disabled
      groupAllowFrom: [
        "+85291234567",              // 只有這些人可以在群組同 Bot 互動
      ],
    },
  },
}
```

#### Group Policy 選項

| 值 | 說明 |
|----|------|
| `allowlist` | 只有 `groupAllowFrom` 列表中的人可以用 |
| `open` | 群組中所有人都可以用 |
| `disabled` | 禁用群組功能 |

> 💡 **Tips：** 
> - WhatsApp 群組沒有 `requireMention` 選項，因為 WhatsApp 沒有 @bot 的概念
> - 如果 `groupPolicy` 是 `open`，Bot 會回應群組中每一個訊息 — 小心 token 燒錢！
> - 建議用 `allowlist` 模式，精確控制誰可以觸發 Bot

#### 完整配置範例

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: [
        "+85291234567",    // 期哥
        "+85267890123",    // 阿強
      ],
      groupPolicy: "allowlist",
      groupAllowFrom: [
        "+85291234567",    // 只有期哥可以在群組觸發 Bot
      ],
    },
  },
}
```

### 第五章小結

- WhatsApp 基於 WhatsApp Web 協議，有被 ban 風險
- **強烈建議用獨立電話號碼**
- 安裝插件 → 掃碼配對 → 設定權限
- 用 `allowlist` 模式控制權限最安全
- 群組沒有 `requireMention`，用 `groupPolicy` 控制

---

## 第六章：連接 Discord

Discord 是另一個非常受歡迎的平台，特別是社群和團隊使用。

### 6.1 建立 Discord Bot

首先，去 Discord Developer Portal 建立 Bot：

#### 步驟 1：去 Developer Portal

1. 打開 [discord.com/developers/applications](https://discord.com/developers/applications)
2. 登入你的 Discord 帳號
3. 按 **New Application**

[截圖：Discord Developer Portal「New Application」按鈕]

#### 步驟 2：建立 Application

1. 輸入 Application 名稱（例如：`我的 AI 助手`）
2. 按 **Create**

#### 步驟 3：建立 Bot

1. 左側欄選擇 **Bot**
2. 你會看到 Bot 已經自動建立了

[截圖：Discord Developer Portal Bot 頁面]

#### 步驟 4：取得 Token

1. 在 Bot 頁面，找到 **Token** 區域
2. 按 **Reset Token**（或 **Copy** 如果有）
3. 複製 Token

> ⚠️ **Token 等同密碼，不要公開！**

### 6.2 設定權限

#### 開啟 Intents

在 Bot 頁面，找到 **Privileged Gateway Intents** 區域，開啟：

- ✅ **Message Content Intent** — 讀取訊息內容
- ✅ **Server Members Intent** — 識別伺服器成員

[截圖：Discord Bot Intents 設定頁面]

> ⚠️ **兩個 Intent 都必須開啟，否則 OpenClaw 收不到訊息！**

#### 設定 OAuth 權限

1. 左側欄選擇 **OAuth2 → URL Generator**
2. **Scopes** 勾選：
   - ✅ `bot`
   - ✅ `applications.commands`
3. **Bot Permissions** 勾選：
   - ✅ View Channels
   - ✅ Send Messages
   - ✅ Read Message History
   - ✅ Embed Links
   - ✅ Attach Files
   - ✅ Add Reactions

[截圖：Discord OAuth2 URL Generator 設定]

#### 邀請 Bot 進伺服器

1. 複製底部生成的 URL
2. 在瀏覽器打開
3. 選擇你想加入的伺服器
4. 按 **Authorize**

[截圖：Discord Bot 授權邀請介面]

### 6.3 連接 OpenClaw

#### 方法一：用環境變數（推薦）

```bash
# 加到 ~/.bashrc 或 ~/.zshrc
export DISCORD_BOT_TOKEN="your-bot-token-here"
```

然後配置 OpenClaw：

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: {
        source: "env",
        provider: "default",
        id: "DISCORD_BOT_TOKEN",
      },
    },
  },
}
```

#### 方法二：直接寫在配置文件

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: {
        source: "env",
        provider: "default",
        id: "DISCORD_BOT_TOKEN",
      },
    },
  },
}
```

> 💡 **建議用環境變數**，比直接寫 token 在配置文件更安全。

#### 重啟 Gateway

```bash
openclaw gateway restart
```

#### 測試

去 Discord，在一個 channel 發訊息給 Bot。如果你收到 AI 回覆，成功！🎉

[截圖：Discord channel 同 AI Bot 的對話截圖]

### 6.4 Guild 頻道管理

Guild 就是 Discord 伺服器。你可以為每個伺服器設定不同的規則。

#### 基本配置

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: {
        source: "env",
        provider: "default",
        id: "DISCORD_BOT_TOKEN",
      },
      groupPolicy: "allowlist",
      guilds: {
        "123456789012345": {  // Server ID（數字）
          requireMention: true,
          users: [
            "987654321098765",  // User ID（數字）
          ],
        },
      },
    },
  },
}
```

#### 怎樣找 Server ID 和 User ID

1. 打開 Discord **設定**
2. 去 **進階** → 開啟 **開發者模式**
3. 右鍵點擊伺服器名稱 → **複製伺服器 ID**（就是 Server ID）
4. 右鍵點擊用戶名稱 → **複製用戶 ID**（就是 User ID）

[截圖：Discord 右鍵選單「複製伺服器 ID」]

#### 多個伺服器配置

```json5
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        // 伺服器 A：工作群組
        "111111111111111": {
          requireMention: true,
          users: ["222222222222222", "333333333333333"],
        },
        // 伺服器 B：朋友群組
        "444444444444444": {
          requireMention: false,
          users: ["222222222222222"],
        },
      },
    },
  },
}
```

### 6.5 進階功能

#### Thread-bound Sessions

OpenClaw Discord 支援 **Thread-bound sessions** — 當有人在一個 thread 中同 Bot 對話，這個 thread 就是一個獨立的 session。

好處：
- 不同話題不會搞亂
- 上下文清晰
- 方便回顧對話歷史

#### Bot 狀態設定

你可以自訂 Bot 的在線狀態：

```json5
{
  channels: {
    discord: {
      enabled: true,
      // ... token etc ...
      status: "online",         // online | idle | dnd | invisible
      activity: {
        type: "playing",        // playing | streaming | listening | watching | custom
        name: "with AI 🤖",
      },
    },
  },
}
```

#### Channel 特定配置

```json5
{
  channels: {
    discord: {
      guilds: {
        "123456789012345": {
          requireMention: true,
          users: ["987654321098765"],
          channels: {
            // 特定 channel 的設定
            "111222333444555": {
              requireMention: false,  // 這個 channel 不用 @
            },
          },
        },
      },
    },
  },
}
```

### 第六章小結

- 建立 Bot：Developer Portal → New Application → Bot → 複製 Token
- **必須開啟 Message Content Intent + Server Members Intent**
- OAuth scopes：`bot` + `applications.commands`
- 用環境變數儲存 Token 最安全
- `Developer Mode` → 右鍵複製 Server ID / User ID
- Thread-bound sessions 幫你組織對話

---

## 📋 Part 1 總結

恭喜你看到這裡！🎉 以下是 Part 1 的重點回顧：

### 你學到的東西

| 章節 | 重點 |
|------|------|
| **第一章** | OpenClaw 是 self-hosted AI agent gateway，四大核心概念：Gateway、Agent、Channel、Session |
| **第二章** | 安裝順序：Node.js → OpenClaw → onboard → 系統服務 |
| **第三章** | 模型格式 `provider/model`，用環境變數存 API Key，設定 fallback |
| **第四章** | Telegram：@BotFather 建 Bot → 配置 → pairing |
| **第五章** | WhatsApp：安裝插件 → 掃碼配對 → 用獨立號碼 |
| **第六章** | Discord：Developer Portal → Bot → 開 Intents → OAuth |

### 下一步

繼續看 **Part 2**，你會學到：

- 進階配置（自訂 Agent 行為、工具權限）
- 安全設定（防火牆、認證）
- 常見問題排解
- 更多實用 Tips

---

> 📝 **這份教程由阿星 ⭐ 為你撰寫。** 如果有任何問題，歡迎去 [OpenClaw GitHub](https://github.com/nicepkg/openclaw) 提 Issue 或者在 Discord 社群討論。
