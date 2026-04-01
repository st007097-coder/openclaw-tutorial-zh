# OpenClaw Beginner Tutorial 🦞 — Build Your AI Assistant From Scratch

> **Version:** 2026.3.28 ｜ **Author:** 阿星 ⭐ ｜ **Language:** English (translated)

---

## Introduction

Hello! Welcome to the OpenClaw beginner tutorial.

Have you ever thought: "It would be great if I could chat with AI directly on WhatsApp / Telegram / Discord, right?" OpenClaw is built exactly for this — it's a **self-hosted** AI agent gateway that connects your chat apps with AI models.

### Who is this tutorial for?

- ✅ Developers / tech enthusiasts who want to build their own AI assistant
- ✅ People who want to use AI on Telegram / WhatsApp / Discord
- ✅ People who want to control their own data and not rely on third-party services
- ✅ People with basic knowledge of Node.js

### What do you need?

| Item | Description |
|------|-------------|
| 💻 A computer / server | Linux, macOS, or Windows all work |
| 🌐 Stable internet | VPS or an always-on machine is recommended |
| 🔑 AI Model API Key | At least one: Anthropic, OpenAI, Google, etc. |
| 📱 Chat app account | Telegram / WhatsApp / Discord, etc. |
| ⏱️ About 30-60 minutes | Follow the tutorial step by step |

### Tutorial Structure

This tutorial is divided into two parts:

- **Part 1 (this one):** Introduction → Getting to know OpenClaw → Installation → Model setup → Telegram → WhatsApp → Discord
- **Part 2 (next one):** Advanced features → Security → Custom Agents → FAQ

Ready? Let's go! 🚀

---

## Chapter 1: Getting to Know OpenClaw

### 1.1 What is OpenClaw?

OpenClaw (🦞) is an **open-source (MIT License)** self-hosted AI agent gateway.

In simple terms, it does three things:

1. **Connect chat apps** — WhatsApp, Telegram, Discord, iMessage, Signal, Slack, Google Chat, and more
2. **Connect AI models** — Anthropic Claude, OpenAI GPT, Google Gemini, OpenRouter, and 35+ providers
3. **Manage conversations** — Sessions, memory, permissions, security — all handled by the Gateway

> 💡 **One-line summary:** OpenClaw is a matchmaker that connects your chat apps with AI models.

```
You (WhatsApp/Telegram/Discord) ←→ OpenClaw Gateway ←→ AI Model (Claude/GPT/Gemini)
```

[Screenshot: OpenClaw Dashboard homepage]

### 1.2 OpenClaw vs Other AI Tools

You might ask: "There are already lots of AI chat tools out there — why use OpenClaw?"

| Feature | OpenClaw | ChatGPT | DIY Telegram Bot |
|---------|----------|---------|-------------------|
| Multi-platform support | ✅ 8+ platforms | ❌ Own app only | ❌ Telegram only |
| Self-hosted | ✅ Your server | ❌ Cloud service | ✅ |
| Model freedom | ✅ 35+ providers | ❌ GPT only | ⚠️ Write it yourself |
| Group support | ✅ Built-in | ❌ | ⚠️ Write it yourself |
| Permission management | ✅ Built-in | ❌ | ⚠️ Write it yourself |
| Open source | ✅ MIT | ❌ | Up to you |
| Difficulty | 🟢 Medium | 🟢 Easy | 🔴 Hard |

**OpenClaw's advantages:**
- One Gateway manages all platforms — no need to write code for each one
- Control your own data — no privacy worries
- Open source and free, with an active community

### 1.3 Core Concepts

Before we start, we need to understand four core concepts:

#### 🔌 Gateway

The Gateway is the **core engine** of OpenClaw. It's responsible for:

- Managing connections to all chat platforms
- Sending and receiving messages
- Calling AI models
- Providing the Web UI (Dashboard)

By default, it runs on `http://127.0.0.1:18789/`.

#### 🤖 Agent

The Agent is your AI assistant itself. It's responsible for:

- Understanding user messages
- Deciding how to respond
- Using tools (web search, file read, etc.)
- Managing memory

You can configure different Agent behaviors, models, tool permissions, and more.

#### 📡 Channel

A Channel is the **connection path** between the Gateway and a chat app. Each chat platform is one Channel:

- `telegram` — Telegram Bot
- `whatsapp` — WhatsApp Web
- `discord` — Discord Bot
- `signal` — Signal
- And more

#### 💬 Session

Every time someone chats with your AI assistant, a Session is created. The Session manages:

- Conversation history
- Context
- User identity

> 💡 **A simple analogy:**
> - **Gateway** = Restaurant manager
> - **Agent** = Waiter (greeting and serving customers)
> - **Channel** = Different entrances (front door, side door, back door)
> - **Session** = A table (where customers sit and chat)

### 1.4 Architecture Diagram

Here's the overall architecture of OpenClaw:

```mermaid
graph TB
    subgraph "User Side"
        TG[📱 Telegram]
        WA[📱 WhatsApp]
        DC[🖥️ Discord]
        IM[📱 iMessage]
        WEB[🌐 WebChat]
    end

    subgraph "OpenClaw Gateway"
        GW[🦞 Gateway<br/>Port 18789]
        AGENT[🤖 Agent]
        SESSION[💬 Session Manager]
        TOOLS[🔧 Tools]
    end

    subgraph "AI Models"
        ANTHROPIC[Anthropic Claude]
        OPENAI[OpenAI GPT]
        GOOGLE[Google Gemini]
        LOCAL[Local Model Ollama]
    end

    TG --> GW
    WA --> GW
    DC --> GW
    IM --> GW
    WEB --> GW

    GW --> AGENT
    AGENT --> SESSION
    AGENT --> TOOLS

    AGENT --> ANTHROPIC
    AGENT --> OPENAI
    AGENT --> GOOGLE
    AGENT --> LOCAL
```

**Data flow:**

1. User sends a message on Telegram
2. Telegram API sends the message to the OpenClaw Gateway
3. Gateway finds the corresponding Agent and Session
4. Agent sends the message + historical context to the AI model
5. AI model replies, Agent processes it, and sends it back to Telegram via Gateway
6. User receives the AI reply

**Client connections (for you to manage):**

```mermaid
graph LR
    subgraph "Management Side"
        CLI[🖥️ CLI]
        MAC[📱 macOS App]
        IOS[📱 iOS App]
        AND[📱 Android App]
    end

    CLI -->|WebSocket| GW[🦞 Gateway]
    MAC -->|WebSocket| GW
    IOS -->|WebSocket| GW
    AND -->|WebSocket| GW
```

### Chapter 1 Summary

- OpenClaw is a self-hosted AI agent gateway — open source and free
- Four core concepts: **Gateway** (engine), **Agent** (assistant), **Channel** (connection), **Session** (conversation)
- Supports 8+ chat platforms and 35+ AI model providers
- Your data stays in your hands — privacy guaranteed

---

## Chapter 2: Installing OpenClaw

### 2.1 System Requirements

Before installing, check your environment:

| Item | Minimum | Recommended |
|------|---------|-------------|
| OS | Linux / macOS / Windows | Linux (Ubuntu/Debian) |
| Node.js | v22.14+ | v24 (recommended) |
| Memory | 512MB | 1GB+ |
| Disk space | 500MB | 1GB+ |
| Network | Stable connection | Public IP / VPS |

> ⚠️ **Note:** If you want WhatsApp or other platforms to receive messages 24/7, use a VPS or a machine that's always on.

### 2.2 Installing Node.js

OpenClaw requires Node.js v22.14 or above. Here are installation methods for each platform:

#### Linux (Ubuntu/Debian)

```bash
# Method 1: Using nvm (recommended, easy version switching)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
source ~/.bashrc
nvm install 24
nvm use 24

# Verify
node -v   # Should show v24.x.x
npm -v    # Should show 11.x.x
```

```bash
# Method 2: Using NodeSource (good for server environments)
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo bash -
sudo apt-get install -y nodejs
```

#### macOS

```bash
# Method 1: Using nvm (recommended)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
source ~/.zshrc
nvm install 24

# Method 2: Using Homebrew
brew install node@24
```

#### Windows

```powershell
# Method 1: Using nvm-windows
# Go to https://github.com/coreybutler/nvm-windows/releases to download the installer
nvm install 24
nvm use 24

# Method 2: Download Node.js installer directly
# Go to https://nodejs.org to download the LTS version
```

[Screenshot: Terminal output of node -v and npm -v]

#### ✅ Verify Node.js Installation

```bash
node -v   # Expected: v24.x.x or v22.x.x
npm -v    # Expected: 10.x.x or 11.x.x
```

> 💡 **Tips:** If you see `command not found`, try opening a new terminal or run `source ~/.bashrc`.

### 2.3 Installing OpenClaw

Once Node.js is ready, installing OpenClaw is super easy.

#### Method 1: One-Line Install Script (Recommended)

**Linux / macOS:**

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

**Windows (PowerShell, run as Administrator):**

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

This script will automatically:
1. Check your Node.js version
2. Install the OpenClaw CLI
3. Set up basic configuration

#### Method 2: Manual Install via npm

```bash
npm install -g openclaw@latest
```

#### ✅ Verify Installation

```bash
openclaw --version
# Expected output: 2026.3.28 or newer

openclaw doctor
# Checks if your environment is complete
```

[Screenshot: Output of openclaw --version and openclaw doctor]

> 💡 **Tips:** If the `openclaw` command is not found, check that your PATH includes the npm global directory:
> ```bash
> npm config get prefix
> # Make sure this path is in your $PATH
> ```

### 2.4 Initial Setup (Onboard)

After installation, run the onboard wizard for initial setup:

```bash
openclaw onboard --install-daemon
```

This wizard will guide you through:

1. **Choose AI model provider** — Pick the provider you want to use (Anthropic / OpenAI / Google, etc.)
2. **Enter API Key** — Your model API key
3. **Configure Gateway** — Port, host, and other basic settings
4. **Install Daemon** — Install OpenClaw as a system service

[Screenshot: Onboard wizard interactive interface]

Just follow the prompts step by step. If you don't want to set up a platform right now, you can skip it and add it later.

#### Manual Setup (Advanced Users)

If you want to configure manually, the config file is at `~/.openclaw/openclaw.json` (JSON5 format). Here's the basic structure:

```json5
{
  // OpenClaw configuration file
  // Format: JSON5 (supports comments, trailing commas)

  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-6",
        fallbacks: ["openai/gpt-5.2"],
      },
    },
  },

  channels: {
    // Telegram, WhatsApp, Discord configs will go here later
  },
}
```

### 2.5 Install as System Service

If you want OpenClaw to start automatically on boot (highly recommended), install it as a system service:

#### Linux (systemd)

```bash
# The onboard wizard does this automatically
# To do it manually:
openclaw gateway install

# Start the service
sudo systemctl start openclaw-gateway
sudo systemctl enable openclaw-gateway

# Check status
sudo systemctl status openclaw-gateway
```

#### macOS (launchd)

```bash
openclaw gateway install
# Will automatically create a LaunchAgent
```

#### Or Use OpenClaw's Quick Commands

```bash
# Check Gateway status
openclaw gateway status

# Start Gateway
openclaw gateway start

# Stop Gateway
openclaw gateway stop

# Restart Gateway
openclaw gateway restart
```

[Screenshot: Output of openclaw gateway status]

### 2.6 Verify Successful Installation

After completing the steps above, do a full check:

```bash
# 1. Check version
openclaw --version

# 2. Run Doctor diagnostics
openclaw doctor

# 3. Check Gateway status
openclaw gateway status

# 4. Open Dashboard
openclaw dashboard
# Browser will auto-open http://127.0.0.1:18789/
```

If `openclaw doctor` shows all ✅, the installation is successful!

[Screenshot: Dashboard homepage and Doctor diagnostic results]

