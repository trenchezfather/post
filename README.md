# 🚀 AI Agent — Ollama + Telegram Bot — Quick Start

## Files You Got

| File | What It Is |
|------|-----------|
| `n8n_ai_agent_workflow.json` | **Import this into n8n** — the full AI agent workflow |
| `ollama_config.json` | Configuration reference — models, prompts, tools |
| `setup_ai_agent_vps.sh` | Ubuntu VPS auto-installer script |

---

## Step-by-Step on Your Ubuntu VPS

### 1. Run the Setup Script

```bash
# SSH into your VPS
chmod +x setup_ai_agent_vps.sh
sudo ./setup_ai_agent_vps.sh
```

This installs: **Ollama + n8n + PostgreSQL + Playwright + ImageMagick + Nginx**

It pulls `llama3.1`, `qwen2.5-coder:7b`, and `llava` models automatically.

### 2. Create a Telegram Bot (if you haven't)

1. Message [@BotFather](https://t.me/BotFather) on Telegram
2. Send `/newbot` → name it → get your **bot token**
3. Copy the token

### 3. Open n8n & Set Credentials

Go to `http://YOUR_VPS_IP:5678`

Set up these credentials in n8n:

| Credential | Type | Details |
|-----------|------|---------|
| **Telegram API** | Telegram | Bot Token from BotFather |
| **PostgreSQL** | Postgres | Host: `localhost`, DB: `ai_agent_memory`, User: `ai_agent`, Pass: from setup |
| **Ollama** | (none needed) | Uses env var `OLLAMA_URL=http://localhost:11434` |

### 4. Import the Workflow

1. In n8n: **Workflows → Import from File**
2. Select `n8n_ai_agent_workflow.json`
3. Fix any credential mapping popups (select your credentials)
4. Click **Save**

### 5. Set Up Telegram Webhook

```bash
# Replace with your values
curl "https://api.telegram.org/bot<YOUR_BOT_TOKEN>/setWebhook?url=<YOUR_N8N_WEBHOOK_URL>"
```

Your n8n webhook URL is: `http://YOUR_VPS_IP:5678/webhook/telegram`

(Or use the production webhook URL from n8n's Telegram Trigger node.)

### 6. Activate & Test

- In n8n, toggle the workflow to **Active**
- Send a message to your Telegram bot
- Watch it flow through: **Planner → Reasoning → Tools → Reviewer → Reply**

---

## Workflow Architecture

```
Telegram Message
    │
    ▼
Identity & Auth (user ID, roles, limits)
    │
    ▼
Conversation Memory (PostgreSQL)
    │
    ▼
Planner Agent (Ollama) ── breaks goal into tasks
    │
    ▼
Tool Router ──┬── Shell Executor (Linux commands)
              ├── HTTP/API (web requests)
              ├── File Ops (read/write/list)
              ├── Web Search (Brave/DuckDuckGo)
              ├── Image Processor (Ollama Vision)
              └── (no tool needed)
    │
    ▼
Reasoning Agent (Ollama) ── evaluates results
    │
    ▼
Reviewer Agent (Ollama) ── polishes answer
    │
    ▼
Save to Memory (PostgreSQL)
    │
    ▼
Telegram Reply
```

---

## Customizing Models

Edit the HTTP Request nodes or set n8n environment variables:

```bash
# In /etc/systemd/system/n8n.service:
Environment="OLLAMA_MODEL=llama3.1"        # Main model
Environment="OLLAMA_VISION_MODEL=llava"    # Vision model for images
```

Or pull new models:
```bash
ollama pull deepseek-r1:8b
ollama pull mistral
ollama pull codellama
```

---

## Adding More Tools

Add new tool nodes to the workflow:

1. Add your tool node (e.g., Email, Calendar, Docker)
2. Add a new route in the **Tool Router** switch node
3. Connect tool output → **Reasoning Agent**
4. Update the Planner Agent's system prompt to know about the new tool

---

## Troubleshooting

**n8n won't start:**
```bash
journalctl -u n8n -n 50 --no-pager
```

**Ollama not responding:**
```bash
systemctl status ollama
curl http://localhost:11434/api/tags   # should list models
```

**Telegram webhook fails:**
```bash
# Delete and re-set webhook
curl "https://api.telegram.org/bot<TOKEN>/deleteWebhook"
curl "https://api.telegram.org/bot<TOKEN>/setWebhook?url=<URL>"
```

**PostgreSQL connection refused:**
```bash
sudo -u postgres psql -c "ALTER USER ai_agent WITH PASSWORD 'new_password';"
# Update password in n8n credential
```

---

## Security Notes

- 🔒 Keep Ollama API on `localhost:11434` only (don't expose to internet)
- 🔒 Use n8n's built-in encryption for credentials
- 🔒 Set up HTTPS with Certbot for production
- 🔒 Add allowed_users in ollama_config.json to restrict bot access

---

Bamzy, you're all set! Hit me up if anything breaks. 🔥
