# ConnectSafely API - MCP Server

An MCP (Model Context Protocol) server that provides tools for interacting with the ConnectSafely LinkedIn API. This server enables LinkedIn automation capabilities through a standardized MCP interface.

## Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Running the Server](#running-the-server)
- [Available Tools](#available-tools)
- [Usage Examples](#usage-examples)
- [Troubleshooting](#troubleshooting)
- [Development](#development)

## Features

- **LinkedIn Automation**: Send messages, connection requests, follow users, and more
- **Post Management**: React, comment, and search LinkedIn posts
- **Profile Management**: Fetch profile information and check relationships
- **Group Management**: Get group members and interact within LinkedIn groups
- **Dual Transport Support**: Works with both stdio and HTTP transports
- **Type-Safe**: Built with TypeScript and runtime validation via Zod

## Prerequisites

- **Node.js**: Version 20.0.0 or higher
- **npm**: Version 7 or higher (comes with Node.js)
- **ConnectSafely API Key**: Get your API key from [ConnectSafely](https://connectsafely.ai)

## Installation

1. **Clone or navigate to the project directory**:
   ```bash
   cd mcp-server-claude
   ```

2. **Install dependencies**:
   ```bash
   npm install
   ```

3. **Build the project**:
   ```bash
   npm run build
   ```

## Configuration

### Environment Variables

Create a `.env` file in the project root directory with your API credentials:

```env
# Bearer Token Authentication (Required for most tools)
BEARER_TOKEN_BEARERAUTH=your_api_key_here
```

### Getting Your API Key

1. Sign up or log in to [ConnectSafely](https://connectsafely.ai)
2. Navigate to your API settings
3. Generate or copy your API key
4. Add it to your `.env` file as shown above

### Optional: OAuth2 Configuration

If you need OAuth2 authentication (for certain advanced features), add these to your `.env`:

```env
OAUTH_CLIENT_ID_SCHEMENAME=your_client_id
OAUTH_CLIENT_SECRET_SCHEMENAME=your_client_secret
OAUTH_SCOPES_SCHEMENAME=required_scopes
```

## Running the Server

### Stdio Transport (Default)

For use with MCP clients that communicate via stdio:

```bash
npm start
```

This will:
1. Build the project automatically (via `prestart` script)
2. Start the MCP server on stdio

### HTTP Transport

For use with HTTP-based MCP clients:

```bash
npm run start:http
```

The server will start on `http://localhost:3000` with:
- **MCP Endpoint**: `POST http://localhost:3000/mcp`
- **Health Check**: `GET http://localhost:3000/health`

### Development Mode

For development with automatic rebuilding:

```bash
# Type check only
npm run typecheck

# Build only
npm run build
```

## Available Tools

The server provides the following LinkedIn automation tools:

### Connection & Messaging

- **`follow-user`**: Follow or unfollow a LinkedIn user
- **`send-message`**: Send a message to a LinkedIn user (supports group messaging)
- **`send-connection-request`**: Send a connection request with optional custom message
- **`check-relationship`**: Check relationship status with a LinkedIn user
- **`check-relationship-specific-account`**: Check relationship using a specific account

### Posts & Engagement

- **`get-latest-posts`**: Get the latest posts from a LinkedIn user
- **`react-to-post`**: React to a LinkedIn post (LIKE, PRAISE, APPRECIATION, etc.)
- **`comment-on-post`**: Comment on a LinkedIn post
- **`get-post-comments`**: Get comments from a post with pagination
- **`get-all-post-comments`**: Get all comments with automatic pagination
- **`search-posts`**: Search LinkedIn posts by keywords with filters
- **`scrape-post`**: Scrape public LinkedIn post content (no auth required)

### Profile & Organization

- **`fetch-profile`**: Fetch detailed LinkedIn profile information
- **`get-linkedin-organizations`**: Retrieve LinkedIn organizations/company pages you manage

### Groups

- **`get-group-members`**: Fetch members of a LinkedIn group by group ID
- **`get-group-members-by-url`**: Fetch group members using the group URL

### Comments

- **`reply-to-comment`**: Reply to a comment on a LinkedIn post (not yet implemented)

## Usage Examples

### Example 1: Send a Connection Request

```json
{
  "tool": "send-connection-request",
  "arguments": {
    "requestBody": {
      "profileId": "john-doe",
      "customMessage": "Hi John, I'd like to connect with you!"
    }
  }
}
```

### Example 2: Search for Posts

```json
{
  "tool": "search-posts",
  "arguments": {
    "requestBody": {
      "keywords": "TypeScript",
      "count": 10,
      "datePosted": "past-week",
      "sortBy": "relevance",
      "authorJobTitles": ["CTO", "Senior Developer"]
    }
  }
}
```

### Example 3: Comment on a Post

```json
{
  "tool": "comment-on-post",
  "arguments": {
    "requestBody": {
      "postUrl": "https://www.linkedin.com/posts/...",
      "comment": "Great insights! Thanks for sharing.",
      "tagPostAuthor": true
    }
  }
}
```

### Example 4: Get Group Members

```json
{
  "tool": "get-group-members",
  "arguments": {
    "requestBody": {
      "groupId": "123456",
      "count": 50,
      "membershipStatuses": ["MEMBER", "MANAGER"]
    }
  }
}
```

### Example 5: Fetch Profile Information

```json
{
  "tool": "fetch-profile",
  "arguments": {
    "requestBody": {
      "profileId": "jane-smith",
      "includeContact": true,
      "includeGeoLocation": false
    }
  }
}
```

## Troubleshooting

### Server Won't Start

1. **Check Node.js version**:
   ```bash
   node --version  # Should be >= 20.0.0
   ```

2. **Verify dependencies are installed**:
   ```bash
   npm install
   ```

3. **Check for build errors**:
   ```bash
   npm run build
   ```

### Authentication Errors

1. **Verify your API key is set**:
   - Check that `.env` file exists in the project root
   - Ensure `BEARER_TOKEN_BEARERAUTH` is set correctly
   - No quotes needed around the API key value

2. **Check API key validity**:
   - Verify the key is active in your ConnectSafely dashboard
   - Ensure there are no extra spaces or newlines

### HTTP Transport Issues

1. **Port already in use**:
   - The default port is 3000
   - Change the port in `src/index.ts` if needed (line 686)

2. **CORS errors**:
   - CORS is enabled by default for all routes
   - Check browser console for specific CORS error messages

### Tool Execution Errors

1. **Validation errors**:
   - Check the tool's input schema requirements
   - Ensure all required fields are provided
   - Verify data types match (strings, numbers, booleans)

2. **API errors**:
   - Check the error message for specific API response details
   - Verify your account has the necessary permissions
   - Some tools may require specific LinkedIn account types

## Development

### Project Structure

```
mcp-server-claude/
├── src/
│   ├── index.ts              # Main MCP server and tool definitions
│   └── streamable-http.ts    # HTTP transport implementation
├── build/                    # Compiled JavaScript (generated)
├── public/                   # Static files for web interface
├── package.json              # Dependencies and scripts
├── tsconfig.json             # TypeScript configuration
└── .env                      # Environment variables (create this)
```

### Adding New Tools

1. Add tool definition to `toolDefinitionMap` in `src/index.ts`
2. Include proper JSON Schema for input validation
3. Specify security requirements
4. Rebuild: `npm run build`

### Testing

Run type checking:
```bash
npm run typecheck
```

### Code Style

- TypeScript strict mode enabled
- ES modules (`type: "module"`)
- Target: ES2022, Node16 module resolution
- All errors logged to stderr (console.error)

## Security Notes

- **Never commit your `.env` file** to version control
- API keys are sensitive credentials - keep them secure
- The server validates all inputs using Zod schemas
- Security schemes are automatically applied based on tool requirements

## Support

For issues related to:
- **MCP Server**: Check this repository's issues
- **ConnectSafely API**: Contact [ConnectSafely support](https://connectsafely.ai/support)
- **MCP Protocol**: See [MCP documentation](https://modelcontextprotocol.io)

## License

This project is private and proprietary.

---

**Version**: 1.0.0  
**Generated**: 2025-11-21  
**API Base URL**: https://api.connectsafely.ai