> 💡 **Tips:**
> - If Doctor finds issues, try `openclaw doctor --fix` for automatic repair
> - If the Gateway won't start, check if port 18789 is already in use: `lsof -i :18789`
> - Remember to open port 18789 in your firewall (if you want remote access to the Dashboard)

### Chapter 2 Summary

- Installation order: **Node.js → OpenClaw → onboard → Install service → Verify**
- The one-line install script is the fastest and easiest
- `openclaw doctor` is your best friend — run it when you have problems
- Remember to install as a system service so the Gateway starts on boot

---

## Chapter 3: AI Model Provider Setup

OpenClaw supports over 35 AI model providers — you can freely choose and mix them.

### 3.1 Common Provider Overview

| Provider | Example Models | Features | Pricing |
|----------|---------------|----------|---------|
| **Anthropic** | Claude Sonnet, Claude Opus | Long context, safe | Medium-High |
| **OpenAI** | GPT-5.2, GPT-4o | All-rounder, great ecosystem | Medium-High |
| **Google** | Gemini 2.5 Pro | Multimodal, good value | Medium |
| **OpenRouter** | Aggregates 100+ models | One API for many models | Variable |
| **Ollama** | Llama, Mistral, etc. | Runs locally, free | Free |

The model format is `provider/model`, for example:
- `anthropic/claude-sonnet-4-6`
- `openai/gpt-5.2`
- `google/gemini-2.5-pro`
- `openrouter/anthropic/claude-sonnet-4-6`

### 3.2 How to Choose a Model

Factors to consider when choosing a model:

| Use Case | Recommended Model | Reason |
|----------|-------------------|--------|
| Daily chat | Claude Sonnet / GPT-4o | Fast response, good quality |
| Complex reasoning | Claude Opus / GPT-5.2 | Strong reasoning ability |
| Budget-friendly | Gemini Flash / OpenRouter | Cheap, good enough |
| Privacy-sensitive | Ollama local models | Data never leaves your machine |
| Coding | Claude Sonnet / GPT-5.2 | High code quality |

> 💡 **Advice for beginners:** Start with just one model, like `anthropic/claude-sonnet-4-6`. Once it's stable, add fallbacks and other providers.

### 3.3 API Key Setup

Each provider needs its own API Key. Here's how to get them:

#### Anthropic

1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Sign up / Log in
3. Settings → API Keys → Create Key
4. Copy the Key (format: `sk-ant-...`)

#### OpenAI

1. Go to [platform.openai.com](https://platform.openai.com)
2. Sign up / Log in
3. API Keys → Create new secret key
4. Copy the Key (format: `sk-...`)

#### Google (Gemini)

1. Go to [aistudio.google.com](https://aistudio.google.com)
2. Get API Key → Create Key
3. Copy the Key

#### Setup Methods

There are two ways to set up API Keys:

**Method 1: Environment Variables (Recommended)**

```bash
# Add to ~/.bashrc or ~/.zshrc
export ANTHROPIC_API_KEY="sk-ant-your-key-here"
export OPENAI_API_KEY="sk-your-key-here"
export GEMINI_API_KEY="your-key-here"
```

**Method 2: Configuration File**

```json5
{
  providers: {
    anthropic: {
      apiKey: "sk-ant-your-key-here",
    },
    openai: {
      apiKey: "sk-your-key-here",
    },
  },
}
```

> ⚠️ **Security warning:** API Keys are like passwords — never upload them to GitHub or public places. Using environment variables is the safest approach.

### 3.4 Model Failover Configuration

Failover means: when the primary model is unavailable (API down, quota exceeded, etc.), it automatically switches to a backup model.

```json5
{
  agents: {
    defaults: {
      model: {
        // Primary model
        primary: "anthropic/claude-sonnet-4-6",
        // Backup models (tried in order)
        fallbacks: [
          "openai/gpt-5.2",
          "google/gemini-2.5-pro",
        ],
      },
    },
  },
}
```

**How it works:**

1. Try the `primary` model first
2. If primary fails (429 rate limit, 500 server error, etc.)
3. Automatically switch to `fallbacks[0]`
4. If fallbacks[0] also fails, try `fallbacks[1]`
5. Only return an error if everything fails

> 💡 **Tips:** It's recommended to set at least one fallback model, especially if you're using a provider that often has rate limits.

### 3.5 Local Models (Ollama)

If you care about privacy or want to save money, you can use Ollama to run local models.

#### Install Ollama

```bash
# Linux / macOS
curl -fsSL https://ollama.ai/install.sh | bash

# Download a model (e.g. Llama 3.1)
ollama pull llama3.1

# Confirm Ollama is running
ollama list
```

#### Set Up Ollama in OpenClaw

```json5
{
  providers: {
    ollama: {
      baseUrl: "http://localhost:11434",
    },
  },
  agents: {
    defaults: {
      model: {
        primary: "ollama/llama3.1",
      },
    },
  },
}
```

> ⚠️ **Note:** Local models need good hardware. At least 16GB RAM is recommended for 7B models, and 32GB+ for 13B+ models.

### Chapter 3 Summary

- Model format: `provider/model` (e.g. `anthropic/claude-sonnet-4-6`)
- API Keys are best stored as environment variables — safer than in config files
- Setting up fallback models prevents service disruption when an API goes down
- Local models (Ollama) are great for privacy-conscious or budget-conscious users

---
## Chapter 4: Connecting Telegram

Telegram is one of the most popular OpenClaw channels, and setup is relatively straightforward.

### 4.1 Creating a Telegram Bot

First, we need to create a Bot on Telegram:

#### Step 1: Find BotFather

1. Open Telegram
2. Search for `@BotFather` (only the one with the blue ✅ is official)
3. Press `/start`

[Screenshot: Telegram BotFather conversation interface]

#### Step 2: Create a New Bot

Enter the command:

```
/newbot
```

BotFather will ask you:
1. **Bot Name** — for example: `My AI Assistant`
2. **Bot Username** — must end with `bot`, for example: `my_ai_helper_bot`

#### Step 3: Get the Token

After successful creation, BotFather will give you a **Bot Token**, in a format like:

```
123456789:ABCdefGHIjklMNOpqrSTUvwxYZ
```

> ⚠️ **This Token is your Bot's password — never make it public!** Copy it, we'll need it in the next section.

[Screenshot: BotFather returning the Bot Token message]

#### Step 4: Set Bot Profile (Optional)

You can continue setting up in BotFather:

```
/setdescription — Bot description
/setabouttext — Bot about info
/setuserpic — Bot avatar
/setcommands — Bot command list
```

### 4.2 Setting Up OpenClaw to Connect to Telegram

Once you have the Token, set up OpenClaw:

#### Method 1: Using CLI (Recommended)

```bash
openclaw onboard
# Choose Telegram → Enter Bot Token
```

#### Method 2: Edit Config File Directly

Edit `~/.openclaw/openclaw.json`:

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

#### Config Explanation

| Option | Description | Default |
|--------|-------------|---------|
| `enabled` | Whether to enable Telegram | `true` |
| `botToken` | The Token from BotFather | — |
| `dmPolicy` | Direct message policy | `"pairing"` |
| `groups` | Group settings | — |
| `requireMention` | Whether @Bot is required to respond in groups | `true` |

#### DM Policy Explanation

`dmPolicy` controls who can DM your Bot:

| Value | Description |
|-------|-------------|
| `pairing` | Must be paired first to use (recommended, safest) |
| `allowlist` | Only people in the `allowFrom` list can use it |
| `open` | Anyone can use it |
| `disabled` | DMs are disabled |

#### Environment Variable Method

You can also use environment variables instead of writing the Token in the config file:

```bash
export TELEGRAM_BOT_TOKEN="123456789:ABCdefGHIjklMNOpqrSTUvwxYZ"
```

```json5
{
  channels: {
    telegram: {
      enabled: true,
      // If using env vars, botToken doesn't need to be written
      // OpenClaw will automatically read TELEGRAM_BOT_TOKEN
    },
  },
}
```

#### Restart the Gateway

After configuration, restart the Gateway for the changes to take effect:

```bash
openclaw gateway restart
```

[Screenshot: Terminal output of openclaw gateway restart]

### 4.3 Testing Message Sending and Receiving

After restarting the Gateway, let's test if it works:

#### Step 1: DM the Bot

1. Search for your Bot username on Telegram
2. Press `/start`
3. If using `pairing` mode, you'll receive a pairing code

#### Step 2: Pairing (Pairing Mode)

If using `pairing` mode, you need to approve it on the server side:

```bash
# View pending pairing requests
openclaw pairing list telegram

# Approve pairing (using the received CODE)
openclaw pairing approve telegram <CODE>
```

[Screenshot: Terminal output of openclaw pairing list and approve]

#### Step 3: Test the Conversation

After successful pairing, send a message to the Bot:

```
Hello!
```

If you get an AI reply, congratulations! 🎉 Telegram connection successful!

[Screenshot: Conversation between Telegram and AI Bot]

#### Common Issues

| Problem | Solution |
|---------|----------|
| Bot not responding | Check if Gateway is running with `openclaw gateway status` |
| Receiving errors | Run `openclaw doctor` to diagnose issues |
| Token error | Make sure the Token is fully copied and no extra spaces |
| Pairing not responding | Run `openclaw pairing list telegram` to check for pending requests |

### 4.4 Group Settings

OpenClaw supports usage in Telegram groups.

#### Basic Group Configuration

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123456789:ABCdef...",
      dmPolicy: "pairing",
      groups: {
        // Default settings for all groups
        "*": {
          requireMention: true,  // @Bot required to respond
        },
      },
    },
  },
}
```

#### Specific Group Configuration

You can set different rules for specific groups:

```json5
{
  channels: {
    telegram: {
      groups: {
        // Default: all groups
        "*": {
          requireMention: true,
        },
        // Specific group (using Group ID)
        "-1001234567890": {
          requireMention: false,  // No @ required to respond
          users: ["123456789"],   // Only respond to these users
        },
      },
    },
  },
}
```

> 💡 **How to find the Group ID?**
> 1. Add the Bot to the group
> 2. Send a message in the group
> 3. Go to `https://api.telegram.org/bot<TOKEN>/getUpdates`
> 4. You'll see `chat.id` — that's the Group ID (negative number)

#### Forum Topics

If your group is in Forum mode (with topics), OpenClaw supports independent topic settings:

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

### 4.5 Advanced Settings

#### Streaming Mode

Controls the "typing effect" of AI replies:

```json5
{
  channels: {
    telegram: {
      streaming: "partial",  // partial | block | off
    },
  },
}
```

| Value | Effect |
|-------|--------|
| `partial` | Display section by section (default, best experience) |
| `block` | Wait for the complete reply before displaying |
| `off` | Disable streaming |

#### Allowlist Mode

If you want only specific people to use the Bot:

```json5
{
  channels: {
    telegram: {
      dmPolicy: "allowlist",
      allowFrom: [
        "123456789",   // User's Telegram ID
        "987654321",
      ],
    },
  },
}
```

> 💡 **How to find your Telegram ID?** Send a message to `@userinfobot` or `@getmyid_bot`.

### Chapter 4 Summary

- Create Bot: Find @BotFather → `/newbot` → Get Token
- Set up OpenClaw: Add `channels.telegram` to config or use CLI
- Pairing mode is safest: `openclaw pairing approve telegram <CODE>`
- Groups require @Bot by default, can change with `requireMention`
- Recommended to store Token in environment variables

---

## Chapter 5: Connecting WhatsApp

WhatsApp connection is slightly more complex because it's based on the WhatsApp Web protocol (Baileys).

### 5.1 Preparation

#### Important Notes

> ⚠️ **WhatsApp connection has risks:**
> - Based on reverse-engineered WhatsApp Web protocol
> - WhatsApp may ban numbers using unofficial clients
> - **Strongly recommended to use a separate phone number** — don't use your daily number
> - Use at your own risk

