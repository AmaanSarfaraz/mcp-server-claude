# Claude Desktop Configuration Guide

This guide shows how to configure Claude Desktop to use the ConnectSafely MCP server with your own API key.

## Config File Location

**macOS:**
```
~/Library/Application Support/Claude/claude_desktop_config.json
```

**Windows:**
```
%APPDATA%\Claude\claude_desktop_config.json
```

## Configuration Options

### Option 1: Local Stdio Transport (Recommended)

Run the server locally with your API key. This is the most reliable method currently.

**Configuration:**
```json
{
  "mcpServers": {
    "connectsafely-api": {
      "command": "node",
      "args": ["C:/Users/YourName/Desktop/mcp-server-claude/build/index.js"],
      "env": {
        "BEARER_TOKEN_BEARERAUTH": "YOUR_API_KEY_HERE"
      }
    }
  }
}
```

**Setup Steps:**
1. **Clone or download** the MCP server repository
2. **Install dependencies**: `npm install`
3. **Build the project**: `npm run build`
4. **Update the config**:
   - Replace the path in `args` with your actual path to `build/index.js`
   - Replace `YOUR_API_KEY_HERE` with your ConnectSafely API key
5. **Restart Claude Desktop**

**Example for Windows:**
```json
{
  "mcpServers": {
    "connectsafely-api": {
      "command": "node",
      "args": ["C:/Users/sarfa/Desktop/mcp-server-claude/build/index.js"],
      "env": {
        "BEARER_TOKEN_BEARERAUTH": "e8bad5be-ef4b-4741-8902-281b019a9b2b"
      }
    }
  }
}
```

**Example for macOS/Linux:**
```json
{
  "mcpServers": {
    "connectsafely-api": {
      "command": "node",
      "args": ["/Users/yourname/mcp-server-claude/build/index.js"],
      "env": {
        "BEARER_TOKEN_BEARERAUTH": "your-api-key-here"
      }
    }
  }
}
```

### Option 2: HTTP Transport (If Supported)

If your Claude Desktop version supports HTTP transport, you can connect directly to a remote server.

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

**Setup Steps:**
1. **Get the deployed server URL** (from deployment guide)
2. **Update the config**:
   - Replace `https://your-deployed-url.com/mcp` with the actual URL
   - Replace `YOUR_API_KEY_HERE` with your ConnectSafely API key
3. **Restart Claude Desktop**

**Note**: HTTP transport support may vary by Claude Desktop version. If this doesn't work, use Option 1.

## Multiple MCP Servers

If you have multiple MCP servers, add this entry to your existing config:

```json
{
  "mcpServers": {
    "other-server": {
      "command": "node",
      "args": ["path/to/other/server.js"]
    },
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

## Getting Your API Key

1. Sign up or log in to [ConnectSafely](https://connectsafely.ai)
2. Navigate to your API settings
3. Generate or copy your API key
4. Add it to your Claude Desktop config

## Troubleshooting

### Server Not Starting

**Check:**
- Node.js is installed (version >= 20.0.0)
- File path is correct (use forward slashes `/` or double backslashes `\\` on Windows)
- Project is built (`npm run build` completed successfully)

**Verify Node.js:**
```bash
node --version  # Should be >= 20.0.0
```

**Verify Build:**
```bash
cd mcp-server-claude
npm run build
# Check that build/index.js exists
```

### Invalid JSON Error

**Check:**
- Config file is valid JSON (no trailing commas, proper quotes)
- Use a JSON validator if needed

**Common Issues:**
- Trailing commas after last item
- Missing quotes around keys or values
- Comments (JSON doesn't support comments)

### API Key Not Working

**Check:**
- API key is correct (no extra spaces or newlines)
- API key is active in your ConnectSafely account
- Environment variable name is exactly `BEARER_TOKEN_BEARERAUTH`

**Test API Key:**
You can test your API key by making a direct API call:
```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
     https://api.connectsafely.ai/linkedin/organizations
```

### Path Issues on Windows

**Use forward slashes or double backslashes:**
```json
// ✅ Correct
"C:/Users/sarfa/Desktop/mcp-server-claude/build/index.js"
"C:\\Users\\sarfa\\Desktop\\mcp-server-claude\\build\\index.js"

// ❌ Incorrect
"C:\Users\sarfa\Desktop\mcp-server-claude\build\index.js"
```

## Security Notes

1. **Never share your API key** - it's unique to your account
2. **Keep your config file private** - it contains your API key
3. **Don't commit config files** to version control
4. **Rotate your API key** if you suspect it's compromised

## After Configuration

1. **Save the config file**
2. **Restart Claude Desktop** completely (quit and reopen)
3. **Check Claude Desktop** - the MCP server should appear in available tools
4. **Test a tool** - try using one of the LinkedIn tools to verify it works

## Example Complete Config

Here's a complete example config file:

```json
{
  "mcpServers": {
    "connectsafely-api": {
      "command": "node",
      "args": ["C:/Users/sarfa/Desktop/mcp-server-claude/build/index.js"],
      "env": {
        "BEARER_TOKEN_BEARERAUTH": "e8bad5be-ef4b-4741-8902-281b019a9b2b"
      }
    }
  }
}
```

**Remember to:**
- Replace the path with your actual path
- Replace the API key with your own API key
- Use forward slashes or double backslashes for Windows paths

---

**Need Help?** Check the deployment guide or review Claude Desktop logs for error messages.

