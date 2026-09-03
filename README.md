# AI Workspace Stack

This repository contains a secure, ready-to-deploy Docker Compose stack designed specifically for **Dokploy** (or other Traefik-based PaaS solutions). It sets up an integrated AI workspace environment containing three main services:

1. **Paperclip** - An AI-powered workspace.
2. **OmniRouter** - A centralized gateway/router for AI models.
3. **OpenCode** - A dynamic web environment for code execution and AI assistance.
4. **9Router** - AI router and token saver.
5. **Hermes Studio** - Multi-agent desktop app, local runtime, and web console for Hermes Agent.

## Built-In Security Features

This stack has been optimized for production security:
- **Network Isolation:** Internal storage databases (`db` and `omnirouter-redis`) are isolated inside a dedicated `backend` Docker network. Frontend services can access them, but they cannot be accessed directly from the internet or cross-communicate unnecessarily.
- **Non-Root Execution:** The `opencode` dynamic environment runs as the restricted `node` user rather than `root`, heavily mitigating the impact of any compromised code execution.
- **Environment-Driven Configuration:** All domains and secrets are managed via environment variables, ensuring no sensitive data or hardcoded URLs are committed to the repository.

## Deployment on Dokploy

1. **Connect Repository:** In your Dokploy dashboard, create a new **Compose** deployment and link it to this Git repository.
2. **Set up Environment Variables:** Copy the contents of `.env.example` and paste it into the **Environment** tab of your Dokploy deployment.
3. **Customize Domains & Secrets:**
   - Update the Domain variables (e.g., `PAPERCLIP_PUBLIC_URL`, `OMNI_PUBLIC_BASE_URL`) to match the domains you plan to use.
   - Generate secure 32-character hex strings for all your secrets (e.g., using `openssl rand -hex 32` locally) and update the values.
   - Add your AI model API Keys (`OPENAI_API_KEY`, `OPENROUTER_API_KEY`, etc.).
4. **Deploy:** Hit the deploy button in Dokploy. *Note: It may take a minute or two on the first run as Dokploy builds the OpenCode image and downloads the required base images.*
5. **Route Domains:** In Dokploy, route your custom domains to the respective services and ports (e.g., route your Paperclip domain to port `3100`, OmniRouter to `20130`, etc.).

## Services Overview

### 1. Paperclip
- **Internal Port:** `3100`
- Configured via `PAPERCLIP_PUBLIC_URL` and `PAPERCLIP_ALLOWED_HOSTNAMES`.
- Includes an automated script on startup to fix any missing markdown built-ins and directory permissions.

### 2. OmniRouter
- **Main API & Dashboard Port:** `20130`
- **WebSocket Port:** `20131`
- Configured via `OMNI_PUBLIC_BASE_URL` and `OMNI_WS_PUBLIC_URL`.
- Connects to an internal Redis instance (`omnirouter-redis`) for caching.

### 3. OpenCode
- **Internal Port:** `8080`
- Uses an inline Dockerfile based on Node 22 to install `opencode-ai` and set up the web environment securely as the `node` user.

### 4. 9Router
- **Internal Port:** `20129`
- Proxy / router to optimize AI tokens and manage model endpoints.

### 5. Hermes Studio
- **Internal Port:** `6060` (Web UI dashboard), `8651` (Frontend preview), `56121` (xAI OAuth)
- Multi-agent web console and local runtime for Hermes Agent, Ekko, Claude Code, Codex, and Pi.

## Local Testing

If you wish to test this stack locally before deploying to Dokploy:
1. Clone the repository and run `cp .env.example .env`.
2. Fill out the `.env` file.
3. Run `docker compose up -d`.
4. Access the services via `localhost` and the respective exposed ports (`3100`, `20130`, `8080`, `20129`, `6060`).

## Volumes

The following persistent volumes are created automatically and managed by Dokploy:
- `paperclip_data`
- `pgdata` (PostgreSQL DB for Paperclip)
- `omnirouter_data`
- `omnirouter_redis_data`
- `opencode_workspace`
- `9router_data`
- `hermes_data` (Hermes configuration & models)
- `hermes_webui_data` (Hermes Web UI persistent sessions and data)