#### What You Need:

| Item | Description |
|------|-------------|
| 📱 Separate phone number | Recommended to use a prepaid SIM or new number |
| 📱 A phone | To scan QR Code |
| 🌐 Stable network | Server needs stable connection |

### 5.2 Installing WhatsApp Plugin

WhatsApp is provided as a plugin and needs to be installed first:

```bash
# Install WhatsApp plugin
openclaw plugins install @openclaw/whatsapp
```

After installation, verify the plugin is enabled:

```bash
# View installed plugins
openclaw plugins list
```

[Screenshot: Output of openclaw plugins list showing whatsapp plugin]

### 5.3 QR Code Pairing

#### Step 1: Start Login

```bash
openclaw channels login --channel whatsapp
```

After running, the terminal will display a QR Code.

[Screenshot: Terminal showing WhatsApp QR Code]

#### Step 2: Scan with Phone

1. Open WhatsApp on your separate phone number
2. Go to **Settings → Linked Devices → Link a Device**
3. Scan the QR Code on the terminal

[Screenshot: WhatsApp "Linked Devices" interface]

#### Step 3: Confirm Connection

After successful scanning, the terminal will show something like:

```
✅ WhatsApp connected successfully!
```

#### Step 4: Start the Gateway

```bash
openclaw gateway start
# Or if already running:
openclaw gateway restart
```

### 5.4 Permission Management

WhatsApp permission management is important because you don't want everyone using your AI.

#### DM (Direct Message) Configuration

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",    // pairing | allowlist | open | disabled
      allowFrom: [
        "+85291234567",       // Use international format phone numbers
        "+85298765432",
      ],
    },
  },
}
```

#### Pairing Mode Operations

Similar to Telegram:

```bash
# View pending pairing requests
openclaw pairing list whatsapp

# Approve pairing
openclaw pairing approve whatsapp <CODE>
```

### 5.5 Group Management

#### Group Configuration

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+85291234567"],
      groupPolicy: "allowlist",      // allowlist | open | disabled
      groupAllowFrom: [
        "+85291234567",              // Only these people can interact with Bot in groups
      ],
    },
  },
}
```

#### Group Policy Options

| Value | Description |
|-------|-------------|
| `allowlist` | Only people in the `groupAllowFrom` list can use it |
| `open` | Everyone in the group can use it |
| `disabled` | Disable group functionality |

> 💡 **Tips:**
> - WhatsApp groups don't have a `requireMention` option because WhatsApp doesn't have the @bot concept
> - If `groupPolicy` is `open`, the Bot will respond to every message in the group — be careful of token costs!
> - Recommended to use `allowlist` mode for precise control over who can trigger the Bot

#### Complete Configuration Example

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: [
        "+85291234567",    // Ah Kei
        "+85267890123",    // Ah Keung
      ],
      groupPolicy: "allowlist",
      groupAllowFrom: [
        "+85291234567",    // Only Ah Kei can trigger Bot in groups
      ],
    },
  },
}
```

### Chapter 5 Summary

- WhatsApp is based on WhatsApp Web protocol, with risk of being banned
- **Strongly recommended to use a separate phone number**
- Install plugin → QR code scan pairing → Set permissions
- Use `allowlist` mode for safest permission control
- Groups don't have `requireMention` — use `groupPolicy` to control

---

## Chapter 6: Connecting Discord

Discord is another very popular platform, especially for communities and teams.

### 6.1 Creating a Discord Bot

First, go to Discord Developer Portal to create a Bot:

#### Step 1: Go to Developer Portal

1. Open [discord.com/developers/applications](https://discord.com/developers/applications)
2. Log in to your Discord account
3. Click **New Application**

[Screenshot: Discord Developer Portal "New Application" button]

#### Step 2: Create Application

1. Enter the Application name (e.g., `My AI Assistant`)
2. Click **Create**

#### Step 3: Create Bot

1. Select **Bot** on the left sidebar
2. You'll see the Bot has already been automatically created

[Screenshot: Discord Developer Portal Bot page]

#### Step 4: Get the Token

1. On the Bot page, find the **Token** section
2. Click **Reset Token** (or **Copy** if available)
3. Copy the Token

> ⚠️ **Token is equivalent to a password — don't make it public!**

### 6.2 Setting Up Permissions

#### Enable Intents

On the Bot page, find **Privileged Gateway Intents** and enable:

- ✅ **Message Content Intent** — Read message content
- ✅ **Server Members Intent** — Identify server members

[Screenshot: Discord Bot Intents settings page]

> ⚠️ **Both Intents must be enabled, otherwise OpenClaw can't receive messages!**

#### Set Up OAuth Permissions

1. Select **OAuth2 → URL Generator** on the left sidebar
2. **Scopes** — check:
   - ✅ `bot`
   - ✅ `applications.commands`
3. **Bot Permissions** — check:
   - ✅ View Channels
   - ✅ Send Messages
   - ✅ Read Message History
   - ✅ Embed Links
   - ✅ Attach Files
   - ✅ Add Reactions

[Screenshot: Discord OAuth2 URL Generator settings]

#### Invite Bot to Server

1. Copy the generated URL at the bottom
2. Open it in a browser
3. Select the server you want to add it to
4. Click **Authorize**

[Screenshot: Discord Bot authorization invite interface]

### 6.3 Connecting OpenClaw

#### Method 1: Using Environment Variables (Recommended)

```bash
# Add to ~/.bashrc or ~/.zshrc
export DISCORD_BOT_TOKEN="your-bot-token-here"
```

Then configure OpenClaw:

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

#### Method 2: Write Directly in Config File

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

> 💡 **Recommend using environment variables** — safer than writing tokens directly in config files.

#### Restart the Gateway

```bash
openclaw gateway restart
```

#### Test

Go to Discord and send a message to the Bot in a channel. If you get an AI reply, success! 🎉

[Screenshot: Conversation between Discord channel and AI Bot]

### 6.4 Guild Channel Management

A Guild is a Discord server. You can set different rules for each server.

#### Basic Configuration

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
        "123456789012345": {  // Server ID (number)
          requireMention: true,
          users: [
            "987654321098765",  // User ID (number)
          ],
        },
      },
    },
  },
}
```

#### How to Find Server ID and User ID

