 # Evolution API - Complete Setup Guide

## 📚 Table of Contents
- [Prerequisites](#prerequisites)
- [Understanding Docker Basics](#understanding-docker-basics-for-beginners)
- [Step 1: Clone the Repository](#step-1-clone-the-repository)
- [Step 2: Configure Environment Variables](#step-2-configure-environment-variables)
- [Step 3: Update Docker Compose Configuration](#step-3-update-docker-compose-configuration)
- [Step 4: Start the Application](#step-4-start-the-application)
- [Step 5: Verify Installation](#step-5-verify-installation)
- [Step 6: Enable and Configure N8N Integration](#step-6-enable-and-configure-n8n-integration)
- [Common Commands](#common-commands)
- [Troubleshooting](#troubleshooting)
- [Performance Optimization](#performance-optimization)
- [Security Best Practices](#security-best-practices)
- [Understanding Evolution API + N8N Architecture](#understanding-evolution-api--n8n-architecture)
- [Docker Networking Reference](#docker-networking-reference)
- [Additional Resources](#additional-resources)
- [Support](#support)

---

## Prerequisites

Before starting, make sure you have the following installed on your computer:

1. **Git** - Download from [git-scm.com](https://git-scm.com/downloads)
2. **Docker Desktop** - Download from [docker.com](https://www.docker.com/products/docker-desktop/)
   - For Windows: Requires Windows 10/11 Pro, Enterprise, or Education
   - For Mac: Requires macOS 10.15 or newer
   - For Linux: Install Docker Engine and Docker Compose

### Verify Prerequisites

Open your terminal/command prompt and run:

```bash
# Check Git installation
git --version

# Check Docker installation
docker --version

# Check Docker Compose installation
docker-compose --version
```

If all commands return version numbers, you're ready to proceed!

---

## Understanding Docker Basics (For Beginners)

### 🏠 Think of Docker Like an Apartment Building

Docker containers are like separate apartments in a building:

```
🏢 Your Computer (The Building)
├── 🚪 Apartment #1: evolution_api (The main app)
├── 🚪 Apartment #2: evolution-postgres (The database)
└── 🚪 Apartment #3: evolution-redis (The cache)
```

### Key Concepts You Must Understand

#### 1. **Why Use Service Names Instead of `localhost`?**

**❌ WRONG:**
```ini
DATABASE_CONNECTION_URI=postgresql://user:pass@localhost:5432/db
```

**✅ CORRECT:**
```ini
DATABASE_CONNECTION_URI=postgresql://user:pass@evolution-postgres:5432/db
```

**Why?**
- Inside Docker, `localhost` means "this container itself"
- To talk to other containers, use their **service names** (like apartment numbers)
- Docker has a built-in DNS that resolves service names to IP addresses

#### 2. **Why Remove External Networks?**

The original `docker-compose.yaml` includes `dokploy-network` with `external: true`. This means Docker expects a network that already exists (created by Dokploy deployment platform).

**For local development, you DON'T have Dokploy**, so this network doesn't exist and causes errors!

#### 3. **When Do You Need Dokploy?**

| Scenario | Use This | Why? |
|----------|----------|------|
| 🎓 **Learning & Practice** | Local Docker | Free, private, can experiment |
| 🧪 **Development & Testing** | Local Docker | Fast, no internet needed |
| 🌐 **Production/Public Access** | Dokploy or similar | 24/7 availability, accessible from anywhere |

**For learning: Local Docker is perfect!** ✅

---

## Step 1: Clone the Repository

### Option A: Using HTTPS (Recommended for beginners)

```bash
# Navigate to your desired folder
cd ~/Documents

# Clone the repository
git clone https://github.com/EvolutionAPI/evolution-api.git

# Enter the project directory
cd evolution-api
```

### Option B: Using SSH (For advanced users with SSH keys)

```bash
git clone git@github.com:EvolutionAPI/evolution-api.git
cd evolution-api
```

---

## Step 2: Configure Environment Variables

### 2.1 Create the `.env` File

**Windows (PowerShell):**
```powershell
Copy-Item env.example .env
```

**macOS/Linux:**
```bash
cp env.example .env
```

### 2.2 Edit the `.env` File

Open the `.env` file with your text editor and make these **critical changes**:

#### ✅ Required Changes:

```ini
# ===========================================
# SERVER CONFIGURATION
# ===========================================
SERVER_URL=http://localhost:8080

# ===========================================
# AUTHENTICATION (⚠️ CHANGE THIS!)
# ===========================================
AUTHENTICATION_API_KEY=YOUR_SUPER_SECRET_API_KEY_HERE
# Change to a strong random string!
# Example: MyStr0ng!SecretK3y2024

# ===========================================
# DATABASE CONFIGURATION
# ===========================================
DATABASE_PROVIDER=postgresql
DATABASE_CONNECTION_URI=postgresql://postgres:evolution_password@evolution-postgres:5432/evolution_db

# These variables are REQUIRED for PostgreSQL container initialization
POSTGRES_DATABASE=evolution_db
POSTGRES_USERNAME=postgres
POSTGRES_PASSWORD=evolution_password

# ===========================================
# REDIS CONFIGURATION
# ===========================================
CACHE_REDIS_ENABLED=true
CACHE_REDIS_URI=redis://evolution-redis:6379
CACHE_REDIS_PREFIX_KEY=evolution
CACHE_REDIS_TTL=604800
CACHE_REDIS_SAVE_INSTANCES=false

# Local cache fallback
CACHE_LOCAL_ENABLED=false

# ===========================================
# ENABLE N8N INTEGRATION (⚠️ IMPORTANT!)
# ===========================================
N8N_ENABLED=true
```

#### 📝 Important Notes:

1. **Database Hostname**: Use `evolution-postgres` (the Docker service name), NOT `localhost`
2. **Redis Hostname**: Use `evolution-redis` (the Docker service name), NOT `localhost`  
3. **API Key**: MUST be changed from default for security
4. **N8N_ENABLED**: Must be `true` to enable n8n chatbot integration
5. **Credentials Consistency**: The database credentials in `DATABASE_CONNECTION_URI` must match the `POSTGRES_*` variables

#### ⚙️ Optional but Recommended Changes:

```ini
# Language (default is Portuguese)
LANGUAGE=en

# Reduce logging for better performance
LOG_LEVEL=ERROR,WARN,INFO
# Remove: DEBUG,VERBOSE,DARK,WEBHOOKS,WEBSOCKET for faster performance

# Disable telemetry if you prefer
TELEMETRY_ENABLED=false

# Webhook events (enable only what you need)
WEBHOOK_EVENTS_MESSAGES_UPSERT=true
WEBHOOK_EVENTS_CONNECTION_UPDATE=true
# Set others to false to improve performance
```

### 2.3 Complete `.env` File Example

Below is a complete, working `.env` configuration optimized for local development with n8n integration:

<details>
<summary><strong>Click to expand complete .env file</strong></summary>

```ini
# ===========================================
# EVOLUTION API - COMPLETE CONFIGURATION
# ===========================================

# ===========================================
# SERVER CONFIGURATION
# ===========================================
SERVER_NAME=evolution
SERVER_TYPE=http
SERVER_PORT=8080
SERVER_URL=http://localhost:8080
SERVER_DISABLE_DOCS=false
SERVER_DISABLE_MANAGER=false

# ===========================================
# CORS
# ===========================================
CORS_ORIGIN=*
CORS_METHODS=POST,GET,PUT,DELETE
CORS_CREDENTIALS=true

# ===========================================
# AUTHENTICATION (⚠️ CHANGE THIS!)
# ===========================================
AUTHENTICATION_API_KEY=CHANGE_THIS_TO_A_SECURE_RANDOM_KEY
AUTHENTICATION_EXPOSE_IN_FETCH_INSTANCES=false

# ===========================================
# DATABASE - POSTGRESQL (FOR DOCKER COMPOSE)
# ===========================================
DATABASE_PROVIDER=postgresql
DATABASE_CONNECTION_URI=postgresql://postgres:evolution_password@evolution-postgres:5432/evolution_db
DATABASE_CONNECTION_CLIENT_NAME=evolution

# PostgreSQL Environment Variables (REQUIRED by docker-compose.yaml)
POSTGRES_DATABASE=evolution_db
POSTGRES_USERNAME=postgres
POSTGRES_PASSWORD=evolution_password

# Database Save Settings
DATABASE_SAVE_DATA_INSTANCE=true
DATABASE_SAVE_DATA_NEW_MESSAGE=true
DATABASE_SAVE_MESSAGE_UPDATE=true
DATABASE_SAVE_DATA_CONTACTS=true
DATABASE_SAVE_DATA_CHATS=true
DATABASE_SAVE_DATA_LABELS=true
DATABASE_SAVE_DATA_HISTORIC=true
DATABASE_SAVE_IS_ON_WHATSAPP=true
DATABASE_SAVE_IS_ON_WHATSAPP_DAYS=7
DATABASE_DELETE_MESSAGE=false

# ===========================================
# REDIS CACHE (FOR DOCKER COMPOSE)
# ===========================================
CACHE_REDIS_ENABLED=true
CACHE_REDIS_URI=redis://evolution-redis:6379
CACHE_REDIS_PREFIX_KEY=evolution
CACHE_REDIS_TTL=604800
CACHE_REDIS_SAVE_INSTANCES=false

# Local cache fallback
CACHE_LOCAL_ENABLED=false

# ===========================================
# LOGS (Optimized for Performance)
# ===========================================
LOG_LEVEL=ERROR,WARN,INFO
LOG_COLOR=true
LOG_BAILEYS=error

# ===========================================
# EVENT EMITTER
# ===========================================
EVENT_EMITTER_MAX_LISTENERS=50

# ===========================================
# INSTANCES
# ===========================================
DEL_INSTANCE=false
DEL_TEMP_INSTANCES=true

# ===========================================
# LANGUAGE
# ===========================================
LANGUAGE=en

# ===========================================
# TELEMETRY
# ===========================================
TELEMETRY_ENABLED=true

# ===========================================
# WEBHOOK
# ===========================================
WEBHOOK_GLOBAL_URL=
WEBHOOK_GLOBAL_ENABLED=false
WEBHOOK_GLOBAL_WEBHOOK_BY_EVENTS=false

# Webhook Events (Set to true only for events you need)
WEBHOOK_EVENTS_APPLICATION_STARTUP=false
WEBHOOK_EVENTS_INSTANCE_CREATE=false
WEBHOOK_EVENTS_INSTANCE_DELETE=false
WEBHOOK_EVENTS_QRCODE_UPDATED=true
WEBHOOK_EVENTS_MESSAGES_SET=false
WEBHOOK_EVENTS_MESSAGES_UPSERT=true
WEBHOOK_EVENTS_MESSAGES_EDITED=false
WEBHOOK_EVENTS_MESSAGES_UPDATE=true
WEBHOOK_EVENTS_MESSAGES_DELETE=false
WEBHOOK_EVENTS_SEND_MESSAGE=true
WEBHOOK_EVENTS_SEND_MESSAGE_UPDATE=false
WEBHOOK_EVENTS_CONTACTS_SET=false
WEBHOOK_EVENTS_CONTACTS_UPDATE=false
WEBHOOK_EVENTS_CONTACTS_UPSERT=false
WEBHOOK_EVENTS_PRESENCE_UPDATE=false
WEBHOOK_EVENTS_CHATS_SET=false
WEBHOOK_EVENTS_CHATS_UPDATE=false
WEBHOOK_EVENTS_CHATS_UPSERT=false
WEBHOOK_EVENTS_CHATS_DELETE=false
WEBHOOK_EVENTS_CONNECTION_UPDATE=true
WEBHOOK_EVENTS_LABELS_EDIT=false
WEBHOOK_EVENTS_LABELS_ASSOCIATION=false
WEBHOOK_EVENTS_GROUPS_UPSERT=false
WEBHOOK_EVENTS_GROUPS_UPDATE=false
WEBHOOK_EVENTS_GROUP_PARTICIPANTS_UPDATE=false
WEBHOOK_EVENTS_CALL=false
WEBHOOK_EVENTS_TYPEBOT_START=false
WEBHOOK_EVENTS_TYPEBOT_CHANGE_STATUS=false
WEBHOOK_EVENTS_ERRORS=false
WEBHOOK_EVENTS_ERRORS_WEBHOOK=

# Webhook Request Settings
WEBHOOK_REQUEST_TIMEOUT_MS=30000
WEBHOOK_RETRY_MAX_ATTEMPTS=10
WEBHOOK_RETRY_INITIAL_DELAY_SECONDS=5
WEBHOOK_RETRY_USE_EXPONENTIAL_BACKOFF=true
WEBHOOK_RETRY_MAX_DELAY_SECONDS=300
WEBHOOK_RETRY_JITTER_FACTOR=0.2
WEBHOOK_RETRY_NON_RETRYABLE_STATUS_CODES=400,401,403,404,422

# ===========================================
# WEBSOCKET
# ===========================================
WEBSOCKET_ENABLED=false
WEBSOCKET_GLOBAL_EVENTS=false

# ===========================================
# RABBITMQ (Disabled for basic setup)
# ===========================================
RABBITMQ_ENABLED=false

# ===========================================
# SQS (Disabled for basic setup)
# ===========================================
SQS_ENABLED=false

# ===========================================
# KAFKA (Disabled for basic setup)
# ===========================================
KAFKA_ENABLED=false

# ===========================================
# PUSHER (Disabled for basic setup)
# ===========================================
PUSHER_ENABLED=false

# ===========================================
# WHATSAPP BUSINESS API
# ===========================================
WA_BUSINESS_TOKEN_WEBHOOK=evolution
WA_BUSINESS_URL=https://graph.facebook.com
WA_BUSINESS_VERSION=v20.0
WA_BUSINESS_LANGUAGE=en_US

# ===========================================
# SESSION CONFIGURATION
# ===========================================
CONFIG_SESSION_PHONE_CLIENT=Evolution API
CONFIG_SESSION_PHONE_NAME=Chrome

# ===========================================
# QR CODE
# ===========================================
QRCODE_LIMIT=30
QRCODE_COLOR=#198754

# ===========================================
# INTEGRATIONS - ENABLE/DISABLE
# ===========================================
# ⚠️ IMPORTANT: Set to true to enable n8n integration!
N8N_ENABLED=true

# Other integrations (disabled by default)
TYPEBOT_ENABLED=false
CHATWOOT_ENABLED=false
OPENAI_ENABLED=false
DIFY_ENABLED=false
EVOAI_ENABLED=false
FLOWISE_ENABLED=false

# ===========================================
# S3 / MINIO STORAGE (Disabled for basic setup)
# ===========================================
S3_ENABLED=false

# ===========================================
# METRICS (Disabled for basic setup)
# ===========================================
PROMETHEUS_METRICS=false

# ===========================================
# SSL CONFIGURATION (For production only)
# ===========================================
SSL_CONF_PRIVKEY=/path/to/cert.key
SSL_CONF_FULLCHAIN=/path/to/cert.crt

# ===========================================
# PROXY CONFIGURATION (Optional)
# ===========================================
PROXY_HOST=
PROXY_PORT=
PROXY_PROTOCOL=
PROXY_USERNAME=
PROXY_PASSWORD=

# ===========================================
# AUDIO CONVERTER API (Optional)
# ===========================================
API_AUDIO_CONVERTER=
API_AUDIO_CONVERTER_KEY=
```

</details>

**💡 Key Points About This Configuration:**

1. ✅ **N8N_ENABLED=true** - Required for n8n integration
2. ✅ **Service names** used for database and Redis (not localhost)
3. ✅ **Reduced logging** for better performance
4. ✅ **Minimal webhooks** enabled for efficiency
5. ✅ **PostgreSQL credentials** match between URI and separate variables
6. ⚠️ **API key** MUST be changed before deployment

**To use this:** Copy the entire content and replace your `.env` file, then update the API key!

---

## Step 3: Update Docker Compose Configuration

### 3.1 Open `docker-compose.yaml`

The default file includes configurations for Dokploy deployment that must be removed for local development.

### 3.2 Required Changes

#### ❌ Remove These Lines:

Find and **DELETE** all references to `dokploy-network`:

```yaml
# DELETE THESE LINES:
    - dokploy-network  # Remove from api service
    - dokploy-network  # Remove from redis service  
    - dokploy-network  # Remove from postgres service

# DELETE THIS ENTIRE SECTION at the bottom:
  dokploy-network:
    external: true
```

#### ✅ Change Port Binding (Optional):

Find:
```yaml
ports:
  - "127.0.0.1:8080:8080"
```

Change to (if you want access from other devices on your network):
```yaml
ports:
  - "8080:8080"
```

### 3.3 Final Correct Structure

Your `docker-compose.yaml` should look like this:

```yaml
version: "3.8"

services:
  api:
    container_name: evolution_api
    image: evoapicloud/evolution-api:latest
    restart: always
    depends_on:
      - redis
      - evolution-postgres
    ports:
      - "8080:8080"
    volumes:
      - evolution_instances:/evolution/instances
    networks:
      - evolution-net
    env_file:
      - .env
    expose:
      - "8080"

  redis:
    container_name: evolution_redis
    image: redis:latest
    restart: always
    command: >
      redis-server --port 6379 --appendonly yes
    volumes:
      - evolution_redis:/data
    networks:
      evolution-net:
        aliases:
          - evolution-redis
    expose:
      - "6379"

  evolution-postgres:
    container_name: evolution_postgres
    image: postgres:15
    restart: always
    env_file:
      - .env
    command:
      - postgres
      - -c
      - max_connections=1000
      - -c
      - listen_addresses=*
    environment:
      - POSTGRES_DB=${POSTGRES_DATABASE}
      - POSTGRES_USER=${POSTGRES_USERNAME}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - evolution-net
    expose:
      - "5432"

volumes:
  evolution_instances:
  evolution_redis:
  postgres_data:

networks:
  evolution-net:
    name: evolution-net
    driver: bridge
```

---

## Step 4: Start the Application

### 4.1 Start Docker Desktop

Make sure **Docker Desktop** is running on your computer.

### 4.2 Start Evolution API

Open your terminal in the `evolution-api` directory and run:

```bash
# Start all services in detached mode (background)
docker-compose up -d
```

### 4.3 Wait for Initialization

The first time you run this, Docker will:
1. Download required images (~500MB-1GB) - takes 2-5 minutes
2. Create volumes for data storage
3. Start containers
4. Run database migrations
5. Initialize the API

**Total first-run time: 3-5 minutes**

### 4.4 Monitor the Startup

Watch the logs to see when the API is ready:

```bash
# View logs from all services
docker-compose logs -f

# Or view only API logs
docker-compose logs -f api
```

**Look for this message** indicating the server is ready:

```
[SERVER] HTTP - ON: 8080
```

Press `Ctrl+C` to stop viewing logs (containers keep running).

---

## Step 5: Verify Installation

### 5.1 Check Container Status

```bash
docker-compose ps
```

You should see 3 containers with status "Up":
- `evolution_api`
- `evolution_postgres`
- `evolution_redis`

### 5.2 Test API Connection

**Via Browser:**
```
http://localhost:8080/manager
```

**Via curl:**
```bash
curl -X GET http://localhost:8080 -H "apikey: YOUR_API_KEY_FROM_ENV_FILE"
```

### 5.3 Access Points

| Service | URL | Purpose |
|---------|-----|---------|
| **Manager UI** | http://localhost:8080/manager | Web interface for managing instances |
| **API Docs** | http://localhost:8080/docs | Swagger/OpenAPI documentation |
| **API Endpoint** | http://localhost:8080 | REST API endpoint |

---

## Step 6: Enable and Configure N8N Integration

### 6.1 Understanding N8N Integration

Evolution API supports **n8n** as a chatbot integration. This allows you to:
- Build custom chatbot workflows in n8n
- Process incoming WhatsApp messages
- Send automated responses
- Integrate with other services (AI, databases, APIs, etc.)

### 6.2 Find Your N8N Port (If Already Running)

**Before configuring**, check which port n8n is using:

**Windows PowerShell:**
```powershell
docker ps | Select-String "n8n"
```

**macOS/Linux:**
```bash
docker ps | grep n8n
```

**Look for the port mapping**, for example:
```
0.0.0.0:5677->5678/tcp    # Your n8n port is 5677
0.0.0.0:5678->5678/tcp    # Your n8n port is 5678
```

**⚠️ IMPORTANT:** 
- **In this guide's example**, we use port **5677**
- **YOUR port might be different** (5678, 5679, etc.)
- **Always use YOUR actual port** in all configurations below
- Replace any mention of `5677` or `XXXX` with your actual port number

**Remember your port number!** You'll need it for all webhook configurations.

### 6.3 Install N8N (If Not Already Running)

You need n8n running locally or accessible via URL. 

#### Option A: Run N8N Locally (Recommended for Learning)

```bash
# Start n8n in Docker
docker run -d --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n

# Access n8n at: http://localhost:5678
```

**Note:** If your n8n is already running on a different port (e.g., `5677`), use that port in all configurations below.

#### Option B: Use Existing N8N Installation

If you already have n8n running, note its URL.

### 6.3 Create N8N Workflow

1. **Open n8n** in your browser using your actual port (e.g., http://localhost:5677 or http://localhost:5678)
2. **Create a new workflow**
3. **Add a "Webhook" trigger node**
4. **Configure the webhook:**
   - **HTTP Method**: POST
   - **Path**: `evoapi` (or any name you want)
   - **Respond**: `Immediately` ← **IMPORTANT!**
5. **Add your processing nodes** (AI, logic, etc.)
6. **Add "Respond to Webhook" node** with your response
7. **Activate the workflow** (toggle ON at top right)
8. **Copy the production webhook URL** from the webhook node
   
   **⚠️ Important:** You'll see a URL like `http://localhost:XXXX/webhook/evoapi` where `XXXX` is your n8n port. Remember this path (`/webhook/evoapi`) - you'll need it for Evolution API configuration!

### 6.4 Configure N8N in Evolution API

#### 🔑 Critical Understanding: Docker Networking

**⚠️ IMPORTANT:** Evolution API runs **inside a Docker container**. From inside the container:
- ❌ `localhost:XXXX` refers to the container itself (won't work!)
- ✅ `host.docker.internal:XXXX` refers to your computer (works!)

**Replace `XXXX` with your actual n8n port!** Use `docker ps` to find it.

```
┌─────────────────────────────────────┐
│  Your Computer (Host)               │
│                                     │
│  n8n: localhost:5677 (example)      │
│         ↑                           │
│         │ Must use special hostname │
│  ┌──────┴────────────────────────┐ │
│  │  Docker Container             │ │
│  │  Evolution API                │ │
│  │                               │ │
│  │  localhost:5677 = ❌ Wrong    │ │
│  │  host.docker.internal:5677 ✅ │ │
│  └───────────────────────────────┘ │
└─────────────────────────────────────┘

⚠️ Replace 5677 with YOUR actual n8n port in all configurations!
```

#### Step-by-Step Configuration:

1. **Open Evolution API Manager**: http://localhost:8080/manager

2. **Select your WhatsApp instance**

3. **Navigate to**: **Integrations** → **N8N**

4. **Click the "+ n8n" button** (top right)

5. **Fill in the configuration form:**

```
┌─────────────────────────────────────────────┐
│ N8N Bot Configuration                       │
├─────────────────────────────────────────────┤
│                                             │
│ Description:                                │
│ ┌─────────────────────────────────────────┐ │
│ │ My N8N Bot                              │ │
│ └─────────────────────────────────────────┘ │
│                                             │
│ Enabled: ✅ ON (Toggle)                     │
│                                             │
│ Webhook URL: ⚠️ CRITICAL!                   │
│ ┌─────────────────────────────────────────┐ │
│ │ http://host.docker.internal:5677/webhook/evoapi │
│ └─────────────────────────────────────────┘ │
│ ⚠️ Replace 5677 with YOUR actual n8n port! │
│                                             │
│ Basic Auth User: (leave empty)              │
│ Basic Auth Password: (leave empty)          │
│                                             │
│ Trigger Type: All                           │
│ Trigger Operator: contains                  │
│ Trigger Value: (leave empty)                │
│                                             │
│ Expire in minutes: 0                        │
│ Keyword Finish: #sair                       │
│ Default Delay Message: 0                    │
│ Unknown Message: (leave empty)              │
│                                             │
│ Listening from me: ❌ OFF                   │
│ Stop bot from me: ✅ ON                     │
│ Keep open: ❌ OFF                           │
│                                             │
│ Debounce Time: 0                            │
│ Split Messages: ❌ OFF                      │
│ Time Per Char: 0                            │
│                                             │
│ Ignore Jids: (leave empty)                  │
│                                             │
│         [Save]  [Cancel]                    │
└─────────────────────────────────────────────┘
```

6. **Click Save**

7. **Verify the bot is enabled** - you should see a green toggle next to your bot

### 6.5 Understanding Each N8N Setting

| Setting | Recommended Value | Purpose |
|---------|------------------|---------|
| **Webhook URL** | `http://host.docker.internal:5677/webhook/evoapi` | Where to send messages (Replace 5677 with YOUR n8n port - use `docker ps` to find it) |
| **Trigger Type** | `all` | Trigger on every message (best for testing) |
| **Expire in minutes** | `0` | Never expire sessions |
| **Default Delay Message** | `0` | No artificial delay (faster responses) |
| **Debounce Time** | `0` | Process messages immediately |
| **Time Per Char** | `0` | No typing simulation delay |
| **Listening from me** | OFF | Only respond to others, not your own messages |
| **Stop bot from me** | ON | You can stop bot by sending a message |
| **Split Messages** | OFF | Don't split responses (faster) |

### 6.6 Test Your N8N Integration

1. **Send a WhatsApp message** to your Evolution API number
2. **Check n8n workflow executions** - you should see a new execution
3. **Check if you receive a response** in WhatsApp

**Expected response time:** < 0.5 seconds with local n8n! ⚡

### 6.7 Common N8N Integration Issues

| Issue | Symptom | Solution |
|-------|---------|----------|
| **502 Bad Gateway** | Messages not reaching n8n, 502 errors in logs | Using wrong URL - must use `host.docker.internal` not `localhost` |
| **Connection Refused** | `ECONNREFUSED` errors in logs | Wrong hostname - use `host.docker.internal:XXXX` (replace XXXX with your n8n port) |
| **Messages not triggering bot** | No `[N8nService]` logs | Check trigger settings, ensure `triggerType: all` |
| **N8N disabled error** | Can't create n8n bot in Manager | Set `N8N_ENABLED=true` in `.env` |
| **Slow responses** | 2+ seconds delay | Check `delayMessage`, `timePerChar`, `debounceTime` - all should be 0 |

---

## Common Commands

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f api
docker-compose logs -f evolution-postgres
docker-compose logs -f redis

# Last 50 lines only
docker-compose logs --tail 50 api

# Logs from last 5 minutes
docker-compose logs --since 5m api
```

### Stop Services

```bash
# Stop all services (data is preserved)
docker-compose down

# Stop and remove all data (⚠️ destructive!)
docker-compose down -v
```

### Restart Services

```bash
# Restart all services
docker-compose restart

# Restart specific service
docker-compose restart api
```

### Update to Latest Version

```bash
# Pull latest images
docker-compose pull

# Restart with new images
docker-compose up -d
```

### Access Container Shell

```bash
# Access API container
docker exec -it evolution_api bash

# Access PostgreSQL
docker exec -it evolution_postgres psql -U postgres -d evolution_db

# Access Redis
docker exec -it evolution_redis redis-cli
```

---

## Troubleshooting

### Problem: Port 8080 is already in use

**Solution:** Change the port in `docker-compose.yaml`:

```yaml
ports:
  - "8081:8080"  # Changed from 8080 to 8081
```

Then access the API at `http://localhost:8081`

### Problem: Cannot find `dokploy-network`

**Error:**
```
ERROR: Network dokploy-network declared as external, but could not be found
```

**Solution:** Remove all `dokploy-network` references from `docker-compose.yaml` (see Step 3)

### Problem: Database connection failed

**Check:**
1. Is PostgreSQL container running? `docker-compose ps`
2. Are environment variables correct in `.env`?
3. Did you use `evolution-postgres` (not `localhost`)?

**Solution:**
```bash
# View PostgreSQL logs
docker-compose logs evolution-postgres

# Restart PostgreSQL
docker-compose restart evolution-postgres

# Or recreate all containers
docker-compose down
docker-compose up -d
```

### Problem: Redis connection failed

**Check:**
1. Is Redis container running? `docker-compose ps`
2. Is `CACHE_REDIS_URI` set to `redis://evolution-redis:6379`?

**Solution:**
```bash
# View Redis logs
docker-compose logs redis

# Restart Redis
docker-compose restart redis
```

### Problem: N8N Integration Disabled

**Error:** "n8n is disabled" when trying to create n8n bot

**Solution:**
1. Open `.env` file
2. Find or add: `N8N_ENABLED=true`
3. Save file
4. Restart Evolution API: `docker-compose restart api`
5. Wait 30 seconds and refresh Manager

### Problem: N8N Connection Refused (ECONNREFUSED)

**Error in logs:**
```
ERROR [N8nService] ECONNREFUSED localhost:XXXX
```

**Solution:** 
You're using `localhost` in the webhook URL. Change to:
```
http://host.docker.internal:5677/webhook/evoapi
```

**⚠️ Replace `5677` with YOUR actual n8n port!** (find it with `docker ps | grep n8n`).

**Why?** Evolution API is inside a Docker container. From inside the container, `localhost` refers to the container itself, not your computer!

### Problem: N8N 502 Bad Gateway

**Error:** 502 Bad Gateway errors in logs

**Solution:**
You're using an external/cloud URL for local n8n. This causes:
- Slow responses (1+ second)
- Intermittent 502 errors
- Unreliable delivery

**Change to local URL:**
```
http://host.docker.internal:5677/webhook/evoapi
```
**⚠️ Replace `5677` with YOUR n8n port!** (check with `docker ps | grep n8n`).

**Performance comparison:**
- Local URL: ~67ms ⚡
- External/Cloud URL: ~1000ms + 502 errors 🐌

### Problem: Manager UI Slow to Load

**Solution:** Reduce logging in `.env`:

```ini
# FROM:
LOG_LEVEL=ERROR,WARN,DEBUG,INFO,LOG,VERBOSE,DARK,WEBHOOKS,WEBSOCKET

# TO:
LOG_LEVEL=ERROR,WARN,INFO
```

Then restart: `docker-compose restart api`

### Problem: Messages Reaching Evolution API but Not N8N

**Check:**
1. Is n8n bot **enabled** in Manager? (green toggle)
2. Is **Trigger Type** set to `all`?
3. Is webhook URL using `host.docker.internal`?
4. Is n8n workflow **active** in n8n UI?

**View n8n-specific logs:**
```bash
# Windows PowerShell
docker-compose logs -f api | Select-String -Pattern "N8nService"

# macOS/Linux
docker-compose logs -f api | grep -i n8nservice
```

You should see:
```
[N8nService] Processing message...
```

If you DON'T see `[N8nService]` logs, the bot is not triggering!

---

## Performance Optimization

### Optimize Logging

**Problem:** Too much logging slows down the application and makes the Manager UI slow.

**Solution:** Edit `.env`:

```ini
# Minimal logging (fastest)
LOG_LEVEL=ERROR,WARN,INFO
LOG_BAILEYS=error

# Moderate logging (balanced)
LOG_LEVEL=ERROR,WARN,INFO,LOG

# Full logging (debugging only)
LOG_LEVEL=ERROR,WARN,DEBUG,INFO,LOG,VERBOSE,DARK,WEBHOOKS,WEBSOCKET
```

### Optimize N8N Response Times

**For fastest responses:**

```json
{
  "delayMessage": 0,      // No artificial delay
  "timePerChar": 0,       // No typing simulation
  "debounceTime": 0,      // Process immediately
  "splitMessages": false  // Don't split responses
}
```

**n8n workflow optimization:**
- Set webhook "Respond" to: **Immediately**
- Remove unnecessary "Wait" nodes
- Keep workflows simple and fast

### Expected Performance Benchmarks

| Setup | Response Time | Reliability |
|-------|---------------|-------------|
| **Local Evolution + Local N8N** | 100-500ms | 99.9% |
| **Local Evolution + Remote N8N** | 1-10 seconds | 60-80% (may have errors) |
| **Remote Evolution + Remote N8N (same region)** | 200-800ms | 95%+ |

---

## Security Best Practices

1. ✅ **Change the default API key** to a strong random string
2. ✅ **Use environment variables** - never hardcode credentials
3. ✅ **Don't commit `.env`** to version control
4. ✅ **Enable HTTPS in production** (use reverse proxy like Nginx)
5. ✅ **Restrict network access** - use firewall rules
6. ✅ **Regular backups** - backup Docker volumes regularly
7. ✅ **Keep updated** - regularly pull latest images

---

## Understanding Evolution API + N8N Architecture

### Complete Flow Diagram

```
┌──────────────────────────────────────────────────────────────┐
│  Your Computer                                               │
│                                                              │
│  📱 WhatsApp Message                                        │
│         ↓                                                    │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  Docker Container: evolution_api                       │ │
│  │                                                        │ │
│  │  1️⃣ Receives WhatsApp message (via Baileys)          │ │
│  │  2️⃣ Checks if n8n bot is enabled & trigger matches   │ │
│  │  3️⃣ Sends to n8n webhook:                            │ │
│  │     http://host.docker.internal:5677/webhook/evoapi   │ │
│  │     (Replace 5677 with YOUR n8n port!)                │ │
│  │         │                                              │ │
│  └─────────┼──────────────────────────────────────────────┘ │
│            │                                                 │
│            ↓ (goes to host machine)                         │
│                                                              │
│  📦 n8n (localhost:5677 - use your actual port)             │
│  4️⃣ Receives webhook request                               │
│  5️⃣ Processes workflow (AI, logic, etc.)                   │
│  6️⃣ Responds immediately with output                       │
│            │                                                 │
│            ↓ (response back)                                │
│  ┌────────┼──────────────────────────────────────────────┐ │
│  │  Docker Container: evolution_api                       │ │
│  │         │                                              │ │
│  │  7️⃣ Receives n8n response                             │ │
│  │  8️⃣ Sends reply to WhatsApp                           │ │
│  └────────────────────────────────────────────────────────┘ │
│         ↓                                                    │
│  📱 User receives response                                  │
│                                                              │
│  ⏱️ Total time: ~100-500ms (with optimized settings)       │
└──────────────────────────────────────────────────────────────┘
```

---

## Docker Networking Reference

### Service Names in This Setup

| Container Name | Service Name (DNS) | Purpose |
|---------------|-------------------|---------|
| `evolution_api` | N/A | Main application |
| `evolution_postgres` | `evolution-postgres` | Database |
| `evolution_redis` | `evolution-redis` | Cache |

### Special Hostnames

| Hostname | From Where | Points To |
|----------|-----------|-----------|
| `localhost` | Your computer | Your computer |
| `localhost` | Inside container | The container itself |
| `host.docker.internal` | Inside container | Your computer |
| `evolution-postgres` | Inside container | PostgreSQL container |
| `evolution-redis` | Inside container | Redis container |

### Port Mapping Example

```yaml
ports:
  - "8080:8080"
```

Means:
- **Left side (8080)**: Port on YOUR computer
- **Right side (8080)**: Port inside the container
- Access via: `http://localhost:8080` (from your computer)

---

## Additional Resources

- **GitHub Repository:** https://github.com/EvolutionAPI/evolution-api
- **Official Documentation:** https://doc.evolution-api.com
- **Discord Community:** https://evolution-api.com/discord
- **WhatsApp Group:** https://evolution-api.com/whatsapp
- **Report Issues:** https://github.com/EvolutionAPI/evolution-api/issues

---

## Support

If you encounter any issues:

1. Check this README's [Troubleshooting](#troubleshooting) section
2. Review logs: `docker-compose logs -f`
3. Search existing [GitHub Issues](https://github.com/EvolutionAPI/evolution-api/issues)
4. Join the [Discord Community](https://evolution-api.com/discord)
5. Ask your instructor for help

---

## License

Evolution API is licensed under Apache License 2.0.  
© 2025 Evolution API

---

**Happy Coding! 🚀**

**Remember:** Local Evolution API + Local n8n = Use `host.docker.internal`!