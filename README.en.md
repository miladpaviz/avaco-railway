<div align="center">

<h1>🛰️ Avaco Railway Relay</h1>

<p>A lightweight free relay for the XHTTP protocol on Railway</p>

[![Node.js](https://img.shields.io/badge/Node.js-≥20-339933?style=flat-square&logo=node.js&logoColor=white)](https://nodejs.org)
[![Railway](https://img.shields.io/badge/Deploy-Railway-0B0D0E?style=flat-square&logo=railway&logoColor=white)](https://railway.app)
[![License](https://img.shields.io/badge/License-GPL--3.0-blue?style=flat-square)](LICENSE)
[![Zero Dependencies](https://img.shields.io/badge/Dependencies-Zero-brightgreen?style=flat-square)](#)
[![Free Tier](https://img.shields.io/badge/Free_Tier-$5%2Fmonth-orange?style=flat-square)](#free-tier-limits)

---

**🌐 زبان / Language**

**[🇮🇷 فارسی](README.md)** &nbsp;|&nbsp; **[🇬🇧 English](README.en.md)**

**[ [Introduction](#-introduction) • [Architecture](#-architecture) • [Deployment](#-deployment-steps) • [Variables](#-environment-variables) • [Troubleshooting](#-troubleshooting) • [Management](#-management) • [License](#-license) ]**

</div>

---

## 📖 Introduction

A lightweight reverse proxy built on pure Node.js (zero dependencies) that runs on **Railway** (free tier) and forwards XHTTP traffic from a public URL to your private backend server.

### ✨ Features

| Feature | Description |
|---------|-------------|
| 🪶 **Lightweight** | No external dependencies — pure standard Node.js |
| 🔐 **Authentication** | Supports secret key via `x-relay-key` header |
| 🚦 **Traffic Control** | Concurrent request limit with `MAX_INFLIGHT` |
| ⏱️ **Timeout Management** | Auto-abort slow connections via `UPSTREAM_TIMEOUT_MS` |
| 🔍 **Built-in Debug** | `/__debug` endpoint for live status inspection |
| 🌍 **Multi-region** | Choose from 4 geographic server locations |
| 💸 **Free** | ~500 hours of uptime with Railway's $5/month credit |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Your Client                          │
│                  (v2rayN / Hiddify / ...)                   │
└──────────────────────────┬──────────────────────────────────┘
                           │  HTTPS
                           │  HOST: xxx.up.railway.app
                           │  Path: /PUBLIC_RELAY_PATH
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   🚂 Railway (Edge)                         │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │           Avaco Railway Relay (Node.js)             │   │
│   │                                                     │   │
│   │  ✓ Path validation  (PUBLIC_RELAY_PATH)             │   │
│   │  ✓ Authentication   (x-relay-key)                   │   │
│   │  ✓ Traffic control  (MAX_INFLIGHT)                  │   │
│   │  ✓ Header forward   (no IP/Host leak)               │   │
│   │  ✓ Bidirectional stream (pipeline)                  │   │
│   └──────────────────────────┬──────────────────────────┘   │
└──────────────────────────────┼──────────────────────────────┘
                               │  HTTP
                               │  Path: /RELAY_PATH
                               ▼
┌─────────────────────────────────────────────────────────────┐
│              🖥️ Your Private Server (Xray-core)             │
│              TARGET_DOMAIN:PORT                             │
│              Protocol: VLESS + XHTTP                        │
└─────────────────────────────────────────────────────────────┘
```

### Request Flow

```
Client  ──►  Railway (PUBLIC_RELAY_PATH)  ──►  Server (RELAY_PATH)  ──►  Response
        ◄──                               ◄──                        ◄──
```

---

## 🚀 Deployment Steps

### Prerequisite: Install Railway CLI

```bash
# via npm
npm install -g @railway/cli

# or via Homebrew (Mac)
brew install railway
```

---

### Step 1 — Get an API Token

1. Go to **[railway.com/account/tokens](https://railway.com/account/tokens)**
2. Click **New Token**
3. Type: **Account Token**
4. Copy the token — it's shown only once!

> ⚠️ Store it somewhere safe. You can always generate a new token when needed.

---

### Step 2 — Clone and Enter Directory

```bash
git clone https://github.com/avacocloud/avaco-railway.git
cd avaco-railway/railway
```

---

### Step 3 — Create Project and Upload Code

```bash
# Create a new project
RAILWAY_API_TOKEN=your_token railway init --name avaco-railway

# Upload and deploy
RAILWAY_API_TOKEN=your_token railway up --detach
```

> 💡 `--detach` means you don't wait — Railway builds and deploys in the background.

---

### Step 4 — Set Environment Variables

```bash
RAILWAY_API_TOKEN=your_token railway variables \
  --set "TARGET_DOMAIN=https://YOUR-SERVER-IP-OR-DOMAIN:PORT" \
  --set "PUBLIC_RELAY_PATH=/api" \
  --set "RELAY_PATH=/api" \
  --set "RELAY_KEY=your-secret-key-min-16-chars" \
  --set "UPSTREAM_TIMEOUT_MS=0" \
  --set "MAX_INFLIGHT=512"
```

> ⚡ After setting variables, Railway **automatically** re-deploys.

---

### Step 5 — Get a Public Domain

```bash
RAILWAY_API_TOKEN=your_token railway domain
```

Output:
```
✔ Created domain: YOUR-APP.up.railway.app
```

Use this address in your client configuration.

---

### Step 6 — Test and Verify

```bash
curl https://YOUR-APP.up.railway.app/__debug
```

Successful output:
```json
{
  "TARGET_BASE": "https://your-server:port",
  "PUBLIC_RELAY_PATH": "/api",
  "RELAY_PATH": "/api",
  "RELAY_KEY_SET": true,
  "UPSTREAM_TIMEOUT_MS": 0,
  "MAX_INFLIGHT": 512,
  "inFlight": 0
}
```

---

## ⚙️ Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `TARGET_DOMAIN` | ✅ | — | Full URL of your server. Example: `https://1.2.3.4:443` |
| `PUBLIC_RELAY_PATH` | ✅ | `/api` | Path the client connects to. Example: `/xhttp` |
| `RELAY_PATH` | ✅ | `/api` | Path configured in your Xray server |
| `RELAY_KEY` | ❌ | — | Secret key (min 16 chars) — recommended |
| `UPSTREAM_TIMEOUT_MS` | ❌ | `0` | Upstream timeout in milliseconds. `0` = no timeout |
| `MAX_INFLIGHT` | ❌ | `512` | Maximum concurrent requests |

### Important Notes

- **`TARGET_DOMAIN`** must not have a trailing `/` — `https://1.2.3.4:443` ✅ not `https://1.2.3.4:443/` ❌
- **`PUBLIC_RELAY_PATH`** and **`RELAY_PATH`** cannot be `/`
- If `RELAY_KEY` is set, the client must send the `x-relay-key` header
- `PUBLIC_RELAY_PATH` and `RELAY_PATH` can be different (to hide the real upstream path)
- **`RELAY_PATH`** can be anything but **must exactly match the path configured in your Xray server**

---

## 🌍 Region Selection

| Region | Location | Code |
|--------|----------|------|
| 🇺🇸 US West | California | `us-west2` |
| 🇺🇸 US East | Virginia | `us-east4` |
| 🇳🇱 EU West | Amsterdam | `europe-west4` |
| 🇸🇬 Southeast Asia | Singapore | `asia-southeast1` |

To change the region, add to `railway.json`:

```json
{
  "deploy": {
    "region": "europe-west4"
  }
}
```

Then redeploy:

```bash
RAILWAY_API_TOKEN=your_token railway up --detach
```

---

## 🔍 Troubleshooting

### Primary Debug Tool

```bash
curl https://YOUR-APP.up.railway.app/__debug
```

This endpoint shows the full relay status — always check here first.

---

### View Live Logs

```bash
RAILWAY_API_TOKEN=your_token railway logs
```

---

### Common Errors and Fixes

<details>
<summary>❌ <b>500 — Misconfigured: TARGET_DOMAIN is not set</b></summary>

**Cause:** `TARGET_DOMAIN` variable is not set.

```bash
RAILWAY_API_TOKEN=your_token railway variables --set "TARGET_DOMAIN=https://YOUR-SERVER:PORT"
```

</details>

<details>
<summary>❌ <b>500 — Misconfigured: RELAY_PATH cannot be '/'</b></summary>

**Cause:** Path is set to `/` which is not allowed.

```bash
RAILWAY_API_TOKEN=your_token railway variables --set "RELAY_PATH=/api"
```

</details>

<details>
<summary>❌ <b>500 — Misconfigured: RELAY_KEY is too short</b></summary>

**Cause:** Secret key is less than 16 characters.

```bash
# Generate a random key:
openssl rand -hex 16

# Then set it:
RAILWAY_API_TOKEN=your_token railway variables --set "RELAY_KEY=your-generated-key"
```

</details>

<details>
<summary>❌ <b>403 — Forbidden</b></summary>

**Cause:** `x-relay-key` header is missing or incorrect in the request.

Add this header in your client config (e.g. v2rayN):
```
x-relay-key: your-secret-key
```

</details>

<details>
<summary>❌ <b>404 — Not Found</b></summary>

**Cause:** Request path does not match `PUBLIC_RELAY_PATH`.

```bash
# Check what's currently set:
RAILWAY_API_TOKEN=your_token railway variables

# Example: if PUBLIC_RELAY_PATH=/xhttp, client must connect to /xhttp
```

</details>

<details>
<summary>❌ <b>503 — Server Busy: Too Many Inflight Requests</b></summary>

**Cause:** Concurrent requests exceed `MAX_INFLIGHT`.

```bash
RAILWAY_API_TOKEN=your_token railway variables --set "MAX_INFLIGHT=1024"
```

</details>

<details>
<summary>❌ <b>502 — Bad Gateway</b></summary>

**Cause:** Relay cannot reach `TARGET_DOMAIN`.

**Checklist:**
- [ ] Is `TARGET_DOMAIN` correct? (IP and port)
- [ ] Has the server firewall opened the port?
- [ ] Is Xray running on the server?
- [ ] Test direct access: `curl http://YOUR-SERVER:PORT`

</details>

<details>
<summary>❌ <b>504 — Gateway Timeout</b></summary>

**Cause:** Server did not respond within the configured timeout.

```bash
RAILWAY_API_TOKEN=your_token railway variables --set "UPSTREAM_TIMEOUT_MS=0"
```

</details>

<details>
<summary>❌ <b>App deployed but won't start</b></summary>

```bash
# Check build and runtime logs:
RAILWAY_API_TOKEN=your_token railway logs

# Verify Node.js version is ≥20
```

</details>

<details>
<summary>❌ <b>Variables set but nothing changed</b></summary>

Railway should re-deploy after a variable change. If it didn't:

```bash
RAILWAY_API_TOKEN=your_token railway up --detach
```

</details>

---

### Health Check Commands

```bash
# Full debug:
curl -v https://YOUR-APP.up.railway.app/__debug

# Test with RELAY_KEY:
curl -H "x-relay-key: your-secret-key" https://YOUR-APP.up.railway.app/api

# Simple ping:
curl -o /dev/null -s -w "%{http_code}\n" https://YOUR-APP.up.railway.app/__debug
# Should return 200
```

---

## 🛠️ Management

```bash
# View all variables
RAILWAY_API_TOKEN=your_token railway variables

# Update a variable
RAILWAY_API_TOKEN=your_token railway variables --set "TARGET_DOMAIN=https://NEW-SERVER:PORT"

# Redeploy (after code changes)
RAILWAY_API_TOKEN=your_token railway up --detach

# View service status
RAILWAY_API_TOKEN=your_token railway status

# View live logs
RAILWAY_API_TOKEN=your_token railway logs --tail

# View current domain
RAILWAY_API_TOKEN=your_token railway domain
```

---

## 💰 Free Tier Limits

| Metric | Value |
|--------|-------|
| 💵 Monthly Credit | $5 |
| ⏱️ Estimated Uptime | ~500 hours/month |
| 📡 Requests | Unlimited |
| 😴 Sleep | None (unlike Render) |
| 🧠 RAM | 512 MB |
| ⚡ CPU | Shared |
| 🌐 Bandwidth | 100 GB/month |

> 💡 Need more uptime? Create a second project with another account.

---

## 📁 File Structure

```
railway/
├── src/
│   └── index.js          ← Main relay code (pure Node.js)
├── package.json          ← Project definition + engine: node ≥20
├── railway.json          ← Railway config (builder, restart policy, region)
└── README.md             ← Persian README
└── README.en.md          ← This file
```

---

## 💖 Support

If this project helped you and you'd like to support it, you can donate with cryptocurrency:

[![Donate with crypto](https://nowpayments.io/images/embeds/donation-button-white.svg)](https://nowpayments.io/donation?api_key=53edc3b4-8a65-451a-9ca9-67c30519c7a5)

---

## 🔗 Related Links

- 🚂 [Railway Dashboard](https://railway.app/dashboard)
- 🔑 [Railway Tokens](https://railway.com/account/tokens)
- 📦 [Railway CLI Docs](https://docs.railway.app/guides/cli)
- 🛠️ [XHTTP-Installer (auto server setup)](https://github.com/avacocloud/XHTTP-Installer)

---

## 📜 License

This project is licensed under **GNU GPL-3.0**.

Any redistribution, fork, or modification **must preserve**:

- ✅ Original copyright: `Copyright (C) 2025 avaco_cloud`
- ✅ Link to original repository: [github.com/avacocloud/avaco-railway](https://github.com/avacocloud/avaco-railway)
- ✅ Credit to original author: [@avaco_cloud](https://t.me/avaco_cloud)
- ✅ This LICENSE file unchanged

> ⚠️ Removing or replacing author attribution violates this license and constitutes copyright infringement, resulting in a **DMCA Takedown**.

For licensing questions or commercial use: [t.me/avaco_cloud](https://t.me/avaco_cloud)

---

<div align="center">

Made with ❤️ by [@avaco_cloud](https://github.com/avacocloud)

</div>