1. Open Discord **Settings**
2. Go to **Advanced** → Enable **Developer Mode**
3. Right-click on server name → **Copy Server ID** (that's the Server ID)
4. Right-click on username → **Copy User ID** (that's the User ID)

[Screenshot: Discord right-click menu "Copy Server ID"]

#### Multi-Server Configuration

```json5
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        // Server A: Work group
        "111111111111111": {
          requireMention: true,
          users: ["222222222222222", "333333333333333"],
        },
        // Server B: Friends group
        "444444444444444": {
          requireMention: false,
          users: ["222222222222222"],
        },
      },
    },
  },
}
```

### 6.5 Advanced Features

#### Thread-bound Sessions

OpenClaw Discord supports **Thread-bound sessions** — when someone chats with the Bot in a thread, that thread becomes an independent session.

Benefits:
- Different topics won't get mixed up
- Clear context
- Easy to review conversation history

#### Bot Status Settings

You can customize the Bot's online status:

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

#### Channel-Specific Configuration

```json5
{
  channels: {
    discord: {
      guilds: {
        "123456789012345": {
          requireMention: true,
          users: ["987654321098765"],
          channels: {
            // Settings for a specific channel
            "111222333444555": {
              requireMention: false,  // No @ required in this channel
            },
          },
        },
      },
    },
  },
}
```

### Chapter 6 Summary

- Create Bot: Developer Portal → New Application → Bot → Copy Token
- **Must enable Message Content Intent + Server Members Intent**
- OAuth scopes: `bot` + `applications.commands`
- Store Token in environment variables for best security
- `Developer Mode` → Right-click to copy Server ID / User ID
- Thread-bound sessions help organize conversations

---

## 📋 Part 1 Summary

Congratulations on making it this far! 🎉 Here's a recap of Part 1:

### What You Learned

| Chapter | Key Points |
|---------|------------|
| **Chapter 1** | OpenClaw is a self-hosted AI agent gateway, 4 core concepts: Gateway, Agent, Channel, Session |
| **Chapter 2** | Installation order: Node.js → OpenClaw → onboard → system service |
| **Chapter 3** | Model format `provider/model`, store API Keys in env vars, set up fallback |
| **Chapter 4** | Telegram: @BotFather create Bot → configure → pairing |
| **Chapter 5** | WhatsApp: install plugin → QR code pairing → use separate number |
| **Chapter 6** | Discord: Developer Portal → Bot → enable Intents → OAuth |

### What's Next

Continue to **Part 2**, where you'll learn about:

- Advanced configuration (custom Agent behavior, tool permissions)
- Security settings (firewall, authentication)
- Troubleshooting common issues
- More practical tips

---

> 📝 **This tutorial was written by Ah Sing ⭐ for you.** If you have any questions, feel free to raise an Issue on [OpenClaw GitHub](https://github.com/nicepkg/openclaw) or discuss in the Discord community.
## Chapter 7: Other Communication Platforms

Telegram and Discord are the most commonly used platforms, but OpenClaw also supports many other communication tools. This chapter teaches you how to connect them.

### 7.1 iMessage

iMessage is Apple's native messaging tool. OpenClaw can connect to iMessage on macOS.

**Requirements:**
- macOS device (Mac computer)
- Logged in with Apple ID
- iMessage enabled

**Setup Steps:**

```bash
# Install OpenClaw on macOS
npm install -g openclaw

# Start OpenClaw
openclaw gateway start
```

Add the iMessage config to `openclaw.json`:

```json
{
  "channels": {
    "imessage": {
      "enabled": true
    }
  }
}
```

**Notes:**
- The iMessage channel **only supports macOS** — it won't work on Linux or Windows
- Requires the macOS Messages app to be working normally
- Supports group chats and one-on-one DMs

> 💡 **Tip:** If you're using a Mac as a server, iMessage is a very convenient option — no need to install any third-party tools.

**Summary:** iMessage setup is simple, but the limitation is you must use macOS. If you already have a Mac running, just add a few lines of config and you're good to go.

---

### 7.2 Signal

Signal is a privacy-focused messaging app. OpenClaw connects to Signal through `signal-cli`.

**Requirements:**
- Install `signal-cli`
- A registered phone number

**Installing signal-cli:**

```bash
# Download signal-cli
wget https://github.com/AsamK/signal-cli/releases/download/v0.13.9/signal-cli-0.13.9.tar.gz
tar xzf signal-cli-0.13.9.tar.gz
sudo mv signal-cli-0.13.9 /opt/signal-cli
sudo ln -s /opt/signal-cli/bin/signal-cli /usr/local/bin/

# Register phone number (need to receive SMS verification code)
signal-cli -a +852XXXXXXXX register

# Verify
signal-cli -a +852XXXXXXXX verify <CODE>
```

**OpenClaw config:**

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

**Test the connection:**

```bash
# Send a message to this number from another phone
# Check if it's received in the OpenClaw logs
openclaw gateway status
```

**Summary:** Signal setup takes a few extra steps, but it's great for privacy. Perfect for security-conscious users 🛡️. Remember that signal-cli requires a Java runtime.

---

### 7.3 Slack

Slack is the go-to tool for team collaboration. OpenClaw can join a workspace as a Slack Bot.

**Step 1: Create a Slack App**

1. Go to [api.slack.com/apps](https://api.slack.com/apps)
2. Click "Create New App" → "From scratch"
3. Enter the app name and select your workspace

**Step 2: Set Bot Permissions**

Add the following scopes under "OAuth & Permissions":

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

**Step 3: Install to Workspace**

1. On the "OAuth & Permissions" page, click "Install to Workspace"
2. Click "Allow" to authorize
3. Copy the "Bot User OAuth Token" (starts with `xoxb-`)

**Step 4: Set Up Event Subscriptions**

1. Enable "Event Subscriptions"
2. Enter the Request URL: `https://your-domain/slack/events`
3. Subscribe to bot events:
   - `app_mention`
   - `message.channels`
   - `message.groups`
   - `message.im`

**Step 5: OpenClaw Config**

```json
{
  "channels": {
    "slack": {
      "enabled": true,
      "config": {
        "botToken": "xoxb-your-Bot-Token",
        "signingSecret": "your-Signing-Secret",
        "appToken": "xapp-your-App-Token"
      }
    }
  }
}
```

**Summary:** Slack Bot setup has more steps, but once it's done, team collaboration becomes super convenient 💼. Make sure to set up Event Subscriptions correctly, or you won't receive messages.

---

### 7.4 Google Chat

Google Chat is great for teams using Google Workspace.

**Step 1: Create a Service Account**

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable the Google Chat API
4. Create a Service Account and download the JSON key file

**Step 2: Set Up Chat App**

On the Google Chat API settings page:
- App name: Enter whatever name you want
- Connection settings: Select "HTTP endpoint"
- URL: `https://your-domain/google-chat/events`

**Step 3: OpenClaw Config**

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

**Summary:** Google Chat is ideal for companies already using Google Workspace. The Service Account setup is a bit tedious, but the integration with the Google ecosystem is excellent 🔗.

---

### 7.5 Other Platforms Overview

OpenClaw also supports more platforms through the plugin system:

| Platform | Connection Method | Use Case |
|----------|------------------|----------|
| **Matrix** | Matrix bridge | Open-source decentralized messaging |
| **Mattermost** | Mattermost plugin | Self-hosted Slack alternative |
| **MS Teams** | Teams plugin | Microsoft ecosystem |
| **IRC** | IRC plugin | Classic chat rooms |
| **Nostr** | Nostr plugin | Web3 decentralized |

**General steps to install a plugin:**

```bash
# Install the corresponding plugin
openclaw plugins install <plugin-name>

# Configure it (each plugin is different)
# Configure in the plugins section of openclaw.json
```

**Plugin config example (using Matrix):**

```json
{
  "plugins": {
    "matrix": {
      "enabled": true,
      "config": {
        "homeserver": "https://matrix.org",
        "accessToken": "syt_your_token",
        "userId": "@yourbot:matrix.org"
      }
    }
  }
}
```

**Summary:** OpenClaw's plugin architecture lets it connect to almost any communication platform 🧩. If the platform you're using isn't in the default support list, you can write your own plugin or find one in the community.

---

## Chapter 8: Deep Dive into Agents

### 8.1 What is an Agent?

Simply put, **an Agent is a complete AI "brain"**. It has its own:

- **Workspace** — Priority memory and operating instructions
- **Sessions** — Conversations with different people
- **Auth** — API keys and permissions

By default, OpenClaw only has one agent called `"main"`. This is the one you usually chat with.

**Why would you need multiple Agents?**

- One as a personal assistant, handling your schedule and emails
- One as a customer service agent, answering customer questions
- One as a translator, specializing in multilingual tasks

Each agent's workspace is isolated from each other, so they won't mess up each other's memories.

---

### 8.2 AGENTS.md — Operating Instructions

`AGENTS.md` is the agent's "employee handbook" — it defines how the agent should behave.

**Location:** `<workspace>/AGENTS.md`

**Example AGENTS.md:**

```markdown
# AGENTS.md - Your Workspace

## Session Startup

Every time a new session starts:
1. Read `SOUL.md` — understand who you are
2. Read `USER.md` — understand who you're serving
3. Read `memory/todays-date.md` — see what happened recently
4. If it's the main session, read `MEMORY.md`

## Memory Rules

- Important decisions → Write to MEMORY.md
- Daily trivial matters → Write to memory/YYYY-MM-DD.md
- Things you forgot → Use memory_search to find

## Red Lines

- Cannot leak private data
- Must ask before deleting things
- Must confirm before external actions (sending emails, posting)
```

**Summary:** AGENTS.md is the most important config file — all behavior rules are defined here 📋. The more detailed you write it, the better the agent performs.

---

### 8.3 SOUL.md — Defining Personality

`SOUL.md` defines the agent's personality, tone, and boundaries. It's like a person's soul.

**Example SOUL.md:**

```markdown
# SOUL.md - Who You Are

## Core Principles

- Genuinely helpful, no fluff
- Have opinions, don't just follow others
- Try yourself first, ask only when stuck
- Earn trust through actions

## Tone

- Concise and direct
- Detailed when needed
- Not a corporate drone
- No sycophancy

## Boundaries

- Private things stay private
- Must ask before external actions
- Don't speak on behalf of the user in group chats
```

**Summary:** SOUL.md gives your agent a "personality" — it's not just a rigid robot 🎭. Spend some time writing it well; the difference in results is huge.

---

### 8.4 USER.md — User Profile

`USER.md` stores information about you (the user), so the agent knows how to communicate with you.

**Example USER.md:**

```markdown
# USER.md - About Your Human

- **Name:** Kei Gor
- **Timezone:** Asia/Hong_Kong (UTC+8)
- **Language:** Mainly Cantonese

## Context

- Lives in Hong Kong
- Job: Quantity Surveyor (QS)
- Communicates via Telegram
```

**Summary:** USER.md helps the agent understand who it's serving, including language preferences, timezone, and background info 👤.

---

### 8.5 IDENTITY.md — Identity Settings

`IDENTITY.md` defines the agent's own identity.

**Example IDENTITY.md:**

```markdown
# IDENTITY.md - Who Am I?

- **Name:** Ah Sing
- **Creature:** AI Assistant
- **Vibe:** Friendly, practical, with a bit of humor
- **Emoji:** ⭐
```

**Summary:** IDENTITY.md is simple but important — it gives the agent its own name and image 🪪.

---

### 8.6 BOOTSTRAP.md — Initialization Guide

`BOOTSTRAP.md` is only used on the first run. It guides the agent through initializing its own settings.

**How it works:**
1. On the first session, the agent receives BOOTSTRAP.md
2. It follows the instructions inside (e.g., ask the user questions, create files)
3. After completion, it **deletes** BOOTSTRAP.md
4. Future sessions won't see it again

**Example BOOTSTRAP.md:**

```markdown
# BOOTSTRAP.md - First Run

This is your first run! Please follow these steps:

1. Ask the user what your name should be
2. Write the answer to IDENTITY.md
3. Ask the user for their timezone
4. Update USER.md
5. Create the memory/ directory
6. Delete this file when done
```

**Summary:** BOOTSTRAP.md is use-and-throw — it's the "delivery guide" for when the agent is born 🍼.

---

### 8.7 TOOLS.md — Tool Notes

`TOOLS.md` records special tool configurations in your environment.

**Example TOOLS.md:**

```markdown
# TOOLS.md - Local Notes

## Obsidian

- Vault path: /home/user/obsidian-vault
- API URL: https://localhost:27124/
- Token: xxxxx

## Cameras

- Front door: camera-01
- Backyard: camera-02

## TTS Settings

- Preferred voice: en-US-Neural2-J
```

**Summary:** TOOLS.md is the agent's "environment cheat sheet" 📝 — it records all the local tool details in one place.

---

### 8.8 Workspace File Management

OpenClaw automatically injects workspace files into the context at the start of each session. But there are size limits:

| Setting | Default | Description |
|---------|---------|-------------|
| `bootstrapMaxChars` | 20,000 | Max characters for a single file |
| `bootstrapTotalMaxChars` | 150,000 | Total max characters for all bootstrap files |

**File injection priority order:**

1. `SOUL.md` — Always injected
2. `USER.md` — Always injected
3. `IDENTITY.md` — Always injected
4. `AGENTS.md` — Always injected
5. `TOOLS.md` — Always injected
6. `MEMORY.md` — Only injected in the main session
7. `HEARTBEAT.md` — Injected when receiving heartbeats
8. `memory/YYYY-MM-DD.md` — Today's and yesterday's daily logs

**Management tips:**

- Large files get auto-truncated, so don't stuff too much into the workspace
- Use `memory_search` to find old data instead of cramming everything into context
- Regularly clean up MEMORY.md — remove outdated content

**Summary:** Workspace files are the agent's memory foundation, but be smart about it — context is limited, so don't waste it on unimportant stuff 💾.

---

## Chapter 9: Session Management

### 9.1 What is a Session?

A **Session** is one conversation. Every time you chat with an agent, there's a session running behind the scenes.

A session contains:
- **Message history** — Conversation records
- **Context** — The agent's workspace files + conversation memory
- **Auth** — Who's chatting

---

### 9.2 DM Scope Settings

DM Scope determines how private message sessions are assigned:

```json
{
  "agents": {
    "defaults": {
      "dmScope": "main"
    }
  }
}
```

**Comparison of four modes:**

| Mode | Description | Use Case |
|------|-------------|----------|
| `main` | All DMs share one session | Single user |
| `per-peer` | Isolated by sender | Multiple people sharing one bot |
| `per-channel-peer` | Isolated by channel + sender | Recommended for multi-user |
| `per-account-channel-peer` | Isolated by account + channel + sender | Strictest isolation |

**Config example (per-channel-peer):**

```json
{
  "agents": {
    "defaults": {
      "dmScope": "per-channel-peer"
    }
  }
}
```

**Real-world impact:**

If you use `main` mode, person A and person B messaging the same bot will share the same session — meaning they can see each other's conversations 😱. If multiple people use your bot, you MUST use `per-peer` or above.

**Summary:** DM Scope is a must-check setting for multi-user scenarios. For personal use, pick `main`. For multiple users, pick `per-peer` or `per-channel-peer`.

---

### 9.3 Group Sessions

In groups, the session ID format is:

```
agent:<agentId>:<channel>:group:<id>
```

For example, a Telegram group session might be:

```
agent:main:telegram:group:-100123456789
```

**Telegram Topics:**

If the group has Topics enabled (forum mode), the session ID gets further divided:

```
agent:main:telegram:group:-100123456789:topic:42
```

This way, each topic has its own independent context — no mixing things up.

**Summary:** Group sessions are automatically isolated — each group (and even each topic) has its own independent conversation context 🗂️.

---

### 9.4 Session Lifecycle

A session's life looks like this:

```
Create → Chat → Compress → Reset → Archive → Cleanup
```

**Session Reset:**

By default, sessions auto-reset at 4:00 AM every day. You can also trigger it manually:

```
/new    # Start a new session
/reset  # Reset the current session
```

**Session maintenance settings:**

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

| Setting | Description |
|---------|-------------|
| `pruneAfter` | Clean up after 30 days of inactivity |
| `maxEntries` | Keep max 500 messages |
| `rotateBytes` | Rotate when exceeding 10MB |

**Summary:** Sessions have complete lifecycle management — old sessions are auto-cleaned, so you don't need to worry about storage blowing up 🧹.

---

### 9.5 Session Compaction

When a conversation gets too long and context approaches the limit, OpenClaw automatically triggers **Compaction**.

**What Compaction does:**
1. Summarizes old conversations
2. Preserves important context
3. Discards detailed but unimportant content

**Pre-compaction Memory Flush:**

Before compaction, the system automatically reminds the agent:
> "Session is about to be compressed. Please write important information to memory files."

This way, the agent saves important stuff to `memory/` files before compaction happens.

**Related config:**

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

**Summary:** Compaction is OpenClaw's "auto-slimming" mechanism 🗜️ — it ensures context doesn't overflow, while using memory flush to protect important information from being lost.

---

### 9.6 Multi-Agent Session Routing

If you have multiple agents, you can set up routing rules to send different types of messages to different agents:

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

**Routing methods:**
- Use the `/agent support` command to switch
- Use @mention to specify an agent in groups
- Use Hooks for automatic routing (see Chapter 13)

**Summary:** Multi-Agent routing is great for scenarios needing different areas of expertise — each agent has its own independent memory and personality 🤖.

---

## Chapter 10: Memory System

### 10.1 How Agents Remember Things

OpenClaw's agent is "brand new" in each session — it won't automatically remember things from the last session. But through the **Memory System**, it can have memory like a human.

**Two levels of memory:**

1. **Short-term memory** — `memory/YYYY-MM-DD.md`, one file per day
2. **Long-term memory** — `MEMORY.md`, curated important information

---

### 10.2 Workspace Files

**Memory-related workspace files:**

```
workspace/
├── MEMORY.md              # Long-term memory (only loaded in main session)
├── memory/
│   ├── 2026-04-01.md      # Today's log
│   ├── 2026-03-31.md      # Yesterday's log
│   └── ...                # Historical logs
```

**Daily log format example:**

```markdown
# 2026-04-01

## Timeline

- 09:00 Kei Gor asked about QS quotation questions
- 10:30 Helped organize the Notion VO Tracker
- 14:00 Found information about contract terms
- 16:00 Discussed project progress with Kei Gor

## Important Items

- VO quotation deadline is April 15
- Kei Gor is going on a business trip to Shenzhen next week
```

**Long-term memory format example:**

```markdown
# MEMORY.md - Long-Term Memory

## About Kei Gor

- Job: Quantity Surveyor (QS)
- Works on engineering projects in Hong Kong
- Prefers communicating in Cantonese

## Decision Log

- 2026-03-15: Decided to use OpenClaw for all communications
- 2026-03-20: Notion as main project management tool

## Common Knowledge

- Contract VO process: Receive change notice → Quote → Approve → Place order
```

**Summary:** Daily logs are raw data, long-term memory is the refined essence 🧠. Use both together and the agent won't "lose its memory."

---

### 10.3 Memory Search

You don't need to cram all memories into context. OpenClaw provides semantic search tools:

**memory_search — Semantic search:**

```
Use memory_search to search: "How much was the VO amount Kei Gor mentioned last time?"
→ Returns relevant memory snippets
```

**memory_get — Precise reading:**

```
Use memory_get to read: memory/2026-03-15.md
→ Returns the full content of that file
```

**Search technology:**

OpenClaw uses **semantic + BM25 hybrid search**:
- Semantic search: Understands meaning, not just word matching
- BM25: Traditional keyword search
- Combined together for much better accuracy

**Summary:** Memory search lets the agent quickly find needed information from a large history of data 🔍, without having to read everything.

---

### 10.4 Long-term vs Short-term Memory

| Feature | Short-term (Daily Logs) | Long-term (MEMORY.md) |
|---------|------------------------|----------------------|
| Format | `memory/YYYY-MM-DD.md` | `MEMORY.md` |
| Content | Raw logs | Curated highlights |
| Loading | Read on demand during session | Only injected in main session |
| Maintenance | Auto append | Manually curated updates |
| Retention | Permanent (but older ones are rarely used) | Continuously updated |

**When to use which:**

- **Real-time recording** → Write to today's `memory/YYYY-MM-DD.md`
- **Important decisions** → Also update `MEMORY.md`
- **Finding old data** → Use `memory_search`
- **Long-term knowledge** → Put in `MEMORY.md`

**Summary:** Short-term memory is like a diary 📓, long-term memory is like an encyclopedia 📚. Each has its purpose — using them together is best practice.

---

### 10.5 Automatic Memory Management

OpenClaw has several automatic mechanisms to help manage memory:

**1. Auto Memory Flush**

When a session approaches compaction, the system automatically reminds the agent to write to memory:

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

**2. Session Startup Memory Loading**

Every time a new session starts, the agent automatically:
- Reads today's and yesterday's `memory/` logs
- Reads `MEMORY.md` in the main session

**3. Periodic Memory Maintenance (via Heartbeat)**

Add this to HEARTBEAT.md:

```markdown
## Memory Maintenance

Do this every few days:
1. Review recent memory/ logs
2. Curate important information into MEMORY.md
3. Remove outdated items from MEMORY.md
```

**Summary:** Automatic memory management reduces the agent's workload 💪, allowing the memory system to run long-term without manual intervention.

---

## Chapter 11: Skills System

### 11.1 What is a Skill?

A **Skill** is a directory containing a `SKILL.md` file that teaches the agent how to do something.

Think of skills like "expansion packs" — you can add new capabilities to the agent without modifying the core program.

**Skill sources (priority order):**

1. `<workspace>/skills/` — Your own custom ones (highest priority)
2. `~/.openclaw/skills/` — Globally installed
3. `~/.agents/skills/` — Agent-specific
4. Bundled — Built into OpenClaw

---

### 11.2 ClawHub — Skill Marketplace

**ClawHub** (https://clawhub.com) is OpenClaw's public skill marketplace — like an App Store with tons of ready-made skills you can download.

**Browse skills:**

```bash
# Search for skills
openclaw skills search weather

# View skill details
openclaw skills info weather
```

---

### 11.3 Installing Skills

**Install:**

```bash
# Install a skill
openclaw skills install weather

# Update all installed skills
openclaw skills update --all

# List installed skills
openclaw skills list
```

**Configure installed skills:**

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

**Gating:**

Some skills need specific conditions to run:
- `bins`: Requires certain command-line tools
- `env`: Requires environment variables
- `config`: Requires specific configuration

If conditions aren't met, the skill will show why installation failed.

**Token impact:**

Each skill uses about 97 characters + the length of its name and description. Installing too many skills takes up context space, so only install what you need.

**Summary:** ClawHub lets you quickly expand the agent's capabilities 🧱 — it's like building with Lego blocks.

---

### 11.4 Creating Custom Skills

**Step 1: Create the directory structure**

```bash
mkdir -p ~/workspace/skills/my-skill
```

**Step 2: Write SKILL.md**

```markdown
---
name: my-skill
description: My custom skill for doing XYZ
version: 1.0.0
author: Your Name
openclaw:
  requires:
    bins:
      - curl
    env:
      - API_KEY
---

# My Skill

## Purpose

This skill is used to do XYZ.

## Usage

When the user requests to do XYZ:

1. Find the relevant data
2. Execute ABC
3. Return the result

## Example

User: "Help me do XYZ"
→ Call API with curl → Format results → Reply to user

## Error Handling

- If the API doesn't respond, wait 5 seconds and retry
- If it fails 3 times, report the error
```

**Step 3: Test the skill**

```bash
# Reload skills
openclaw gateway restart

# Test
# Trigger your skill in a conversation
```

**AgentSkills format details:**

- **YAML frontmatter**: Defines metadata (name, description, requires, etc.)
- **Markdown body**: Instructions that teach the agent how to use it
- Supports referencing files with relative paths

**Summary:** Writing a custom skill isn't hard — the core is just one `SKILL.md` file ✍️. A well-written skill file is like teaching the agent a new ability.

---

### 11.5 Recommended Skills

Here are some very useful skills that are highly recommended:

| Skill | Purpose | Install Command |
|-------|---------|-----------------|
| `weather` | Check weather | `openclaw skills install weather` |
| `healthcheck` | Host security check | `openclaw skills install healthcheck` |
| `skill-creator` | Create new skills | `openclaw skills install skill-creator` |
| `obsidian` | Manage Obsidian notes | `openclaw skills install obsidian` |
| `nano-banana-pro` | AI image generation | `openclaw skills install nano-banana-pro` |

**Summary:** ClawHub has lots of useful ready-made skills — using them well can save you tons of time ⏱️.

---
## Chapter 12: Tools Usage

### 12.1 exec — Running Shell Commands

`exec` is one of the most powerful tools — it lets you execute any shell command on the server! 💪

**Basic usage:**

```bash
# List files
ls -la

# Check system status
df -h
free -m

# Run a Python script
python3 script.py
```

**Background execution:**

```bash
# For long-running tasks, use yieldMs to auto-background
# Setting yieldMs: 5000 means it goes to background after 5 seconds
```

**Security modes:**

| Mode | Description |
|------|------|
| `sandbox` | Most strict, isolated environment |
| `gateway` | Medium, limited permissions |
| `node` | Full permissions (use with caution ⚠️) |

**Exec Approvals:**

Some dangerous commands need your approval before they run:
- `rm`, `mv` of large numbers of files
- Installing software
- Modifying system settings

```bash
# If you receive an approval request
/approve allow-once   # Allow once
/approve allow-always  # Allow permanently
/deny                  # Deny
```

**Summary:** exec lets the agent do almost anything, but safety first — use sandbox mode and enable approvals! 🔒

---

### 12.2 browser — Browser Automation

The `browser` tool can control a browser for web automation 🌐

**Two browser modes:**

| Mode | Description | Use Case |
|------|------|------|
| `openclaw` profile | Isolated agent-only browser | Automation tasks |
| `user` profile | Connects to your real Chrome | When you need login state |

**Basic operation flow:**

```
1. Open page → 2. Snapshot → 3. Find ref → 4. Action (click/type)
```

**Two snapshot modes:**

- **AI Snapshot**: Good for complex pages, AI understands page structure
- **Role Snapshot**: Standard accessibility tree, faster

**Example: Auto-filling a form**

```
1. browser open "https://example.com/form"
2. browser snapshot (get page structure)
3. browser click ref="e12" (click an element)
4. browser type ref="e15" text="Hello World"
5. browser click ref="submit-btn"
```

**Remote CDP:**

Supports Browserless, Browserbase, and other remote browser services — no need to run a browser locally! 🎉

**Summary:** browser automation lets the agent do things on the web — fill forms, search for info, take screenshots. Very versatile! 📸

---

### 12.3 web_search / web_fetch — Web Search

**web_search — Search engine:**

OpenClaw supports multiple search engines:
- Brave Search
- Perplexity
- Gemini (Google Search)
- Grok

```bash
# Basic search
web_search "OpenClaw tutorial"

# Limit results
web_search "Hong Kong weather" count=3
```

**web_fetch — Fetching web pages:**

```bash
# Fetch page content, convert to Markdown
web_fetch "https://example.com/article"

# Text only (no formatting)
web_fetch "https://example.com/article" extractMode=text

# Limit character count
web_fetch "https://example.com" maxChars=5000
```

**Real-world usage scenarios:**

```
User: "What's in the news today?"
→ web_search "Hong Kong news today"
→ Find results
→ Reply with summary
```

```
User: "Help me see what this webpage says"
→ web_fetch "https://example.com/long-article"
→ Read content
→ Summarize key points
```

**Summary:** web_search and web_fetch are the agent's two big tools for going online — one for finding things 🔍, one for reading things 📖

---

### 12.4 File Operations

OpenClaw supports basic file operations 📁

**read — Read a file:**

```bash
# Read a file
read "/path/to/file.txt"

# Read part of a file (for large files)
read "/path/to/large-file.txt" offset=100 limit=50
```

**write — Write a file:**

```bash
# Create or overwrite a file
write "/path/to/new-file.txt" content="Hello World"
```

**edit — Edit a file:**

```bash
# Precise replacement
edit "/path/to/file.txt" oldText="old content" newText="new content"
```

**Real example:**

```
User: "Change the port to 8080 in config.json"
→ read config.json
→ Find "port": 3000
→ edit config.json oldText='"port": 3000' newText='"port": 8080'
```

**Summary:** File operations are simple and straightforward — read / write / edit three commands handle most needs! ✅

---

### 12.5 TTS — Text-to-Speech

TTS (Text-to-Speech) converts text into speech 🔊

**Basic usage:**

```
tts "Hello, I'm Ah Sing! How can I help you?"
```

**Use cases:**

- Telling stories, summarizing news
- For long replies, voice is more convenient
- Friendly for visually impaired users

**Notes:**
- TTS audio is automatically sent to the current channel
- Reply with `NO_REPLY` after success to avoid duplicate messages

**Summary:** TTS lets the agent "speak out loud" — adds an interactive experience! 🎙️

---

### 12.6 Other Tools

**image_generate — Image generation:**

```
image_generate "A cartoon cat wearing a safety helmet at a construction site"
```

Supports multiple image generation models like OpenAI, Google, etc.

**pdf — PDF analysis:**

```
pdf "/path/to/document.pdf" prompt="Summarize the key points of this document"
```

Can analyze both text and image content in PDFs.

**canvas — UI rendering:**

```
canvas present url="https://example.com"
```

Renders a webpage or HTML on a canvas — supports screenshots and interaction.

**Summary:** OpenClaw's toolbox is very rich — almost any task can find the right tool! 🧰

---

## Chapter 13: Automation

### 13.1 Heartbeat — Polling

**Heartbeat** is OpenClaw's periodic "heartbeat check" — it lets the agent do things proactively without waiting to be asked! 💓

**How it works:**

```
Set interval → Trigger on schedule → Read HEARTBEAT.md → Execute checklist → Report results
```

**Configuration:**

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

The above sets a trigger every 30 minutes (1,800,000 ms).

**HEARTBEAT.md example:**

```markdown
# HEARTBEAT.md - Heartbeat Checklist

## Every check (3-4 times/day)

- [ ] Any new emails?
- [ ] Any calendar events in the next 24 hours?
- [ ] Any new Twitter mentions?

## Notes

- Late night (23:00-08:00) — don't disturb user unless urgent
- If nothing new, reply HEARTBEAT_OK
- Each check should be 2-3 items max, save tokens
```

**State tracking:**

Use `memory/heartbeat-state.json` to track last check times:

```json
{
  "lastChecks": {
    "email": 1743460800,
    "calendar": 1743457200,
    "weather": null
  }
}
```

**When to disturb the user vs stay quiet:**

| Situation | Action |
|------|------|
| Important email arrived | Send message to user |
| Calendar event < 2 hours away | Remind user |
| Nothing new | HEARTBEAT_OK (stay quiet 🤫) |
| Late night 23:00-08:00 | HEARTBEAT_OK unless urgent |

**Summary:** Heartbeat changes the agent from "passive responder" to "proactive helper" — it can patrol your emails, calendar, and more! 🛡️

---

### 13.2 Cron Jobs — Scheduled Tasks

**Cron Jobs** are precisely timed tasks — different from heartbeat because cron requires exact timing, while heartbeat can drift ⏰

**Configuration:**

In `openclaw.json`:

```json
{
  "cron": {
    "jobs": [
      {
        "name": "morning-briefing",
        "schedule": "0 9 * * *",
        "agent": "main",
        "prompt": "Create today's morning briefing: weather, calendar, email summary"
      },
      {
        "name": "weekly-report",
        "schedule": "0 17 * * 5",
        "agent": "main",
        "prompt": "Compile this week's work log, create a weekly report"
      }
    ]
  }
}
```

**Cron syntax quick reference:**

```
minute hour day month weekday

# Examples:
0 9 * * *     # Every day at 9 AM
30 14 * * 1   # Every Monday at 2:30 PM
0 0 1 * *     # 1st of every month at midnight
*/30 * * * *  # Every 30 minutes
```

**Cron vs Heartbeat:**

| Feature | Heartbeat | Cron |
|------|-----------|------|
| Timing precision | Can drift | Precise |
| Execution environment | Main session | Independent session |
| Best for | Batch patrol checks | Precisely timed tasks |
| Resource usage | Higher (periodic triggers) | Lower (on-demand triggers) |

**Real example: Daily security check**

```json
{
  "name": "security-audit",
  "schedule": "0 3 * * *",
  "agent": "main",
  "prompt": "Run host security check: check SSH config, firewall rules, system update status, write results to memory/security-audit.md"
}
```

**Summary:** Cron jobs are great for automation tasks that need precise timing. Combined with heartbeat, you can cover most automation needs! 🤖

---

### 13.3 Hooks — Event-Driven Automation

**Hooks** are event-driven automation — when certain events happen, specified actions run automatically! 🪝

**Hook types:**

| Hook | When it triggers |
|------|---------|
| `onSessionStart` | When a new session starts |
| `onSessionEnd` | When a session ends |
| `onMessage` | When a message is received |
| `onError` | When an error occurs |

**Configuration example:**

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

**Practical use cases:**

1. **Auto-read latest data when session starts**
2. **Auto-notify admin when errors occur**
3. **Auto-route specific message types to the appropriate agent**

**Summary:** Hooks provide event-driven automation, letting OpenClaw react instantly to various events! ⚡

---

### 13.4 Webhook Receiving

**Webhooks** let external services notify OpenClaw agents directly 📬

**How it works:**

```
External service (GitHub, Stripe, etc.)
    ↓ HTTP POST
OpenClaw Webhook Endpoint
    ↓
Agent processes
    ↓
Execute corresponding action
```

**Webhook configuration:**

```json
{
  "webhooks": {
    "github": {
      "path": "/webhooks/github",
      "secret": "your-webhook-secret",
      "agent": "main",
      "handler": "GitHub push {{event}} event: {{payload.commits[0].message}}"
    }
  }
}
```

**Real example: GitHub Webhook**

1. Go to GitHub repo → Settings → Webhooks
2. Payload URL: `https://your-domain/webhooks/github`
3. Content type: `application/json`
4. Secret: Same as the config above
5. Select events: Push, PR, Issues

When there's a new push, the OpenClaw agent receives a notification and can automatically:
- Update project status
- Notify team members
- Trigger CI/CD pipelines

**Webhook security:**

- Set a `secret` to verify the source
- Use HTTPS
- Restrict IP sources (if possible)

**Summary:** Webhooks let OpenClaw receive notifications from the outside world — it's the key to integrating third-party services! 🔗

---

## 🎉 Tutorial Summary

Congratulations on finishing this tutorial! 🎊 From Telegram basic setup to multi-platform connections, Agent concepts, Session management, Memory system, Skill expansion, Tool usage, and Automation — you've now mastered the core knowledge of OpenClaw!

**Quick recap:**

- **Chapter 7:** iMessage, Signal, Slack, Google Chat and other multi-platform connections
- **Chapter 8:** What Agents are, how workspace files work
- **Chapter 9:** Session lifecycle, DM Scope, Compaction
- **Chapter 10:** Memory system, semantic search, auto-management
- **Chapter 11:** ClawHub skill marketplace, custom skills
- **Chapter 12:** exec, browser, web_search and other tools
- **Chapter 13:** Heartbeat, Cron, Hooks, Webhook automation

**Next steps suggested:**

1. Install a few useful skills (weather, healthcheck)
2. Write your own SOUL.md and AGENTS.md
3. Set up heartbeat or cron for auto-patrolling
4. Browse ClawHub for skills you might need

The world of OpenClaw is vast — take your time exploring! 🚀
---

## Chapter 14: Mobile Nodes

### 14.1 What is a Node?

A **Node** is your phone, tablet, or headless device (like a Raspberry Pi) that connects back to the Gateway via **WebSocket** 📱

Simple explanation:
- **Gateway** = The brain (runs on a server)
- **Node** = The hands and feet (runs on your phone)

A Node can help you do things the Gateway can't — like taking photos, getting GPS location, screen recording, etc.

**Node types:**
| Type | Description |
|------|------|
| iOS | iPhone / iPad, using the OpenClaw iOS App |
| Android | Android phone, using the OpenClaw Android App |
| Headless | No UI devices, like Raspberry Pi, servers |

### 14.2 Pairing Process

The Node and Gateway need to **pair** before they can communicate. Pairing is device-based — each device needs to be approved individually.

#### Step 1: Initiate pairing from the Node App

Open the OpenClaw App, select "Pair new Gateway", and enter the Gateway URL.

#### Step 2: Gateway receives the pairing request

The Gateway receives a pairing request. You can approve it via Telegram or the Web console:

```
/pair
```

The Gateway will display a **pairing code**.

#### Step 3: Enter the pairing code on the Node

Enter the pairing code into the Node App.

#### Step 4: Approve the pairing

Approve on the Gateway side:

```
/nodes approve <node-id>
```

#### Managing paired devices

```bash
# List all paired Nodes
openclaw nodes list

# Revoke pairing
openclaw nodes revoke <node-id>

# Check Node status
openclaw nodes status <node-id>
```

> **💡 Tip:** Pairing info is stored in the Gateway's device pairing store — it's persistent, no need to re-pair every time.

### 14.3 Camera, Location, and Voice Features

Once paired, you can remotely control the Node's hardware through the Gateway! 🎮

#### Camera

```bash
# Take a photo
camera.capture --facing back    # Rear camera
camera.capture --facing front   # Front camera

# Start recording video
camera.record start
camera.record stop
```

#### Location

```bash
# Get current GPS location
location.get

# Start tracking
location.track start --interval 60

# Stop tracking
location.track stop
```

#### Screen Recording

```bash
# Start screen recording
screen.record start

# Stop and save
screen.record stop
```

#### Voice

```bash
# Start recording
voice.record start

# Stop
voice.record stop
```

#### Real-world usage examples

You tell Ah Sing in Telegram:

> **You:** Take a photo and show me
> **Ah Sing:** Sure! (Automatically uses the Node's camera to take a photo, then sends it to you)

> **You:** Where am I?
> **Ah Sing:** (Gets GPS location) You're near Nathan Road, Tsim Sha Tsui, Hong Kong.

### 14.4 Canvas

**Canvas** is a powerful Node feature that can display an interactive interface on the Node device 🖥️

#### Canvas commands

```bash
# Present a URL
canvas.present --url https://example.com

# Present HTML content
canvas.present --html "<h1>Hello World</h1>"

# Take a Canvas screenshot
canvas.snapshot

# Hide Canvas
canvas.hide

# Execute JavaScript in Canvas
canvas.eval --script "document.title = 'Updated'"
```

#### Practical applications

Imagine displaying a Dashboard on a Node device (like a tablet) at home:

```bash
# Display a weather dashboard
canvas.present --url https://weather-widget.example.com

# Display a calendar
canvas.present --html "<iframe src='https://calendar.google.com'></iframe>"
```

### 📝 Chapter 14 Summary

- **Node** is a phone/tablet/headless device connected to the Gateway
- Pairing process: Initiate request → Gateway approve → Done
- Node provides camera, GPS, screen recording, voice, and more
- **Canvas** can display interactive interfaces on the Node
- All operations are remotely controlled through the Gateway

---

## Chapter 15: Security & Permissions

Security is the top priority! 🔐 OpenClaw provides multi-layered security mechanisms to protect your system.

### 15.1 Gateway Security

The Gateway is your central control point — protecting it means protecting your entire system.

#### Authentication methods

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

- **Token**: For API authentication — all external requests need a token
- **Password**: For Web console login

> **⚠️ Important:** Never put your token in a public place (like GitHub)!

### 15.2 Token Management

#### Setting up a token

```json
{
  "gateway": {
    "auth": {
      "token": "gw_abc123def456ghi789"
    }
  }
}
```

#### Rotating a token

```bash
# Rotate using openclaw CLI
openclaw gateway token rotate

# Or edit config directly
# Edit gateway.auth.token in openclaw.json
openclaw gateway restart
```

#### Token best practices

- Generate with a strong password generator (at least 32 characters)
- Rotate tokens regularly
- Use different tokens for different environments
- Reference tokens via environment variables, don't hardcode

### 15.3 Sandbox

The Sandbox uses Docker containers to isolate the Agent's operations, preventing destructive commands from affecting the host system 🛡️

#### Enabling sandbox

```json
{
  "sandbox": {
    "enabled": true,
    "mode": "non-main",
    "scope": "session"
  }
}
```

#### Sandbox modes

| Mode | Description |
|------|------|
| `off` | No sandbox (dangerous! ⚠️) |
| `non-main` | Sandbox only for non-main sessions (recommended ✅) |
| `all` | Sandbox for all sessions |

#### Sandbox scopes

| Scope | Description |
|------|------|
| `session` | Each session gets its own container |
| `agent` | Same agent shares a container |
| `shared` | All operations share one container |

#### How sandbox works

```
Agent command → Sandbox check → Execute in Docker container → Return result
                    ↓
         (Isolated environment, doesn't affect host)
```

### 15.4 Tool Policy

Tool Policy controls which tools the Agent can use and which it can't.

#### Allowlist mode

```json
{
  "tools": {
    "allow": ["read", "write", "exec", "web_search"],
    "deny": []
  }
}
```

#### Denylist mode

```json
{
  "tools": {
    "allow": [],
    "deny": ["exec", "browser"]
  }
}
```

#### Common tools list

| Tool | Function | Risk Level |
|------|------|----------|
| `read` | Read files | 🟢 Low |
| `write` | Write files | 🟡 Medium |
| `exec` | Execute commands | 🔴 High |
| `browser` | Browser automation | 🟡 Medium |
| `web_search` | Web search | 🟢 Low |
| `message` | Send messages | 🟡 Medium |

### 15.5 allowFrom Whitelist

`allowFrom` controls who can interact with the Agent.

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

Controls Direct Message policy:

```json
{
  "dmPolicy": "pairing"
}
```

| Value | Description |
|----|------|
| `pairing` | Need pairing before DMs (most secure 🔒) |
| `allowlist` | Only people in the whitelist can DM |
| `open` | Anyone can DM |
| `disabled` | DMs not accepted |

### 15.6 Gateway Lock

Gateway Lock prevents unauthorized configuration changes 🔒

```json
{
  "gateway": {
    "lock": true
  }
}
```

Once enabled:
- All config changes require authentication
- RPC calls need authorization
- Protects your settings from being tampered with

### 15.7 Security Audits

OpenClaw has a built-in security audit tool:

```bash
# Run security audit
openclaw security audit
```

The audit checks:
- ✅ Token strength
- ✅ Sandbox configuration
- ✅ Tool Policy settings
- ✅ allowFrom whitelist
- ✅ dmPolicy configuration
- ✅ SSRF policy for browser
- ✅ exec approvals settings

#### Security best practices checklist

- [ ] Gateway has authentication token set
- [ ] Token is at least 32 characters
- [ ] Sandbox is enabled (at least non-main mode)
- [ ] exec tool is controlled in Tool Policy
- [ ] allowFrom whitelist is configured
- [ ] dmPolicy is NOT set to `open`
- [ ] Gateway Lock is enabled
- [ ] Regularly run `openclaw security audit`

### 📝 Chapter 15 Summary

- **Gateway Security**: Use token authentication to protect the API
- **Sandbox**: Docker isolation, prevents destructive operations
- **Tool Policy**: Allowlist/denylist controls which tools are available
- **allowFrom**: Controls who can interact with the Agent
- **Gateway Lock**: Prevents unauthorized config changes
- **Regular audits**: `openclaw security audit`

---
## Chapter 16: Web Console

### 16.1 Control UI Introduction

OpenClaw has a built-in Web Console (Control UI) — you can manage everything from your browser.

#### Opening the Console

Enter this in your browser:
```
http://127.0.0.1:18789/
```

If you've set a password, you'll be asked to log in.

#### Console Tabs

| Tab | Function |
|-----|------|
| **Chat** | Chat with the Agent |
| **Config** | Edit settings |
| **Sessions** | Manage Sessions |
| **Nodes** | Manage Node devices |

### 16.2 Chat Interface

The **Chat Tab** is the most basic feature — you can talk directly with the Agent in your browser.

Features:
- Real-time conversation (supports streaming)
- Upload files/images
- View conversation history
- Switch between different Sessions

### 16.3 Settings Management

The **Config Tab** provides two editing modes:

#### Form Mode

Great for beginners — use a GUI form to change settings, no need to know JSON.

#### Raw JSON Editor

For advanced users — directly edit `openclaw.json`:

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

After making changes, press **Save** — the Config will auto hot reload.

### 16.4 Session Management

The **Sessions Tab** lets you manage all active Sessions:

- View Session list
- View conversation history for each Session
- Delete Sessions
- View Session status (active/idle)

### 📝 Chapter 16 Summary

- **Control UI** is at `http://127.0.0.1:18789/`
- Four main Tabs: Chat, Config, Sessions, Nodes
- Config supports Form mode and Raw JSON editing
- You can manage Sessions and Nodes from the Web

---

## Chapter 17: Advanced Configuration

### 17.1 Multi-Gateway Configuration

You can run multiple Gateway instances on the same machine — for example, separating testing and production environments.

#### Method: Use Different Ports and Configs

```bash
# First instance (default)
openclaw gateway start --config ~/.openclaw/openclaw.json --port 18789

# Second instance
openclaw gateway start --config ~/.openclaw/openclaw-staging.json --port 18790
```

#### Use Environment Variables to Distinguish

```bash
OPENCLAW_CONFIG=~/.openclaw/openclaw-staging.json OPENCLAW_PORT=18790 openclaw gateway start
```

### 17.2 Remote Access (Tailscale, SSH)

If your Gateway is running on a VPS or a home machine, you'll need remote access.

#### Method 1: Tailscale (Recommended)

Tailscale is the simplest and most secure method:

```bash
# Install Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# Log in
sudo tailscale up

# Then access using Tailscale IP
# http://100.x.x.x:18789/
```

**Benefits:**
- No need to expose ports to the public internet
- Encrypted connection
- Free plan is sufficient

#### Method 2: SSH Tunnel

If you don't want to use Tailscale, you can use an SSH tunnel:

```bash
# Run this on your local computer
ssh -N -L 18789:127.0.0.1:18789 user@your-server-ip

# Then open in your browser
# http://127.0.0.1:18789/
```

`-N` = Don't execute remote commands
`-L 18789:127.0.0.1:18789` = Forward local port 18789 to remote port 18789

### 17.3 Custom System Prompts

You can customize the System Prompt for each Agent:

```json
{
  "agents": {
    "list": [
      {
        "name": "default",
        "model": "gpt-4o",
        "systemPrompt": "You are a professional Hong Kong QS assistant. Answer in Cantonese, familiar with construction engineering and quantity surveying."
      },
      {
        "name": "coder",
        "model": "claude-sonnet-4-20250514",
        "systemPrompt": "You are a full-stack development expert. Write clean code注重 best practices."
      }
    ]
  }
}
```

#### System Prompt Best Practices

- Keep it concise, don't make it too long
- Clearly specify language and style
- Include specific domain knowledge
- Regularly adjust based on usage results

### 17.4 Provider Failover

Set up backup models — if the primary model fails, it automatically switches:

```json
{
  "model": {
    "primary": "gpt-4o",
    "fallbacks": ["claude-sonnet-4-20250514", "gemini-2.5-pro"]
  }
}
```

#### How It Works

```
Request → gpt-4o
         ↓ (fail)
       claude-sonnet-4-20250514
         ↓ (fail)
       gemini-2.5-pro
         ↓ (all fail)
       Return error
```

### 17.5 Queue and Rate Limit

#### Queue Mode

Controls how multiple messages are handled when they arrive simultaneously:

```json
{
  "queue": {
    "mode": "steer"
  }
}
```

| Mode | Description |
|------|------|
| `steer` | New messages can "cut in line" and interrupt current processing (default) |
| `followup` | New messages queue up, waiting for current processing to finish |
| `collect` | Collect messages over a period of time, then process them together |

#### Block Streaming

Send long replies in chunks, avoiding sending too much content at once:

```json
{
  "blockStreaming": {
    "enabled": true,
    "maxChars": 2000
  }
}
```

### 17.6 Config Hot Reload

Control how config changes are reloaded:

```json
{
  "configReload": "hybrid"
}
```

| Mode | Description |
|------|------|
| `hybrid` | Hot reload if possible, restart if not (default) |
| `hot` | Everything hot reloads (may have risks) |
| `restart` | Everything restarts (safest but will disconnect) |
| `off` | No auto reload |

#### $include: Split Config

Large configs can be split into multiple files using `$include`:

**Main file `openclaw.json`:**
```json
{
  "$include": ["./config/gateway.json", "./config/agents.json", "./config/channels.json"]
}
```

**`config/gateway.json`:**
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

Programmatically modify configuration:

```json
// config.apply — Complete config replacement
{
  "method": "config.apply",
  "params": { "config": { ... } }
}

// config.patch — Partial config update
{
  "method": "config.patch",
  "params": { "patch": { "gateway": { "port": 18790 } } }
}
```

### 📝 Chapter 17 Summary

- **Multi-Gateway**: Run multiple instances with different ports + configs
- **Remote Access**: Tailscale (recommended) or SSH Tunnel
- **Custom System Prompts**: Customize each Agent's personality
- **Provider Failover**: Auto-switch to backup if primary model fails
- **Queue Modes**: Control how multiple messages are processed
- **Config Hot Reload**: Hybrid mode is the most balanced
- **$include**: Split large configs for easier management

---

## Chapter 18: Deploying to a Server

### 18.1 VPS Deployment

Deploy OpenClaw to a VPS (Virtual Private Server) for 24/7 operation.

#### Recommended VPS Providers

| Provider | Minimum Config | Price (Monthly) | Recommendation |
|----------|----------|-----------|--------|
| DigitalOcean | 1 vCPU / 1GB | ~$6 | ⭐⭐⭐⭐ |
| Hetzner | 2 vCPU / 4GB | ~€4 | ⭐⭐⭐⭐⭐ |
| Oracle Cloud Free | 4 vCPU / 24GB | Free | ⭐⭐⭐⭐ |
| AWS Lightsail | 1 vCPU / 1GB | ~$3.5 | ⭐⭐⭐ |
| Vultr | 1 vCPU / 1GB | ~$5 | ⭐⭐⭐ |

#### Installation Steps

```bash
# 1. SSH into the VPS
ssh root@your-vps-ip

# 2. Install Node.js (if not already installed)
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo bash -
sudo apt install -y nodejs

# 3. Install OpenClaw
sudo npm install -g openclaw

# 4. Initialize
openclaw init

# 5. Edit config
nano ~/.openclaw/openclaw.json

# 6. Start Gateway
openclaw gateway start
```

#### Using systemd for Service Management

Create a service file:

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

# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable openclaw
sudo systemctl start openclaw

# Check status
sudo systemctl status openclaw

# View logs
sudo journalctl -u openclaw -f
```

### 18.2 Docker Deployment

Deploying with Docker is the easiest and has the best environment consistency.

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

#### Start It Up

```bash
# Start
docker-compose up -d

# View logs
docker-compose logs -f

# Stop
docker-compose down

# Update
docker-compose pull
docker-compose up -d
```

#### Single Container Method

```bash
docker run -d \
  --name openclaw \
  -p 18789:18789 \
  -v openclaw-data:/root/.openclaw \
  --restart unless-stopped \
  openclaw/openclaw:latest
```

### 18.3 Cloud Deployment

#### Deploy to Cloud Platforms

**Railway:**
```bash
# Install Railway CLI
npm install -g @railway/cli

# Log in
railway login

# Deploy
railway init
railway up
```

**Fly.io:**
```bash
# Install flyctl
curl -L https://fly.io/install.sh | sh

# Log in
fly auth launch

# Deploy
fly launch
fly deploy
```

### 18.4 Raspberry Pi

OpenClaw supports ARM architecture and can run on a Raspberry Pi.

#### Installation Steps

```bash
# 1. Make sure the system is updated
sudo apt update && sudo apt upgrade -y

# 2. Install Node.js (ARM version)
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo bash -
sudo apt install -y nodejs

# 3. Verify
node -v
npm -v

# 4. Install OpenClaw
sudo npm install -g openclaw

# 5. Initialize and configure
openclaw init
nano ~/.openclaw/openclaw.json

# 6. Start
openclaw gateway start
```

#### Pi-Specific Tips

- Use **Raspberry Pi 4** or newer (2GB+ RAM)
- Use SSD instead of SD card (huge performance difference)
- Set up swap (at least 2GB): `sudo dphys-swapfile swapoff && sudo nano /etc/dphys-swapfile && sudo dphys-swapfile setup && sudo dphys-swapfile swapon`
- Use Tailscale for remote access

### 📝 Chapter 18 Summary

- **VPS**: DigitalOcean / Hetzner / Oracle Cloud Free recommended
- **systemd**: Linux service management with auto-restart
- **Docker**: Best environment consistency, manage with docker-compose
- **Raspberry Pi**: ARM supported, use SSD + enough RAM
- **Cloud Platforms**: Railway / Fly.io one-click deployment

---

## Chapter 19: Troubleshooting

### 19.1 Common Issues FAQ

#### ❌ Gateway Won't Start

**Symptom:** `openclaw gateway start` shows an error or has no response

**Solution Steps:**
```bash
# 1. Run doctor diagnosis first
openclaw doctor

# 2. Check config syntax
openclaw config validate

# 3. Check error logs
openclaw logs --lines 50

# 4. Try auto-fix
openclaw doctor --fix

# 5. If nothing works, re-initialize
openclaw init --force
```

#### ❌ WhatsApp Disconnected

**Symptom:** WhatsApp messages not received

**Solution Steps:**
```bash
# Reconnect
openclaw channels login whatsapp

# Check status
openclaw channels status --probe

# If QR code expired, regenerate
openclaw channels login whatsapp --refresh
```

#### ❌ Telegram Poll Failed

**Symptom:** Telegram not receiving messages

**Solution Steps:**
```bash
# 1. Check DNS
nslookup api.telegram.org

# 2. If using IPv6, switch to IPv4
# Add to openclaw.json:
# "telegram": { "ipv6": false }

# 3. If in China or restricted network, set up proxy
# "telegram": { "proxy": "socks5://127.0.0.1:1080" }

# 4. Restart
openclaw gateway restart
```

#### ❌ Discord Can't See Group Messages

**Symptom:** Bot receives DMs but not group messages

**Solution Steps:**
```bash
# 1. Check Discord intents
# In Discord Developer Portal, make sure these are enabled:
# - SERVER MEMBERS INTENT
# - MESSAGE CONTENT INTENT

# 2. Check groupPolicy
# In openclaw.json, confirm:
# "discord": { "groupPolicy": "allow" }

# 3. Re-invite the bot (make sure permissions are correct)
```

#### ❌ High Token Consumption

**Symptom:** API costs are very expensive

**Solution:**
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

- `isolatedSession`: Each session has its own context, won't accumulate old conversations
- `lightContext`: Reduces context window size

### 19.2 Doctor Diagnostic Tool

OpenClaw has a built-in diagnostic tool to help you quickly find problems.

#### Basic Diagnosis

```bash
openclaw doctor
```

Example output:
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

#### Auto Fix

```bash
openclaw doctor --fix
```

Will automatically try to fix problems that can be fixed.

### 19.3 Log Viewing

#### View Latest Logs

```bash
openclaw logs
```

#### Real-Time Log Tracking

```bash
openclaw logs --follow
# or
openclaw logs -f
```

#### View Specific Number of Lines

```bash
openclaw logs --lines 100
```

#### View Overall Status

```bash
openclaw status
```

Example output:
```
📊 OpenClaw Status

Gateway: running (port 18789)
Uptime: 3d 5h 23m
Sessions: 12 active
Channels: 3 connected (Telegram, Discord, WhatsApp)
Nodes: 2 paired
Model: gpt-4o (primary) | claude-sonnet-4-20250514 (fallback)
```

#### Channel Status

```bash
openclaw channels status --probe
```

### 📝 Chapter 19 Summary

- **Doctor**: `openclaw doctor` to diagnose + `--fix` for auto-repair
- **Logs**: `openclaw logs -f` for real-time tracking
- **Common Issues**: Config validation, reconnecting, DNS/IPv6 checks
- **Token Explosion**: Use `isolatedSession` + `lightContext` to reduce consumption

---

## Chapter 20: Real-World Use Cases

### 20.1 Personal AI Assistant

The most common use case — turn OpenClaw into your personal AI assistant.

#### Example Configuration

```json
{
  "agents": {
    "list": [{
      "name": "personal",
      "model": "gpt-4o",
      "systemPrompt": "You are a caring personal assistant. Communicate in Cantonese. Help manage schedules, reply to emails, and search for information. Be concise and direct, no fluff."
    }]
  },
  "channels": {
    "telegram": {
      "token": "YOUR_TELEGRAM_BOT_TOKEN"
    }
  }
}
```

#### Daily Use Scenarios

- 📅 **Schedule Management:** "What meetings do I have tomorrow?"
- 📧 **Email Summary:** "Any important emails?"
- 🔍 **Information Search:** "Find me the latest news about XXX"
- 🌤️ **Weather Forecast:** "Will it rain tomorrow?"
- 📝 **Note Organization:** "Help me organize these meeting notes"

### 20.2 Developer Workflow

Developers can use OpenClaw for code review, automated testing, and more.

#### Example Configuration

```json
{
  "agents": {
    "list": [{
      "name": "dev",
      "model": "claude-sonnet-4-20250514",
      "systemPrompt": "You are a senior full-stack developer. Write clean code,注重 best practices. Explain technical concepts in English."
    }]
  }
}
```

#### Use Scenarios

- 🔍 **Code Review:** Paste code directly for review
- 🐛 **Debug:** "This code has a bug, help me look at it"
- 📖 **Documentation:** "Write a docstring for this function"
- 🧪 **Testing:** "Write unit tests for this module"
- 🔄 **Git Operations:** "Help me create a feature branch"

### 20.3 Team Collaboration

OpenClaw can serve as a team assistant in Discord / Slack groups.

#### Example Configuration

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
      "systemPrompt": "You are the team's AI assistant. Don't respond too frequently in the group. Only speak when someone @mentions you or when you feel you can genuinely help."
    }]
  }
}
```

#### Use Scenarios

- 📋 **Meeting Notes:** "Ah Sing, help summarize today's meeting highlights"
- ❓ **FAQ:** Common questions from new members
- 📊 **Data Query:** "What's the progress on this project?"
- 🔔 **Notification Forwarding:** CI/CD failures, PRs needing review, etc.

### 20.4 Automated Monitoring

Use OpenClaw for scheduled checks and monitoring.

#### Example Configuration

```json
{
  "cron": [
    {
      "schedule": "0 */6 * * *",
      "prompt": "Check server status and disk usage, send a warning if anything exceeds 80%",
      "channel": "telegram",
      "target": "YOUR_CHAT_ID"
    },
    {
      "schedule": "0 9 * * 1",
      "prompt": "Summarize last week's Git commits and generate a weekly report",
      "channel": "telegram",
      "target": "YOUR_CHAT_ID"
    }
  ]
}
```

#### Use Scenarios

- 🖥️ **Server Monitoring:** CPU, RAM, Disk usage
- 📈 **Website Monitoring:** Is the website running normally?
- 💰 **Cost Tracking:** AWS/GCP monthly bills
- 📰 **News Summary:** Daily industry news summary
- 🔄 **Backup Check:** Confirm backups completed successfully

### 📝 Chapter 20 Summary

- **Personal Assistant**: Schedule, email, search, weather
- **Developer Workflow**: Code Review, Debug, Documentation
- **Team Collaboration**: Discord/Slack group assistant
- **Automated Monitoring**: Cron scheduled execution, server monitoring

---

## Conclusion

Congratulations! If you've read this far, you've mastered the core features of OpenClaw.

### 🚀 Advanced Learning Resources

| Resource | Link |
|----------|------|
| Official Docs | https://docs.openclaw.ai |
| Discord Community | https://discord.com/invite/clawd |
| Skills Marketplace | https://clawhub.com |
| GitHub | https://github.com/openclaw/openclaw |

### 🤝 Community Support

Join the **Discord Community** (https://discord.com/invite/clawd) to exchange tips with other users, share Skills, and get help.

### 🛠️ Contribute to OpenClaw

OpenClaw is open source — contributions are welcome:
- 🐛 Report Bugs
- 💡 Suggest New Features
- 📝 Improve Documentation
- 🧩 Develop Skills and share on ClawHub
- ⭐ Star on GitHub

### Next Steps

1. **Fine-tune your setup**: Adjust the config to your needs
2. **Explore Skills**: Find useful Skills on ClawHub
3. **Join the Community**: Lots of experts in the Discord
4. **Share Your Experience**: Share your usage tips with others

> The possibilities of OpenClaw are nearly unlimited. Use your creativity to build your own AI assistant! ⭐

---

## Appendix

### A. Common CLI Command Quick Reference

#### Basic Operations

| Command | Description |
|------|------|
| `openclaw init` | Initialize OpenClaw |
| `openclaw gateway start` | Start Gateway |
| `openclaw gateway stop` | Stop Gateway |
| `openclaw gateway restart` | Restart Gateway |
| `openclaw gateway status` | View Gateway status |
| `openclaw status` | Overall status overview |

#### Diagnostics and Logs

| Command | Description |
|------|------|
| `openclaw doctor` | Diagnostic tool |
| `openclaw doctor --fix` | Auto-fix |
| `openclaw logs` | View logs |
| `openclaw logs --follow` | Real-time log tracking |
| `openclaw logs --lines 100` | View last 100 lines |

#### Channel Management

| Command | Description |
|------|------|
| `openclaw channels list` | List channels |
| `openclaw channels login <channel>` | Log in to a channel |
| `openclaw channels status --probe` | Channel status check |

#### Node Management

| Command | Description |
|------|------|
| `openclaw nodes list` | List paired Nodes |
| `openclaw nodes approve <id>` | Approve pairing |
| `openclaw nodes revoke <id>` | Revoke pairing |

#### Security

| Command | Description |
|------|------|
| `openclaw security audit` | Security audit |
| `openclaw gateway token rotate` | Rotate Token |
| `openclaw config validate` | Validate config |

### B. openclaw.json Configuration Examples

#### Minimal Configuration

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

#### Full Production Configuration

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
      "systemPrompt": "You are a helpful AI assistant.",
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

### C. Environment Variables List

| Environment Variable | Description | Default |
|----------|------|--------|
| `OPENCLAW_CONFIG` | Config file path | `~/.openclaw/openclaw.json` |
| `OPENCLAW_PORT` | Gateway port | `18789` |
| `OPENCLAW_HOME` | OpenClaw home directory | `~/.openclaw` |
| `OPENAI_API_KEY` | OpenAI API Key | — |
| `ANTHROPIC_API_KEY` | Anthropic API Key | — |
| `GOOGLE_API_KEY` | Google AI API Key | — |
| `TELEGRAM_BOT_TOKEN` | Telegram Bot Token | — |
| `DISCORD_BOT_TOKEN` | Discord Bot Token | — |
| `NODE_ENV` | Runtime environment | `development` |
| `DEBUG` | Debug mode | — |

#### Using Environment Variables

```bash
# Set in your shell profile (~/.bashrc or ~/.zshrc)
export OPENAI_API_KEY="sk-..."
export TELEGRAM_BOT_TOKEN="123456:ABC..."

# Or set in a .env file
echo 'OPENAI_API_KEY=sk-...' >> ~/.openclaw/.env
```

> **💡 Tip:** Environment variables take priority over the same settings in the config file.

---

*Tutorial complete. Thanks for reading! Feel free to visit the Discord community (https://discord.com/invite/clawd) if you have questions.* ⭐
