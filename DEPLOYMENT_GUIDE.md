# Remote MCP Server Deployment Guide

This guide explains how to deploy your ConnectSafely MCP server remotely and configure Claude Desktop to use it with per-user API keys.

## Overview

Your MCP server supports **remote deployment** via HTTP transport. The server can be deployed to cloud platforms and accessed via URL, allowing multiple users to connect with their own API keys.

**Server Endpoints:**
- **MCP Endpoint**: `POST https://your-url.com/mcp`
- **Health Check**: `GET https://your-url.com/health`

## Quick Start

1. **Deploy to a cloud platform** (Railway, Render, Fly.io, etc.)
2. **Get your deployment URL**
3. **Configure Claude Desktop** with your API key (see Claude Config section)

## Deployment Platforms

### Railway (Recommended - Easiest)

**Why Railway**: Simple setup, automatic HTTPS, free tier available.

#### Steps:

1. **Sign up** at [railway.app](https://railway.app)

2. **Create New Project**:
   - Click "New Project"
   - Select "Deploy from GitHub repo" (connect your repository)

3. **Configure Service**:
   - Railway auto-detects Node.js
   - **Start Command**: `node build/index.js --transport=streamable-http`
   - **Build Command**: `npm install && npm run build`

4. **Add Environment Variables**:
   - Go to **Variables** tab
   - Add: `PORT` = `3000` (optional, Railway auto-assigns)
   - **Note**: You don't need to set `BEARER_TOKEN_BEARERAUTH` here - users will provide their own API keys in Claude Desktop config

5. **Deploy**:
   - Railway automatically deploys on git push
   - Get your URL from service settings (e.g., `https://your-app.up.railway.app`)

6. **Test**:
   ```bash
   curl https://your-app.up.railway.app/health
   ```
   Should return: `{"status":"OK","server":"connectsafely-api","version":"1.0.0"}`

---

### Render

**Why Render**: Simple deployment, free tier, automatic HTTPS.

#### Steps:

1. **Sign up** at [render.com](https://render.com)

2. **Create New Web Service**:
   - Connect your GitHub repository
   - Select the repository

3. **Configure**:
   - **Name**: `connectsafely-mcp-server`
   - **Environment**: `Node`
   - **Build Command**: `npm install && npm run build`
   - **Start Command**: `node build/index.js --transport=streamable-http`
   - **Plan**: Free tier available

4. **Add Environment Variables**:
   - Go to **Environment** tab
   - Add: `PORT` = `3000` (optional)

5. **Deploy**:
   - Click "Create Web Service"
   - Your URL: `https://your-app.onrender.com`

---

### Fly.io

**Why Fly.io**: Global edge deployment, good performance.

#### Steps:

1. **Install Fly CLI**:
   ```bash
   # macOS/Linux
   curl -L https://fly.io/install.sh | sh
   
   # Windows (PowerShell)
   iwr https://fly.io/install.ps1 -useb | iex
   ```

2. **Login**:
   ```bash
   fly auth signup
   fly auth login
   ```

3. **Initialize App**:
   ```bash
   fly launch
   ```
   - Answer prompts (use defaults)
   - Don't deploy yet

4. **Create `fly.toml`**:
   ```toml
   app = "connectsafely-mcp-server"
   primary_region = "iad"

   [build]
     builder = "paketobuildpacks/builder:base"

   [env]
     PORT = "3000"
     NODE_ENV = "production"

   [[services]]
     internal_port = 3000
     protocol = "tcp"
     processes = ["app"]

     [[services.ports]]
       port = 80
       handlers = ["http"]
       force_https = true

     [[services.ports]]
       port = 443
       handlers = ["tls", "http"]

   [[services.http_checks]]
     interval = "10s"
     timeout = "2s"
     grace_period = "5s"
     method = "GET"
     path = "/health"
   ```

5. **Deploy**:
   ```bash
   fly deploy
   ```

6. **Get URL**:
   ```bash
   fly info
   ```
   Your URL: `https://connectsafely-mcp-server.fly.dev`

---

### DigitalOcean App Platform

**Why DigitalOcean**: Reliable, good for production.

#### Steps:

1. **Sign up** at [digitalocean.com](https://digitalocean.com)

2. **Create New App**:
   - Go to App Platform
   - Click "Create App"
   - Connect GitHub repository

3. **Configure**:
   - **Resource Type**: Web Service
   - **Build Command**: `npm install && npm run build`
   - **Run Command**: `node build/index.js --transport=streamable-http`
   - **HTTP Port**: `3000`

4. **Deploy**:
   - Click "Create Resources"
   - Your URL: `https://your-app.ondigitalocean.app`

---

## Configuring Claude Desktop

After deploying your server, configure Claude Desktop to connect to it. Each user will use their own API key.

### Option 1: Local Stdio with Remote Server (Current Recommended)

Since Claude Desktop primarily supports stdio transport, users can run the server locally but it will still work with the remote deployment pattern. Each user provides their own API key:

**Config Location:**
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

**Configuration:**
```json
{
  "mcpServers": {
    "connectsafely-api": {
      "command": "node",
      "args": ["C:/path/to/mcp-server-claude/build/index.js"],
      "env": {
        "BEARER_TOKEN_BEARERAUTH": "YOUR_API_KEY_HERE"
      }
    }
  }
}
```

**Instructions for Users:**
1. Clone or download the MCP server code
2. Run `npm install && npm run build`
3. Update the config file:
   - Replace `C:/path/to/mcp-server-claude/build/index.js` with the actual path
   - Replace `YOUR_API_KEY_HERE` with their ConnectSafely API key
4. Restart Claude Desktop

### Option 2: HTTP Transport (If Supported by Claude Desktop)

If your Claude Desktop version supports HTTP transport, users can connect directly to the remote server:

**Configuration:**
```json
{
  "mcpServers": {
    "connectsafely-api": {
      "url": "https://your-deployed-url.com/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY_HERE"
      }
    }
  }
}
```

**Instructions for Users:**
1. Replace `https://your-deployed-url.com/mcp` with the actual deployed URL
2. Replace `YOUR_API_KEY_HERE` with their ConnectSafely API key
3. Restart Claude Desktop

**Note**: HTTP transport support in Claude Desktop may vary by version. If this doesn't work, use Option 1.

---

## Testing Your Deployment

### 1. Health Check

Test that your server is running:

```bash
curl https://your-deployed-url.com/health
```

**Expected Response:**
```json
{
  "status": "OK",
  "server": "connectsafely-api",
  "version": "1.0.0"
}
```

### 2. MCP Endpoint Test

Test the MCP endpoint with an initialize request:

```bash
curl -X POST https://your-deployed-url.com/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "initialize",
    "params": {
      "protocolVersion": "2024-11-05",
      "capabilities": {},
      "clientInfo": {
        "name": "test-client",
        "version": "1.0.0"
      }
    }
  }'
```

---

## Troubleshooting

### Server Not Starting

**Check:**
- Build logs for compilation errors
- Node.js version (must be >= 20.0.0)
- Port is not already in use

**Solution:**
```bash
# Test locally first
npm run build
npm run start:http

# Check if port 3000 is available
# On Windows:
netstat -ano | findstr :3000
```

### Environment Variables

**Note**: For remote deployment, you don't need to set `BEARER_TOKEN_BEARERAUTH` on the server. Each user provides their own API key in their Claude Desktop config.

### Connection Errors

**Check:**
- URL is accessible (test `/health` endpoint)
- HTTPS is enabled (most platforms do this automatically)
- CORS is configured (already enabled in code)

**Solution:**
```bash
# Test connectivity
curl -v https://your-deployed-url.com/health
```

### Claude Desktop Not Connecting

**If using stdio (Option 1):**
- Verify local Node.js is installed (>= 20.0.0)
- Check file paths are correct
- Ensure `npm run build` completed successfully
- Verify API key is correct in config

**If using HTTP (Option 2):**
- Verify Claude Desktop version supports HTTP transport
- Check config file syntax (valid JSON)
- Restart Claude Desktop after config changes
- Verify the deployed URL is accessible

---

## Security Best Practices

1. **Never commit API keys** to version control
2. **Each user manages their own API key** in their Claude Desktop config
3. **Enable HTTPS** (automatic on most platforms)
4. **Monitor usage** and set up alerts
5. **CORS is enabled** for all origins (consider restricting if needed)

To restrict CORS, update `src/streamable-http.ts`:

```typescript
app.use('*', cors({
  origin: ['https://claude.ai', 'https://claude-desktop.app']
}));
```

---

## Quick Reference

### Deployment URLs by Platform

- **Railway**: `https://your-app.up.railway.app`
- **Render**: `https://your-app.onrender.com`
- **Fly.io**: `https://your-app.fly.dev`
- **DigitalOcean**: `https://your-app.ondigitalocean.app`

### Required Environment Variables (Server)

```bash
PORT=3000  # Optional, most platforms auto-assign
```

**Note**: `BEARER_TOKEN_BEARERAUTH` is NOT needed on the server - users provide it in their Claude Desktop config.

### Start Command

```bash
node build/index.js --transport=streamable-http
```

### Health Check

```bash
curl https://your-url.com/health
```

---

## Next Steps

1. ✅ Deploy to your chosen platform
2. ✅ Test health endpoint
3. ✅ Share deployment URL with users
4. ✅ Users configure Claude Desktop with their API keys
5. ⚠️ Monitor logs for errors
6. ⚠️ Set up alerts/notifications

---

**Need Help?** Check platform-specific documentation or review server logs for detailed error messages.

