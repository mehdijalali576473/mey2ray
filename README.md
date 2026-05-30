# Mey2Ray - VLESS Proxy for GitHub Codespaces

[![GitHub Codespaces](https://img.shields.io/badge/GitHub-Codespaces-181717?logo=github)](https://github.com/features/codespaces)
[![Xray-core](https://img.shields.io/badge/Xray--core-1.8.24-00BFFF?logo=xray)](https://github.com/XTLS/Xray-core)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

**Mey2Ray** is a lightweight, self-hosted VLESS + WebSocket proxy that runs inside **GitHub Codespaces** using [Xray-core](https://github.com/XTLS/Xray-core).  
It automatically generates a unique UUID, provides ready-to-use VLESS links with multiple static IPs (both old and new), and keeps itself alive with a simple heartbeat.

> 🚀 **Perfect for testing, learning, or temporary proxy needs – all within GitHub’s free Codespaces tier.**

---

## 📖 Table of Contents

- [Features](#features)
- [How It Works](#how-it-works)
- [Quick Start](#quick-start)
- [Configuration Files](#configuration-files)
- [Obtaining VLESS Links](#obtaining-vless-links)
- [Manual Usage (Outside Codespaces)](#manual-usage-outside-codespaces)
- [Important Notes](#important-notes)
- [Disclaimer & Compliance](#disclaimer--compliance)
- [License](#license)

---

## ✨ Features

- **Zero configuration** – Just create a Codespace and get VLESS links instantly.
- **Automatic UUID generation** – Random UUID per session (or override with `VLESS_UUID` env).
- **Dynamic SNI** – Uses your Codespace name to generate a valid TLS SNI (`<codespace-name>-443.app.github.dev`).
- **Multiple static IPs** – Predefined set of working IPs (old & new) for redundancy.
- **WebSocket + TLS** – Traffic is wrapped in TLS over port 443, resembling normal HTTPS.
- **Healthcheck & keepalive** – The container reports every 5 minutes to stay alive.
- **Port 443 exposed** – Automatically made public by GitHub Codespaces (HTTP protocol label).

---

## ⚙️ How It Works

1. **GitHub Codespaces** launches a container based on the provided `Dockerfile`.
2. `install.sh` downloads and installs the latest Xray-core binary.
3. `entrypoint.sh`:
   - Generates (or reads) a UUID.
   - Substitutes `${UUID}` into `config.json.template` → `config.json`.
   - Builds VLESS links using the UUID + a list of static IPs + the dynamic SNI.
   - Starts Xray with the generated configuration.
   - Prints a heartbeat message every 5 minutes.
4. Port `443` is forwarded and made public, so the proxy is reachable from the internet.

---

## 🚀 Quick Start

### Prerequisites
- A GitHub account (free tier works).
- No local installations required.

### Steps

1. **Open the repository**  
   Go to [https://github.com/meytiii/mey2ray](https://github.com/meytiii/mey2ray)

2. **Create a Codespace**  
   Click the **Code** button → **Codespaces** → **Create codespace on main**

3. **Wait for build** (about 1–2 minutes)  
   The container will build, install Xray, and start the proxy automatically.

4. **Get your VLESS links**  
   Once the post‑attach command runs, scroll through the terminal output.  
   You will see two groups of links: **OLD IPs** and **New IPs**.

5. **Connect**  
   Copy any VLESS link and paste it into a compatible client (e.g., V2Ray, Nekobox, Shadowrocket, Streisand, etc.).

> 💡 **Tip:** The terminal output stays in the Codespace log. You can always reopen the Codespace and view the boot logs to retrieve the links again.

---

## 📁 Configuration Files

| File | Purpose |
|------|---------|
| `Dockerfile` | Builds the Debian‑based image, installs dependencies, Xray, and copies scripts. |
| `install.sh` | Detects system architecture, fetches the latest Xray release, and installs the binary. |
| `config.json` | Template for Xray – VLESS inbound on `/` (WebSocket), no decryption, routing to `freedom`. |
| `entrypoint.sh` | Main runtime script: UUID substitution, link generation, Xray execution, keepalive loop. |
| `devcontainer.json` | Codespaces configuration: port forwarding, post‑attach commands, resource requirements. |

### Key Configuration Snippets

**Xray Inbound (`config.json`)** – VLESS + WS on port 443:
```json
{
  "port": 443,
  "protocol": "vless",
  "settings": {
    "clients": [ { "id": "${UUID}" } ],
    "decryption": "none"
  },
  "streamSettings": {
    "network": "ws",
    "wsSettings": { "path": "/" }
  }
}
