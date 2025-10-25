---
name: mcp-builder
description: "Comprehensive guide for building Model Context Protocol (MCP) servers with support for tools, resources, prompts, and authentication. Use when: (1) Creating custom MCP servers, (2) Integrating external APIs with Claude, (3) Building tool servers for specialized domains, (4) Creating resource providers for documentation, (5) Implementing authentication and security"
---

# MCP Builder - Comprehensive Guide to Building Model Context Protocol Servers

## Table of Contents
1. [MCP Fundamentals](#mcp-fundamentals)
2. [Core Components](#core-components)
3. [Server Implementation](#server-implementation)
4. [Detailed Workflows](#detailed-workflows)
5. [Code Examples](#code-examples)
6. [Best Practices](#best-practices)
7. [Integration Guides](#integration-guides)
8. [Common Pitfalls](#common-pitfalls)
9. [Testing and Debugging](#testing-and-debugging)
10. [Production Deployment](#production-deployment)

---

## MCP Fundamentals

### What is MCP?

Model Context Protocol (MCP) is an open standard created by Anthropic that enables AI assistants like Claude to securely connect to external data sources and tools. Think of it as a universal adapter that allows Claude to interact with any system, API, or data source through a standardized interface.

**Key Benefits:**
- **Standardization**: One protocol for all integrations
- **Security**: Built-in authentication and permission controls
- **Flexibility**: Support for tools, resources, and prompts
- **Scalability**: Designed for production workloads
- **Modularity**: Create reusable MCP servers for different domains

### Architecture Overview

MCP follows a client-server architecture:

```
┌─────────────┐         ┌─────────────┐         ┌──────────────┐
│   Claude    │ ←──MCP──→ │ MCP Server  │ ←──────→ │ External API │
│  (Client)   │         │  (Your Code) │         │  Database    │
└─────────────┘         └─────────────┘         └──────────────┘
```

**Components:**
- **Client**: Claude Desktop, Claude Code, or custom applications
- **Server**: Your MCP implementation (Python, TypeScript, etc.)
- **Transport**: Communication channel (stdio, HTTP, SSE)
- **Protocol**: Standardized message format (JSON-RPC 2.0)

### Protocol Specification Basics

MCP uses JSON-RPC 2.0 for message exchange:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "calculator_add",
    "arguments": {
      "a": 5,
      "b": 3
    }
  }
}
```

**Message Types:**
- **Request**: Client → Server (expects response)
- **Response**: Server → Client (answers request)
- **Notification**: One-way message (no response expected)

### Transport Mechanisms

**1. STDIO (Standard Input/Output)**
- Best for: Local tools, command-line applications
- Pros: Simple, secure, no network configuration
- Cons: Limited to single machine
- Use case: Claude Desktop integration

```python
# Server runs as subprocess, communicates via stdin/stdout
# Claude Desktop launches: python server.py
```

**2. HTTP with Server-Sent Events (SSE)**
- Best for: Web services, remote servers
- Pros: Network accessible, scalable, firewall-friendly
- Cons: More complex setup, requires hosting
- Use case: Cloud deployments, multi-user scenarios

```python
# Server exposes HTTP endpoint
# Client connects to: http://localhost:8000/mcp
```

**3. WebSocket (Future)**
- Best for: Real-time bidirectional communication
- Status: Planned for future MCP versions

### Message Format and Structure

**Tool Call Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "search_database",
    "arguments": {
      "query": "SELECT * FROM users",
      "limit": 10
    }
  }
}
```

**Tool Call Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Found 10 users: Alice, Bob, ..."
      }
    ]
  }
}
```

**Error Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32602,
    "message": "Invalid params",
    "data": {
      "details": "query parameter is required"
    }
  }
}
```

---

## Core Components

### 1. Tools: Exposing Functions Claude Can Call

Tools are the primary way to give Claude new capabilities. Each tool is a function that Claude can invoke with specific arguments.

**Tool Definition Structure:**
```python
{
    "name": "tool_name",
    "description": "Clear description of what this tool does",
    "inputSchema": {
        "type": "object",
        "properties": {
            "param1": {
                "type": "string",
                "description": "Description of parameter"
            }
        },
        "required": ["param1"]
    }
}
```

**Key Principles:**
- **Clear naming**: Use descriptive, action-oriented names (e.g., `search_database`, not `db_query`)
- **Comprehensive descriptions**: Explain what the tool does, when to use it, and what it returns
- **Strong schemas**: Use JSON Schema to validate inputs and guide Claude
- **Error handling**: Return clear error messages when things go wrong

**Tool Handler Pattern:**
```python
async def handle_search_database(query: str, limit: int = 10):
    """
    Tool handler: Executes the actual logic

    Args:
        query: SQL query to execute
        limit: Maximum number of results

    Returns:
        List of results or error message
    """
    try:
        results = database.execute(query, limit=limit)
        return {
            "content": [
                {
                    "type": "text",
                    "text": f"Found {len(results)} results:\n{format_results(results)}"
                }
            ]
        }
    except Exception as e:
        return {
            "content": [
                {
                    "type": "text",
                    "text": f"Error: {str(e)}"
                }
            ],
            "isError": True
        }
```

### 2. Resources: Providing Data/Documentation Access

Resources allow Claude to access files, documentation, or structured data. Unlike tools (which perform actions), resources provide information.

**Resource Types:**
- **Static**: Fixed content (e.g., documentation files)
- **Dynamic**: Generated on-demand (e.g., database queries)
- **Templates**: Parameterized resources (e.g., user profiles)

**Resource Definition:**
```python
{
    "uri": "file:///docs/api-reference.md",
    "name": "API Reference",
    "description": "Complete API documentation",
    "mimeType": "text/markdown"
}
```

**Resource URI Patterns:**
```
file:///path/to/file.txt          # Local file
http://example.com/api/docs       # HTTP resource
custom://database/users/123       # Custom scheme
template://report/{user_id}       # Template resource
```

**Resource Handler Pattern:**
```python
async def handle_read_resource(uri: str):
    """
    Resource handler: Returns content based on URI

    Args:
        uri: Resource identifier

    Returns:
        Resource content with metadata
    """
    if uri.startswith("file:///"):
        path = uri[7:]  # Remove file:// prefix
        with open(path, 'r') as f:
            content = f.read()
        return {
            "contents": [
                {
                    "uri": uri,
                    "mimeType": "text/plain",
                    "text": content
                }
            ]
        }
```

### 3. Prompts: Reusable Prompt Templates

Prompts are pre-defined message templates that users can invoke. They help standardize common workflows and best practices.

**Prompt Definition:**
```python
{
    "name": "code_review",
    "description": "Comprehensive code review checklist",
    "arguments": [
        {
            "name": "language",
            "description": "Programming language",
            "required": True
        }
    ]
}
```

**Prompt Handler Pattern:**
```python
async def handle_get_prompt(name: str, arguments: dict):
    """
    Prompt handler: Returns prompt template

    Args:
        name: Prompt identifier
        arguments: Prompt parameters

    Returns:
        Formatted prompt messages
    """
    if name == "code_review":
        language = arguments.get("language", "Python")
        return {
            "messages": [
                {
                    "role": "user",
                    "content": {
                        "type": "text",
                        "text": f"""Please review this {language} code for:
1. Code quality and style
2. Security vulnerabilities
3. Performance issues
4. Best practices adherence
5. Documentation completeness"""
                    }
                }
            ]
        }
```

### 4. Authentication: OAuth, API Keys, Tokens

MCP supports multiple authentication methods to secure your servers.

**Authentication Methods:**

**1. No Authentication (Development Only)**
```python
# No auth configuration needed
# WARNING: Only use for local development!
```

**2. API Key Authentication**
```python
# Server side: Validate API key
async def authenticate_request(api_key: str):
    if api_key != os.getenv("MCP_API_KEY"):
        raise ValueError("Invalid API key")
```

**3. OAuth 2.0 Flow**
```python
# Server configuration
{
    "oauth": {
        "authorizationUrl": "https://provider.com/oauth/authorize",
        "tokenUrl": "https://provider.com/oauth/token",
        "clientId": "your-client-id",
        "scopes": ["read", "write"]
    }
}
```

**4. Bearer Token**
```python
# Client sends token in Authorization header
headers = {
    "Authorization": f"Bearer {token}"
}
```

**Security Best Practices:**
- Always use HTTPS for production servers
- Store credentials in environment variables or secrets management
- Implement rate limiting to prevent abuse
- Use short-lived tokens with refresh mechanism
- Log authentication attempts for audit trails

---

## Server Implementation

### Python SDK Usage

The official Python SDK (`mcp`) simplifies server creation:

**Installation:**
```bash
pip install mcp
# or
uv pip install mcp
```

**Basic Server Structure:**
```python
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent
import asyncio

# Initialize server
app = Server("my-mcp-server")

# Register tools
@app.list_tools()
async def list_tools():
    return [
        Tool(
            name="my_tool",
            description="Description of what this tool does",
            inputSchema={
                "type": "object",
                "properties": {
                    "param": {"type": "string"}
                },
                "required": ["param"]
            }
        )
    ]

# Implement tool handlers
@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "my_tool":
        param = arguments["param"]
        result = f"Processed: {param}"
        return [TextContent(type="text", text=result)]

# Run server
async def main():
    async with stdio_server() as (read_stream, write_stream):
        await app.run(read_stream, write_stream, app.create_initialization_options())

if __name__ == "__main__":
    asyncio.run(main())
```

### TypeScript/JavaScript SDK

**Installation:**
```bash
npm install @modelcontextprotocol/sdk
```

**Basic Server Structure:**
```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

// Initialize server
const server = new Server({
  name: "my-mcp-server",
  version: "1.0.0"
}, {
  capabilities: {
    tools: {}
  }
});

// Register tool handler
server.setRequestHandler("tools/list", async () => {
  return {
    tools: [
      {
        name: "my_tool",
        description: "Description of what this tool does",
        inputSchema: {
          type: "object",
          properties: {
            param: { type: "string" }
          },
          required: ["param"]
        }
      }
    ]
  };
});

// Implement tool execution
server.setRequestHandler("tools/call", async (request) => {
  const { name, arguments: args } = request.params;

  if (name === "my_tool") {
    return {
      content: [
        {
          type: "text",
          text: `Processed: ${args.param}`
        }
      ]
    };
  }
});

// Run server
const transport = new StdioServerTransport();
await server.connect(transport);
```

### Server Initialization

**Capability Declaration:**
```python
from mcp.server import Server

app = Server(
    name="my-server",
    version="1.0.0"
)

# Declare capabilities during initialization
initialization_options = {
    "capabilities": {
        "tools": {},           # Server provides tools
        "resources": {},       # Server provides resources
        "prompts": {},         # Server provides prompts
        "logging": {}          # Server supports logging
    },
    "serverInfo": {
        "name": "my-server",
        "version": "1.0.0"
    }
}
```

### Tool Registration and Handlers

**Registration Pattern:**
```python
# Method 1: Decorator pattern
@app.list_tools()
async def list_tools():
    return [
        Tool(
            name="calculator_add",
            description="Add two numbers",
            inputSchema={
                "type": "object",
                "properties": {
                    "a": {"type": "number", "description": "First number"},
                    "b": {"type": "number", "description": "Second number"}
                },
                "required": ["a", "b"]
            }
        )
    ]

# Method 2: Explicit registration
tools = [
    Tool(name="tool1", description="...", inputSchema={...}),
    Tool(name="tool2", description="...", inputSchema={...})
]

@app.list_tools()
async def list_tools():
    return tools
```

**Handler Pattern:**
```python
@app.call_tool()
async def call_tool(name: str, arguments: dict):
    """
    Route tool calls to appropriate handlers
    """
    if name == "calculator_add":
        return await handle_calculator_add(arguments)
    elif name == "calculator_multiply":
        return await handle_calculator_multiply(arguments)
    else:
        raise ValueError(f"Unknown tool: {name}")

async def handle_calculator_add(arguments: dict):
    """Individual tool handler"""
    a = arguments["a"]
    b = arguments["b"]
    result = a + b
    return [TextContent(type="text", text=f"{a} + {b} = {result}")]
```

### Resource Handlers

**Resource Listing:**
```python
from mcp.types import Resource

@app.list_resources()
async def list_resources():
    return [
        Resource(
            uri="file:///docs/readme.md",
            name="README",
            description="Project documentation",
            mimeType="text/markdown"
        ),
        Resource(
            uri="custom://data/users",
            name="User Data",
            description="Access to user database",
            mimeType="application/json"
        )
    ]
```

**Resource Reading:**
```python
from mcp.types import ResourceContents, TextResourceContents

@app.read_resource()
async def read_resource(uri: str):
    if uri == "file:///docs/readme.md":
        with open("docs/readme.md", "r") as f:
            content = f.read()
        return ResourceContents(
            contents=[
                TextResourceContents(
                    uri=uri,
                    mimeType="text/markdown",
                    text=content
                )
            ]
        )
    elif uri.startswith("custom://data/"):
        # Handle custom resource schemes
        data = await fetch_custom_data(uri)
        return ResourceContents(
            contents=[
                TextResourceContents(
                    uri=uri,
                    mimeType="application/json",
                    text=json.dumps(data)
                )
            ]
        )
```

### Error Handling and Validation

**Input Validation:**
```python
from jsonschema import validate, ValidationError

async def call_tool(name: str, arguments: dict):
    # Get tool schema
    tool_schema = get_tool_schema(name)

    # Validate arguments
    try:
        validate(instance=arguments, schema=tool_schema["inputSchema"])
    except ValidationError as e:
        return [TextContent(
            type="text",
            text=f"Validation error: {e.message}",
            isError=True
        )]

    # Execute tool
    return await execute_tool(name, arguments)
```

**Error Response Pattern:**
```python
async def handle_tool_call(name: str, arguments: dict):
    try:
        result = await execute_dangerous_operation(arguments)
        return [TextContent(type="text", text=result)]
    except ValueError as e:
        # User error (bad input)
        return [TextContent(
            type="text",
            text=f"Invalid input: {str(e)}",
            isError=True
        )]
    except ConnectionError as e:
        # External service error
        return [TextContent(
            type="text",
            text=f"Service unavailable: {str(e)}",
            isError=True
        )]
    except Exception as e:
        # Unexpected error
        logger.exception("Unexpected error in tool handler")
        return [TextContent(
            type="text",
            text=f"Internal error: {type(e).__name__}",
            isError=True
        )]
```

**Logging Integration:**
```python
import logging

logger = logging.getLogger(__name__)

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('mcp_server.log'),
        logging.StreamHandler()
    ]
)

async def call_tool(name: str, arguments: dict):
    logger.info(f"Tool called: {name} with args: {arguments}")
    try:
        result = await execute_tool(name, arguments)
        logger.info(f"Tool {name} succeeded")
        return result
    except Exception as e:
        logger.error(f"Tool {name} failed: {str(e)}", exc_info=True)
        raise
```

---

## Detailed Workflows

### Creating a Basic MCP Server

**Step 1: Project Setup**
```bash
# Create project directory
mkdir my-mcp-server
cd my-mcp-server

# Initialize Python project
uv init
uv add mcp

# Create server file
touch server.py
```

**Step 2: Minimal Server Implementation**
```python
# server.py
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent
import asyncio

app = Server("hello-world-server")

@app.list_tools()
async def list_tools():
    return [
        Tool(
            name="say_hello",
            description="Returns a friendly greeting",
            inputSchema={
                "type": "object",
                "properties": {
                    "name": {
                        "type": "string",
                        "description": "Name to greet"
                    }
                },
                "required": ["name"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "say_hello":
        greeting = f"Hello, {arguments['name']}!"
        return [TextContent(type="text", text=greeting)]

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await app.run(
            read_stream,
            write_stream,
            app.create_initialization_options()
        )

if __name__ == "__main__":
    asyncio.run(main())
```

**Step 3: Test the Server**
```bash
# Run server directly (for testing)
python server.py

# Or use MCP inspector
npx @modelcontextprotocol/inspector python server.py
```

**Step 4: Configure Claude Desktop**
```json
// ~/Library/Application Support/Claude/claude_desktop_config.json
{
  "mcpServers": {
    "hello-world": {
      "command": "python",
      "args": ["/absolute/path/to/server.py"]
    }
  }
}
```

### Adding Tools with Schemas

**JSON Schema Best Practices:**

```python
# ✅ Good: Detailed schema with descriptions
Tool(
    name="search_api",
    description="Search external API for information. Returns JSON results.",
    inputSchema={
        "type": "object",
        "properties": {
            "query": {
                "type": "string",
                "description": "Search query string (supports AND, OR operators)",
                "minLength": 1,
                "maxLength": 500
            },
            "filters": {
                "type": "object",
                "description": "Optional filters to narrow results",
                "properties": {
                    "category": {
                        "type": "string",
                        "enum": ["news", "blog", "documentation"],
                        "description": "Content category"
                    },
                    "date_range": {
                        "type": "object",
                        "properties": {
                            "start": {"type": "string", "format": "date"},
                            "end": {"type": "string", "format": "date"}
                        }
                    }
                }
            },
            "limit": {
                "type": "integer",
                "description": "Maximum number of results (1-100)",
                "minimum": 1,
                "maximum": 100,
                "default": 10
            }
        },
        "required": ["query"]
    }
)

# ❌ Bad: Minimal schema without guidance
Tool(
    name="search",
    description="Search",
    inputSchema={
        "type": "object",
        "properties": {
            "q": {"type": "string"}
        }
    }
)
```

**Complex Schema Example:**
```python
Tool(
    name="create_user",
    description="Create a new user account with validation",
    inputSchema={
        "type": "object",
        "properties": {
            "username": {
                "type": "string",
                "description": "Unique username (alphanumeric, 3-20 chars)",
                "pattern": "^[a-zA-Z0-9_]{3,20}$"
            },
            "email": {
                "type": "string",
                "description": "Valid email address",
                "format": "email"
            },
            "role": {
                "type": "string",
                "description": "User role determining permissions",
                "enum": ["admin", "editor", "viewer"],
                "default": "viewer"
            },
            "metadata": {
                "type": "object",
                "description": "Optional user metadata",
                "properties": {
                    "department": {"type": "string"},
                    "title": {"type": "string"}
                },
                "additionalProperties": False
            }
        },
        "required": ["username", "email"],
        "additionalProperties": False
    }
)
```

### Implementing Resources

**Static Resource Example:**
```python
from mcp.types import Resource, ResourceContents, TextResourceContents
import os
import glob

@app.list_resources()
async def list_resources():
    """List all markdown files in docs directory"""
    resources = []
    docs_path = "docs/"

    for filepath in glob.glob(f"{docs_path}**/*.md", recursive=True):
        uri = f"file://{os.path.abspath(filepath)}"
        name = os.path.basename(filepath)
        resources.append(Resource(
            uri=uri,
            name=name,
            description=f"Documentation: {name}",
            mimeType="text/markdown"
        ))

    return resources

@app.read_resource()
async def read_resource(uri: str):
    """Read file content"""
    if uri.startswith("file://"):
        filepath = uri[7:]  # Remove file:// prefix

        if not os.path.exists(filepath):
            raise FileNotFoundError(f"Resource not found: {uri}")

        with open(filepath, 'r', encoding='utf-8') as f:
            content = f.read()

        return ResourceContents(
            contents=[
                TextResourceContents(
                    uri=uri,
                    mimeType="text/markdown",
                    text=content
                )
            ]
        )
```

**Dynamic Resource Example:**
```python
from datetime import datetime

@app.list_resources()
async def list_resources():
    """List available dynamic resources"""
    return [
        Resource(
            uri="dynamic://system/status",
            name="System Status",
            description="Current system status and metrics",
            mimeType="application/json"
        ),
        Resource(
            uri="dynamic://logs/latest",
            name="Latest Logs",
            description="Most recent system logs",
            mimeType="text/plain"
        )
    ]

@app.read_resource()
async def read_resource(uri: str):
    """Generate resource content on-demand"""
    if uri == "dynamic://system/status":
        status = {
            "timestamp": datetime.now().isoformat(),
            "uptime": get_uptime(),
            "cpu_usage": get_cpu_usage(),
            "memory_usage": get_memory_usage(),
            "active_connections": get_active_connections()
        }
        return ResourceContents(
            contents=[
                TextResourceContents(
                    uri=uri,
                    mimeType="application/json",
                    text=json.dumps(status, indent=2)
                )
            ]
        )
    elif uri == "dynamic://logs/latest":
        logs = get_latest_logs(limit=100)
        return ResourceContents(
            contents=[
                TextResourceContents(
                    uri=uri,
                    mimeType="text/plain",
                    text="\n".join(logs)
                )
            ]
        )
```

**Template Resource Example:**
```python
@app.list_resources()
async def list_resources():
    """List template resources"""
    return [
        Resource(
            uri="template://user/{user_id}",
            name="User Profile Template",
            description="Access user profile by ID",
            mimeType="application/json"
        )
    ]

@app.read_resource()
async def read_resource(uri: str):
    """Handle parameterized resource URIs"""
    if uri.startswith("template://user/"):
        # Extract user_id from URI
        user_id = uri.split("/")[-1]

        # Fetch user data
        user_data = await database.get_user(user_id)

        if not user_data:
            raise ValueError(f"User not found: {user_id}")

        return ResourceContents(
            contents=[
                TextResourceContents(
                    uri=uri,
                    mimeType="application/json",
                    text=json.dumps(user_data, indent=2)
                )
            ]
        )
```

### Setting Up Authentication

**API Key Authentication:**
```python
import os
from functools import wraps

# Load API key from environment
VALID_API_KEY = os.getenv("MCP_API_KEY")

def require_auth(func):
    """Decorator to require API key authentication"""
    @wraps(func)
    async def wrapper(*args, **kwargs):
        # In production, API key would come from request headers
        # For stdio transport, use environment variable
        if not VALID_API_KEY:
            raise ValueError("Server not configured with API key")
        return await func(*args, **kwargs)
    return wrapper

@app.call_tool()
@require_auth
async def call_tool(name: str, arguments: dict):
    """Protected tool handler"""
    # Tool logic here
    pass
```

**OAuth 2.0 Example (HTTP Transport):**
```python
from aiohttp import web
import jwt

class OAuthMCPServer:
    def __init__(self):
        self.app = web.Application()
        self.app.router.add_post('/mcp', self.handle_mcp_request)

    async def verify_token(self, request):
        """Verify OAuth bearer token"""
        auth_header = request.headers.get('Authorization')
        if not auth_header or not auth_header.startswith('Bearer '):
            raise web.HTTPUnauthorized(text="Missing or invalid token")

        token = auth_header[7:]  # Remove 'Bearer ' prefix

        try:
            # Verify JWT token
            payload = jwt.decode(
                token,
                os.getenv('JWT_SECRET'),
                algorithms=['HS256']
            )
            return payload
        except jwt.InvalidTokenError:
            raise web.HTTPUnauthorized(text="Invalid token")

    async def handle_mcp_request(self, request):
        """Handle authenticated MCP request"""
        # Verify authentication
        user_info = await self.verify_token(request)

        # Parse MCP request
        mcp_request = await request.json()

        # Process request with user context
        response = await self.process_mcp_request(mcp_request, user_info)

        return web.json_response(response)
```

### Testing MCP Servers

**Unit Testing Tools:**
```python
import pytest
from mcp.client import ClientSession
from mcp.client.stdio import stdio_client

@pytest.mark.asyncio
async def test_calculator_add():
    """Test calculator add tool"""
    async with stdio_client(
        command="python",
        args=["server.py"]
    ) as (read, write):
        async with ClientSession(read, write) as session:
            # Initialize connection
            await session.initialize()

            # List available tools
            tools = await session.list_tools()
            assert any(t.name == "calculator_add" for t in tools.tools)

            # Call tool
            result = await session.call_tool(
                "calculator_add",
                {"a": 5, "b": 3}
            )

            assert result.content[0].text == "5 + 3 = 8"

@pytest.mark.asyncio
async def test_error_handling():
    """Test error handling"""
    async with stdio_client(
        command="python",
        args=["server.py"]
    ) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()

            # Test with invalid arguments
            result = await session.call_tool(
                "calculator_add",
                {"a": "not a number", "b": 3}
            )

            assert result.isError == True
            assert "Invalid input" in result.content[0].text
```

**Integration Testing with MCP Inspector:**
```bash
# Install MCP inspector
npm install -g @modelcontextprotocol/inspector

# Test server interactively
npx @modelcontextprotocol/inspector python server.py

# Test with specific tool call
npx @modelcontextprotocol/inspector python server.py \
  --tool calculator_add \
  --args '{"a": 5, "b": 3}'
```

### Deploying to Production

**Production Checklist:**
- [ ] Environment variables for secrets
- [ ] Comprehensive error handling
- [ ] Request/response logging
- [ ] Rate limiting
- [ ] Input validation
- [ ] Health check endpoint
- [ ] Monitoring and alerts
- [ ] Documentation

**Production Server Template:**
```python
import os
import logging
from datetime import datetime
from mcp.server import Server
from mcp.server.stdio import stdio_server

# Configure production logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler(f'mcp_server_{datetime.now():%Y%m%d}.log'),
        logging.StreamHandler()
    ]
)
logger = logging.getLogger(__name__)

# Load configuration
API_KEY = os.getenv("API_KEY")
DATABASE_URL = os.getenv("DATABASE_URL")
ENVIRONMENT = os.getenv("ENVIRONMENT", "production")

if not API_KEY or not DATABASE_URL:
    raise ValueError("Missing required environment variables")

# Initialize server
app = Server(
    name="production-server",
    version=os.getenv("APP_VERSION", "1.0.0")
)

# Request tracking
request_count = 0

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    global request_count
    request_count += 1
    request_id = f"req_{request_count}"

    logger.info(f"[{request_id}] Tool call: {name}")
    logger.debug(f"[{request_id}] Arguments: {arguments}")

    start_time = datetime.now()

    try:
        result = await execute_tool(name, arguments)
        duration = (datetime.now() - start_time).total_seconds()
        logger.info(f"[{request_id}] Success in {duration:.2f}s")
        return result
    except Exception as e:
        duration = (datetime.now() - start_time).total_seconds()
        logger.error(
            f"[{request_id}] Failed in {duration:.2f}s: {str(e)}",
            exc_info=True
        )
        raise

async def main():
    logger.info(f"Starting MCP server in {ENVIRONMENT} mode")
    async with stdio_server() as (read_stream, write_stream):
        await app.run(
            read_stream,
            write_stream,
            app.create_initialization_options()
        )

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

---

## Code Examples

### Example 1: Calculator Server (Simple Tools)
See `examples/simple-server.py`

### Example 2: REST API Wrapper
```python
import aiohttp
from mcp.server import Server
from mcp.types import Tool, TextContent

app = Server("github-api-server")

@app.list_tools()
async def list_tools():
    return [
        Tool(
            name="get_user",
            description="Fetch GitHub user profile",
            inputSchema={
                "type": "object",
                "properties": {
                    "username": {
                        "type": "string",
                        "description": "GitHub username"
                    }
                },
                "required": ["username"]
            }
        ),
        Tool(
            name="list_repos",
            description="List user's repositories",
            inputSchema={
                "type": "object",
                "properties": {
                    "username": {"type": "string"},
                    "limit": {"type": "integer", "default": 10}
                },
                "required": ["username"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    async with aiohttp.ClientSession() as session:
        if name == "get_user":
            username = arguments["username"]
            url = f"https://api.github.com/users/{username}"

            async with session.get(url) as response:
                if response.status == 200:
                    data = await response.json()
                    result = f"User: {data['name']}\nBio: {data['bio']}\nRepos: {data['public_repos']}"
                    return [TextContent(type="text", text=result)]
                else:
                    return [TextContent(
                        type="text",
                        text=f"Error: User not found",
                        isError=True
                    )]

        elif name == "list_repos":
            username = arguments["username"]
            limit = arguments.get("limit", 10)
            url = f"https://api.github.com/users/{username}/repos?per_page={limit}"

            async with session.get(url) as response:
                if response.status == 200:
                    repos = await response.json()
                    repo_list = "\n".join([
                        f"- {repo['name']}: {repo['description']}"
                        for repo in repos
                    ])
                    return [TextContent(type="text", text=repo_list)]
```

### Example 3: Database Query Server
```python
import sqlite3
from mcp.server import Server
from mcp.types import Tool, TextContent
import json

app = Server("database-server")

# Initialize database connection
conn = sqlite3.connect('app.db')
conn.row_factory = sqlite3.Row  # Enable column access by name

@app.list_tools()
async def list_tools():
    return [
        Tool(
            name="execute_query",
            description="Execute read-only SQL query",
            inputSchema={
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "SQL SELECT query"
                    },
                    "limit": {
                        "type": "integer",
                        "default": 100,
                        "maximum": 1000
                    }
                },
                "required": ["query"]
            }
        ),
        Tool(
            name="get_schema",
            description="Get database schema information",
            inputSchema={
                "type": "object",
                "properties": {
                    "table_name": {
                        "type": "string",
                        "description": "Optional: specific table name"
                    }
                }
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "execute_query":
        query = arguments["query"]
        limit = arguments.get("limit", 100)

        # Security: Only allow SELECT queries
        if not query.strip().upper().startswith("SELECT"):
            return [TextContent(
                type="text",
                text="Error: Only SELECT queries are allowed",
                isError=True
            )]

        try:
            cursor = conn.cursor()
            cursor.execute(f"{query} LIMIT {limit}")
            rows = cursor.fetchall()

            # Convert to list of dicts
            results = [dict(row) for row in rows]

            return [TextContent(
                type="text",
                text=json.dumps(results, indent=2)
            )]
        except sqlite3.Error as e:
            return [TextContent(
                type="text",
                text=f"Database error: {str(e)}",
                isError=True
            )]

    elif name == "get_schema":
        table_name = arguments.get("table_name")

        try:
            cursor = conn.cursor()
            if table_name:
                cursor.execute(f"PRAGMA table_info({table_name})")
                columns = cursor.fetchall()
                schema = {table_name: [dict(col) for col in columns]}
            else:
                cursor.execute("SELECT name FROM sqlite_master WHERE type='table'")
                tables = [row[0] for row in cursor.fetchall()]
                schema = {"tables": tables}

            return [TextContent(
                type="text",
                text=json.dumps(schema, indent=2)
            )]
        except sqlite3.Error as e:
            return [TextContent(
                type="text",
                text=f"Error: {str(e)}",
                isError=True
            )]
```

### Example 4: File System Access Server
See `examples/resource-server.py`

### Example 5: Web Scraping Server
```python
from bs4 import BeautifulSoup
import aiohttp
from mcp.server import Server
from mcp.types import Tool, TextContent

app = Server("web-scraper-server")

@app.list_tools()
async def list_tools():
    return [
        Tool(
            name="scrape_webpage",
            description="Extract text content from a webpage",
            inputSchema={
                "type": "object",
                "properties": {
                    "url": {
                        "type": "string",
                        "description": "URL to scrape",
                        "format": "uri"
                    },
                    "selector": {
                        "type": "string",
                        "description": "CSS selector (optional)",
                    }
                },
                "required": ["url"]
            }
        ),
        Tool(
            name="extract_links",
            description="Extract all links from a webpage",
            inputSchema={
                "type": "object",
                "properties": {
                    "url": {"type": "string", "format": "uri"}
                },
                "required": ["url"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    async with aiohttp.ClientSession() as session:
        url = arguments["url"]

        try:
            async with session.get(url, timeout=10) as response:
                if response.status != 200:
                    return [TextContent(
                        type="text",
                        text=f"Error: HTTP {response.status}",
                        isError=True
                    )]

                html = await response.text()
                soup = BeautifulSoup(html, 'html.parser')

                if name == "scrape_webpage":
                    selector = arguments.get("selector")

                    if selector:
                        elements = soup.select(selector)
                        text = "\n\n".join([el.get_text(strip=True) for el in elements])
                    else:
                        text = soup.get_text(separator="\n", strip=True)

                    return [TextContent(type="text", text=text)]

                elif name == "extract_links":
                    links = []
                    for a in soup.find_all('a', href=True):
                        href = a['href']
                        text = a.get_text(strip=True)
                        links.append(f"{text}: {href}")

                    return [TextContent(
                        type="text",
                        text="\n".join(links)
                    )]

        except asyncio.TimeoutError:
            return [TextContent(
                type="text",
                text="Error: Request timeout",
                isError=True
            )]
        except Exception as e:
            return [TextContent(
                type="text",
                text=f"Error: {str(e)}",
                isError=True
            )]
```

### Example 6: Authentication-Enabled Server
See `examples/advanced-server.py`

---

## Best Practices

### Tool Schema Design

**1. Use Descriptive Names**
```python
# ✅ Good
"search_customer_by_email"
"calculate_shipping_cost"
"generate_invoice_pdf"

# ❌ Bad
"search"
"calc"
"gen"
```

**2. Provide Comprehensive Descriptions**
```python
# ✅ Good
description="""
Search for customers by email address. Returns customer profile including:
- Contact information
- Order history
- Account status
Use this when you need to look up a specific customer by their email.
"""

# ❌ Bad
description="Search customers"
```

**3. Use Enums for Fixed Options**
```python
# ✅ Good
"status": {
    "type": "string",
    "enum": ["pending", "approved", "rejected"],
    "description": "Application status"
}

# ❌ Bad
"status": {
    "type": "string",
    "description": "Status (pending/approved/rejected)"
}
```

**4. Set Reasonable Constraints**
```python
# ✅ Good
"limit": {
    "type": "integer",
    "minimum": 1,
    "maximum": 1000,
    "default": 50,
    "description": "Number of results (1-1000)"
}

# ❌ Bad
"limit": {
    "type": "integer"
}
```

**5. Use Nested Objects for Complex Data**
```python
# ✅ Good
"filters": {
    "type": "object",
    "properties": {
        "date_range": {
            "type": "object",
            "properties": {
                "start": {"type": "string", "format": "date"},
                "end": {"type": "string", "format": "date"}
            }
        },
        "categories": {
            "type": "array",
            "items": {"type": "string"}
        }
    }
}
```

### Error Handling Strategies

**1. Categorize Errors**
```python
class MCPError(Exception):
    """Base MCP error"""
    pass

class ValidationError(MCPError):
    """Input validation failed"""
    pass

class AuthenticationError(MCPError):
    """Authentication failed"""
    pass

class ExternalServiceError(MCPError):
    """External service unavailable"""
    pass

async def call_tool(name: str, arguments: dict):
    try:
        return await execute_tool(name, arguments)
    except ValidationError as e:
        return [TextContent(
            type="text",
            text=f"Invalid input: {str(e)}",
            isError=True
        )]
    except AuthenticationError as e:
        return [TextContent(
            type="text",
            text="Authentication required",
            isError=True
        )]
    except ExternalServiceError as e:
        return [TextContent(
            type="text",
            text=f"Service temporarily unavailable: {str(e)}",
            isError=True
        )]
```

**2. Provide Actionable Error Messages**
```python
# ✅ Good
"Error: Email 'invalid-email' is not valid. Please provide a valid email address like user@example.com"

# ❌ Bad
"Invalid input"
```

**3. Log Errors for Debugging**
```python
async def call_tool(name: str, arguments: dict):
    try:
        return await execute_tool(name, arguments)
    except Exception as e:
        logger.error(
            f"Tool {name} failed",
            extra={
                "tool_name": name,
                "arguments": arguments,
                "error": str(e),
                "error_type": type(e).__name__
            },
            exc_info=True
        )
        raise
```

### Security Considerations

**1. Input Validation**
```python
import re
from urllib.parse import urlparse

def validate_url(url: str) -> bool:
    """Validate URL is safe"""
    parsed = urlparse(url)

    # Check scheme
    if parsed.scheme not in ['http', 'https']:
        raise ValidationError("Only HTTP/HTTPS URLs allowed")

    # Block local/private IPs
    if parsed.hostname in ['localhost', '127.0.0.1', '0.0.0.0']:
        raise ValidationError("Local URLs not allowed")

    return True

def sanitize_sql(query: str) -> str:
    """Basic SQL injection prevention"""
    dangerous_keywords = ['DROP', 'DELETE', 'INSERT', 'UPDATE', 'ALTER']
    query_upper = query.upper()

    for keyword in dangerous_keywords:
        if keyword in query_upper:
            raise ValidationError(f"Dangerous SQL keyword: {keyword}")

    return query
```

**2. Rate Limiting**
```python
from collections import defaultdict
from datetime import datetime, timedelta

class RateLimiter:
    def __init__(self, max_requests: int, time_window: timedelta):
        self.max_requests = max_requests
        self.time_window = time_window
        self.requests = defaultdict(list)

    def is_allowed(self, client_id: str) -> bool:
        now = datetime.now()
        cutoff = now - self.time_window

        # Remove old requests
        self.requests[client_id] = [
            req_time for req_time in self.requests[client_id]
            if req_time > cutoff
        ]

        # Check limit
        if len(self.requests[client_id]) >= self.max_requests:
            return False

        # Record request
        self.requests[client_id].append(now)
        return True

# Usage
rate_limiter = RateLimiter(max_requests=100, time_window=timedelta(minutes=1))

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    client_id = get_client_id()  # From auth context

    if not rate_limiter.is_allowed(client_id):
        return [TextContent(
            type="text",
            text="Rate limit exceeded. Please try again later.",
            isError=True
        )]

    return await execute_tool(name, arguments)
```

**3. Secrets Management**
```python
import os
from pathlib import Path

# ✅ Good: Environment variables
API_KEY = os.getenv("API_KEY")
DATABASE_URL = os.getenv("DATABASE_URL")

# ✅ Good: Secrets file (gitignored)
def load_secrets():
    secrets_path = Path.home() / ".mcp" / "secrets.json"
    with open(secrets_path) as f:
        return json.load(f)

# ❌ Bad: Hardcoded secrets
API_KEY = "sk-1234567890abcdef"  # NEVER DO THIS!
```

### Performance Optimization

**1. Connection Pooling**
```python
import aiohttp

# ✅ Good: Reuse session
class APIClient:
    def __init__(self):
        self.session = None

    async def __aenter__(self):
        self.session = aiohttp.ClientSession()
        return self

    async def __aexit__(self, *args):
        await self.session.close()

    async def fetch(self, url: str):
        async with self.session.get(url) as response:
            return await response.json()

# Usage
client = APIClient()

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    async with client:
        data = await client.fetch(arguments["url"])
        return [TextContent(type="text", text=json.dumps(data))]
```

**2. Caching**
```python
from functools import lru_cache
from datetime import datetime, timedelta
import asyncio

class AsyncLRUCache:
    def __init__(self, maxsize: int, ttl: timedelta):
        self.cache = {}
        self.maxsize = maxsize
        self.ttl = ttl

    async def get(self, key: str):
        if key in self.cache:
            value, timestamp = self.cache[key]
            if datetime.now() - timestamp < self.ttl:
                return value
            del self.cache[key]
        return None

    async def set(self, key: str, value):
        if len(self.cache) >= self.maxsize:
            # Remove oldest entry
            oldest = min(self.cache.items(), key=lambda x: x[1][1])
            del self.cache[oldest[0]]

        self.cache[key] = (value, datetime.now())

# Usage
cache = AsyncLRUCache(maxsize=1000, ttl=timedelta(minutes=5))

async def fetch_user_data(user_id: str):
    # Check cache
    cached = await cache.get(f"user:{user_id}")
    if cached:
        return cached

    # Fetch from database
    data = await database.get_user(user_id)

    # Cache result
    await cache.set(f"user:{user_id}", data)

    return data
```

**3. Async Best Practices**
```python
# ✅ Good: Parallel execution
async def call_tool(name: str, arguments: dict):
    if name == "fetch_multiple_users":
        user_ids = arguments["user_ids"]

        # Fetch in parallel
        tasks = [fetch_user_data(uid) for uid in user_ids]
        results = await asyncio.gather(*tasks)

        return [TextContent(type="text", text=json.dumps(results))]

# ❌ Bad: Sequential execution
async def call_tool(name: str, arguments: dict):
    if name == "fetch_multiple_users":
        results = []
        for user_id in arguments["user_ids"]:
            result = await fetch_user_data(user_id)  # Slow!
            results.append(result)
        return [TextContent(type="text", text=json.dumps(results))]
```

### Documentation Standards

**1. Tool Documentation Template**
```python
Tool(
    name="tool_name",
    description="""
    [One sentence summary of what this tool does]

    [Detailed explanation of functionality]

    Use this tool when:
    - [Specific use case 1]
    - [Specific use case 2]

    Returns: [Description of return value format]

    Example usage:
    - Input: {"param": "value"}
    - Output: "Expected result format"

    Note: [Any important caveats or limitations]
    """,
    inputSchema={...}
)
```

**2. Server README Template**
```markdown
# MCP Server Name

## Overview
Brief description of what this server does and why it's useful.

## Installation
\`\`\`bash
pip install -r requirements.txt
\`\`\`

## Configuration
Required environment variables:
- `API_KEY`: Your API key from [provider]
- `DATABASE_URL`: Connection string

## Available Tools

### tool_name
Description of the tool and what it does.

**Parameters:**
- `param1` (string, required): Description
- `param2` (integer, optional): Description

**Example:**
\`\`\`json
{
  "param1": "value",
  "param2": 42
}
\`\`\`

## Available Resources

### resource_uri
Description of the resource.

## Testing
\`\`\`bash
pytest tests/
\`\`\`

## Deployment
Instructions for deploying to production.

## License
MIT
```

### Testing Strategies

**1. Unit Tests for Tool Handlers**
```python
import pytest
from server import handle_calculator_add

@pytest.mark.asyncio
async def test_calculator_add():
    result = await handle_calculator_add({"a": 5, "b": 3})
    assert result[0].text == "5 + 3 = 8"

@pytest.mark.asyncio
async def test_calculator_add_negative():
    result = await handle_calculator_add({"a": -5, "b": 3})
    assert result[0].text == "-5 + 3 = -2"

@pytest.mark.asyncio
async def test_calculator_add_floats():
    result = await handle_calculator_add({"a": 1.5, "b": 2.5})
    assert result[0].text == "1.5 + 2.5 = 4.0"
```

**2. Integration Tests**
```python
@pytest.mark.asyncio
async def test_full_tool_workflow():
    """Test complete MCP workflow"""
    async with stdio_client(
        command="python",
        args=["server.py"]
    ) as (read, write):
        async with ClientSession(read, write) as session:
            # Initialize
            await session.initialize()

            # List tools
            tools_response = await session.list_tools()
            assert len(tools_response.tools) > 0

            # Call tool
            result = await session.call_tool(
                "calculator_add",
                {"a": 5, "b": 3}
            )
            assert not result.isError
            assert "8" in result.content[0].text
```

**3. Mock External Dependencies**
```python
from unittest.mock import AsyncMock, patch

@pytest.mark.asyncio
async def test_api_call_with_mock():
    """Test API call with mocked HTTP request"""
    with patch('aiohttp.ClientSession.get') as mock_get:
        # Setup mock
        mock_response = AsyncMock()
        mock_response.status = 200
        mock_response.json = AsyncMock(return_value={"key": "value"})
        mock_get.return_value.__aenter__.return_value = mock_response

        # Test
        result = await handle_api_call({"url": "https://api.example.com"})
        assert "value" in result[0].text
```

---

## Integration Guides

### Claude Desktop Configuration

**Location:**
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`
- Linux: `~/.config/Claude/claude_desktop_config.json`

**Basic Configuration:**
```json
{
  "mcpServers": {
    "my-server": {
      "command": "python",
      "args": ["/absolute/path/to/server.py"]
    }
  }
}
```

**With Environment Variables:**
```json
{
  "mcpServers": {
    "my-server": {
      "command": "python",
      "args": ["/absolute/path/to/server.py"],
      "env": {
        "API_KEY": "your-api-key",
        "DATABASE_URL": "postgresql://localhost/mydb"
      }
    }
  }
}
```

**Using UV (Python Package Manager):**
```json
{
  "mcpServers": {
    "my-server": {
      "command": "uv",
      "args": [
        "--directory",
        "/absolute/path/to/project",
        "run",
        "server.py"
      ]
    }
  }
}
```

**Multiple Servers:**
```json
{
  "mcpServers": {
    "calculator": {
      "command": "python",
      "args": ["/path/to/calculator-server.py"]
    },
    "github": {
      "command": "node",
      "args": ["/path/to/github-server.js"]
    },
    "database": {
      "command": "python",
      "args": ["/path/to/database-server.py"],
      "env": {
        "DATABASE_URL": "postgresql://localhost/mydb"
      }
    }
  }
}
```

### Claude Code Integration

Claude Code automatically discovers MCP servers configured in Claude Desktop.

**Additional Configuration:**
```json
{
  "mcpServers": {
    "code-helper": {
      "command": "python",
      "args": ["/path/to/code-helper-server.py"],
      "env": {
        "WORKSPACE_PATH": "/path/to/workspace"
      }
    }
  }
}
```

### API Integration

**HTTP Transport Example:**
```python
from mcp.server import Server
from aiohttp import web
import json

app = Server("http-server")

# Configure tools...

async def handle_mcp_request(request):
    """Handle HTTP MCP requests"""
    try:
        mcp_request = await request.json()

        # Route to appropriate handler
        method = mcp_request.get("method")
        params = mcp_request.get("params", {})

        if method == "tools/list":
            result = await app.list_tools_handler()
        elif method == "tools/call":
            result = await app.call_tool_handler(
                params["name"],
                params.get("arguments", {})
            )

        return web.json_response({
            "jsonrpc": "2.0",
            "id": mcp_request.get("id"),
            "result": result
        })
    except Exception as e:
        return web.json_response({
            "jsonrpc": "2.0",
            "id": mcp_request.get("id"),
            "error": {
                "code": -32603,
                "message": str(e)
            }
        }, status=500)

# Setup HTTP server
http_app = web.Application()
http_app.router.add_post('/mcp', handle_mcp_request)

if __name__ == "__main__":
    web.run_app(http_app, host='0.0.0.0', port=8000)
```

**Client Configuration:**
```json
{
  "mcpServers": {
    "remote-server": {
      "url": "http://localhost:8000/mcp",
      "transport": "http"
    }
  }
}
```

### Common Integration Patterns

**Pattern 1: Chaining Multiple Servers**
```python
# Server A: Data fetcher
# Provides: fetch_data tool

# Server B: Data processor
# Provides: process_data tool
# Uses: fetch_data from Server A (via Claude)

# Claude workflow:
# 1. Call fetch_data on Server A
# 2. Pass result to process_data on Server B
# 3. Return final result
```

**Pattern 2: Resource + Tool Combination**
```python
# Provide documentation as resources
@app.list_resources()
async def list_resources():
    return [
        Resource(
            uri="docs://api-reference",
            name="API Reference",
            description="Complete API documentation"
        )
    ]

# Provide tools that reference the documentation
@app.list_tools()
async def list_tools():
    return [
        Tool(
            name="call_api",
            description="Call the API (see docs://api-reference for details)",
            inputSchema={...}
        )
    ]
```

**Pattern 3: Prompt-Guided Workflows**
```python
# Provide prompt templates for common tasks
@app.list_prompts()
async def list_prompts():
    return [
        Prompt(
            name="analyze_data",
            description="Comprehensive data analysis workflow",
            arguments=[
                {"name": "dataset", "required": True}
            ]
        )
    ]

@app.get_prompt()
async def get_prompt(name: str, arguments: dict):
    if name == "analyze_data":
        dataset = arguments["dataset"]
        return {
            "messages": [
                {
                    "role": "user",
                    "content": f"""
                    Please analyze the dataset '{dataset}' using these steps:
                    1. Call fetch_data tool to load the dataset
                    2. Call describe_stats tool to get summary statistics
                    3. Call detect_anomalies tool to find outliers
                    4. Provide comprehensive analysis report
                    """
                }
            ]
        }
```

---

## Common Pitfalls

### Schema Validation Errors

**Pitfall 1: Missing Required Properties**
```python
# ❌ Bad: Schema says required, but tool doesn't validate
Tool(
    name="create_user",
    inputSchema={
        "type": "object",
        "properties": {
            "username": {"type": "string"},
            "email": {"type": "string"}
        },
        "required": ["username", "email"]  # Required!
    }
)

async def handle_create_user(arguments: dict):
    # No validation - will crash if missing!
    username = arguments["username"]
    email = arguments["email"]

# ✅ Good: Validate inputs
async def handle_create_user(arguments: dict):
    if "username" not in arguments or "email" not in arguments:
        return [TextContent(
            type="text",
            text="Error: username and email are required",
            isError=True
        )]

    username = arguments["username"]
    email = arguments["email"]
```

**Pitfall 2: Type Mismatches**
```python
# ❌ Bad: Schema says number, but you treat as string
Tool(
    name="multiply",
    inputSchema={
        "properties": {
            "value": {"type": "number"}
        }
    }
)

async def handle_multiply(arguments: dict):
    # If value is number, this will crash!
    return arguments["value"].upper()

# ✅ Good: Respect schema types
async def handle_multiply(arguments: dict):
    value = float(arguments["value"])  # Ensure it's a number
    return value * 2
```

### Authentication Issues

**Pitfall 1: Insecure API Key Storage**
```python
# ❌ Bad: Hardcoded API key
API_KEY = "sk-1234567890abcdef"

# ❌ Bad: API key in version control
# config.json committed to git
{"api_key": "sk-1234567890abcdef"}

# ✅ Good: Environment variables
API_KEY = os.getenv("API_KEY")
if not API_KEY:
    raise ValueError("API_KEY environment variable required")
```

**Pitfall 2: Missing Authentication Checks**
```python
# ❌ Bad: No authentication
@app.call_tool()
async def call_tool(name: str, arguments: dict):
    # Anyone can call this!
    return await execute_sensitive_operation(arguments)

# ✅ Good: Require authentication
@app.call_tool()
async def call_tool(name: str, arguments: dict):
    # Validate API key from environment
    request_key = get_request_api_key()  # From request context
    if request_key != os.getenv("VALID_API_KEY"):
        return [TextContent(
            type="text",
            text="Authentication required",
            isError=True
        )]

    return await execute_sensitive_operation(arguments)
```

### Transport Configuration

**Pitfall 1: Path Issues with STDIO**
```python
# ❌ Bad: Relative path in Claude Desktop config
{
  "command": "python",
  "args": ["server.py"]  # Won't work!
}

# ✅ Good: Absolute path
{
  "command": "python",
  "args": ["/Users/username/projects/mcp-server/server.py"]
}

# ✅ Better: Use UV with directory
{
  "command": "uv",
  "args": ["--directory", "/Users/username/projects/mcp-server", "run", "server.py"]
}
```

**Pitfall 2: Port Conflicts with HTTP**
```python
# ❌ Bad: Hardcoded port
web.run_app(app, port=8000)  # May already be in use!

# ✅ Good: Configurable port
PORT = int(os.getenv("MCP_PORT", "8000"))
web.run_app(app, port=PORT)
```

### Error Propagation

**Pitfall 1: Swallowing Errors**
```python
# ❌ Bad: Silent failure
async def call_tool(name: str, arguments: dict):
    try:
        return await execute_tool(name, arguments)
    except Exception:
        return [TextContent(type="text", text="Something went wrong")]

# ✅ Good: Descriptive error messages
async def call_tool(name: str, arguments: dict):
    try:
        return await execute_tool(name, arguments)
    except ValueError as e:
        return [TextContent(
            type="text",
            text=f"Invalid input: {str(e)}",
            isError=True
        )]
    except ConnectionError as e:
        return [TextContent(
            type="text",
            text=f"Service unavailable: {str(e)}",
            isError=True
        )]
    except Exception as e:
        logger.exception("Unexpected error")
        return [TextContent(
            type="text",
            text=f"Unexpected error: {type(e).__name__}",
            isError=True
        )]
```

**Pitfall 2: Not Marking Errors**
```python
# ❌ Bad: Error looks like success
return [TextContent(type="text", text="Error: Something failed")]

# ✅ Good: Use isError flag
return [TextContent(
    type="text",
    text="Error: Something failed",
    isError=True
)]
```

### Resource Caching

**Pitfall 1: No Cache Invalidation**
```python
# ❌ Bad: Cache never expires
cache = {}

@app.read_resource()
async def read_resource(uri: str):
    if uri in cache:
        return cache[uri]  # May be stale!

    data = await fetch_resource(uri)
    cache[uri] = data
    return data

# ✅ Good: TTL-based caching
from datetime import datetime, timedelta

cache = {}
cache_ttl = timedelta(minutes=5)

@app.read_resource()
async def read_resource(uri: str):
    if uri in cache:
        data, timestamp = cache[uri]
        if datetime.now() - timestamp < cache_ttl:
            return data
        del cache[uri]

    data = await fetch_resource(uri)
    cache[uri] = (data, datetime.now())
    return data
```

**Pitfall 2: Unbounded Cache Growth**
```python
# ❌ Bad: Cache grows forever
cache = {}

@app.read_resource()
async def read_resource(uri: str):
    if uri not in cache:
        cache[uri] = await fetch_resource(uri)  # Memory leak!
    return cache[uri]

# ✅ Good: LRU cache with size limit
from functools import lru_cache

@lru_cache(maxsize=1000)
def get_cached_resource(uri: str):
    return fetch_resource_sync(uri)
```

---

## Testing and Debugging

### Using MCP Inspector

**Installation:**
```bash
npm install -g @modelcontextprotocol/inspector
```

**Basic Usage:**
```bash
# Start inspector with your server
npx @modelcontextprotocol/inspector python server.py

# Inspector opens in browser at http://localhost:5173
```

**Features:**
- Interactive tool testing
- Real-time message inspection
- Schema validation
- Performance monitoring

### Logging Best Practices

```python
import logging
import sys

# Configure logging
logging.basicConfig(
    level=logging.DEBUG,  # Use INFO in production
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('mcp_server.log'),
        logging.StreamHandler(sys.stderr)  # STDERR for stdio transport
    ]
)

logger = logging.getLogger(__name__)

# Log at appropriate levels
logger.debug("Detailed debugging information")
logger.info("General information about operations")
logger.warning("Something unexpected but handled")
logger.error("Error that needs attention")
logger.exception("Error with full traceback")
```

### Debugging Common Issues

**Issue 1: Server Not Starting**
```python
# Add startup logging
async def main():
    logger.info("Starting MCP server...")
    logger.info(f"Python version: {sys.version}")
    logger.info(f"Working directory: {os.getcwd()}")

    try:
        async with stdio_server() as (read_stream, write_stream):
            logger.info("STDIO transport initialized")
            await app.run(read_stream, write_stream, app.create_initialization_options())
    except Exception as e:
        logger.exception("Server startup failed")
        raise
```

**Issue 2: Tool Not Appearing**
```python
# Debug tool registration
@app.list_tools()
async def list_tools():
    tools = [...]
    logger.info(f"Returning {len(tools)} tools: {[t.name for t in tools]}")
    return tools
```

**Issue 3: Arguments Not Received**
```python
# Log received arguments
@app.call_tool()
async def call_tool(name: str, arguments: dict):
    logger.info(f"Tool called: {name}")
    logger.debug(f"Arguments received: {json.dumps(arguments, indent=2)}")
    logger.debug(f"Argument types: {[(k, type(v)) for k, v in arguments.items()]}")

    result = await execute_tool(name, arguments)
    logger.debug(f"Result: {result}")
    return result
```

---

## Production Deployment

### Environment Configuration

**Production Checklist:**
- [ ] Use environment variables for all secrets
- [ ] Configure logging to files with rotation
- [ ] Set appropriate log levels (INFO or WARNING)
- [ ] Enable monitoring and health checks
- [ ] Configure rate limiting
- [ ] Set up error alerting
- [ ] Document deployment process

**Example Production Configuration:**
```python
import os
import logging
from logging.handlers import RotatingFileHandler

# Environment
ENVIRONMENT = os.getenv("ENVIRONMENT", "production")
DEBUG = ENVIRONMENT == "development"

# Logging
log_level = logging.DEBUG if DEBUG else logging.INFO
log_handler = RotatingFileHandler(
    "mcp_server.log",
    maxBytes=10*1024*1024,  # 10MB
    backupCount=5
)
log_handler.setFormatter(logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
))

logger = logging.getLogger(__name__)
logger.setLevel(log_level)
logger.addHandler(log_handler)

# Secrets (from environment)
API_KEY = os.getenv("API_KEY")
DATABASE_URL = os.getenv("DATABASE_URL")

if not API_KEY or not DATABASE_URL:
    raise ValueError("Missing required environment variables")

# Rate limiting
MAX_REQUESTS_PER_MINUTE = int(os.getenv("RATE_LIMIT", "100"))
```

### Monitoring

**Health Check Implementation:**
```python
from datetime import datetime

health_stats = {
    "start_time": datetime.now(),
    "request_count": 0,
    "error_count": 0
}

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    health_stats["request_count"] += 1

    if name == "health_check":
        uptime = (datetime.now() - health_stats["start_time"]).total_seconds()
        return [TextContent(
            type="text",
            text=json.dumps({
                "status": "healthy",
                "uptime_seconds": uptime,
                "total_requests": health_stats["request_count"],
                "error_count": health_stats["error_count"],
                "error_rate": health_stats["error_count"] / health_stats["request_count"] if health_stats["request_count"] > 0 else 0
            })
        )]

    try:
        return await execute_tool(name, arguments)
    except Exception as e:
        health_stats["error_count"] += 1
        raise
```

### Scaling Considerations

**Horizontal Scaling:**
```python
# Use stateless design
# - No in-memory caching (use Redis instead)
# - No file-based storage (use object storage)
# - Connection pooling for databases

import aioredis

class ScalableMCPServer:
    def __init__(self):
        self.redis = None

    async def initialize(self):
        """Initialize shared resources"""
        self.redis = await aioredis.create_redis_pool(
            os.getenv("REDIS_URL", "redis://localhost")
        )

    async def get_cached(self, key: str):
        """Get from shared cache"""
        value = await self.redis.get(key)
        return json.loads(value) if value else None

    async def set_cached(self, key: str, value, ttl: int = 300):
        """Set in shared cache with TTL"""
        await self.redis.setex(
            key,
            ttl,
            json.dumps(value)
        )
```

### Security Hardening

**Production Security:**
```python
import secrets
import hashlib
from functools import wraps

# Generate secure tokens
def generate_api_key():
    return secrets.token_urlsafe(32)

# Hash sensitive data
def hash_password(password: str) -> str:
    salt = secrets.token_bytes(32)
    hashed = hashlib.pbkdf2_hmac('sha256', password.encode(), salt, 100000)
    return salt.hex() + hashed.hex()

# Verify hashed data
def verify_password(password: str, hashed: str) -> bool:
    salt = bytes.fromhex(hashed[:64])
    stored_hash = bytes.fromhex(hashed[64:])
    new_hash = hashlib.pbkdf2_hmac('sha256', password.encode(), salt, 100000)
    return secrets.compare_digest(stored_hash, new_hash)

# Require HTTPS in production
def require_https(func):
    @wraps(func)
    async def wrapper(request, *args, **kwargs):
        if request.scheme != 'https' and os.getenv('ENVIRONMENT') == 'production':
            raise web.HTTPBadRequest(text="HTTPS required")
        return await func(request, *args, **kwargs)
    return wrapper
```

---

## Additional Resources

**Official Documentation:**
- MCP Specification: https://modelcontextprotocol.io/
- Python SDK: https://github.com/modelcontextprotocol/python-sdk
- TypeScript SDK: https://github.com/modelcontextprotocol/typescript-sdk

**Example Servers:**
- See `examples/` directory in this skill
- Official examples: https://github.com/modelcontextprotocol/servers

**Community:**
- GitHub Discussions: https://github.com/modelcontextprotocol/discussions
- Discord: [Link to community Discord]

**Tools:**
- MCP Inspector: https://github.com/modelcontextprotocol/inspector
- Claude Desktop: https://claude.ai/download

---

## Quick Reference

### Server Template (Python)
```python
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent
import asyncio

app = Server("my-server")

@app.list_tools()
async def list_tools():
    return [Tool(name="my_tool", description="...", inputSchema={...})]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "my_tool":
        return [TextContent(type="text", text="Result")]

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await app.run(read_stream, write_stream, app.create_initialization_options())

if __name__ == "__main__":
    asyncio.run(main())
```

### Claude Desktop Config
```json
{
  "mcpServers": {
    "my-server": {
      "command": "python",
      "args": ["/absolute/path/to/server.py"],
      "env": {"API_KEY": "your-key"}
    }
  }
}
```

### Common Patterns

**Error Handling:**
```python
return [TextContent(type="text", text="Error message", isError=True)]
```

**Async Operations:**
```python
results = await asyncio.gather(*tasks)
```

**Input Validation:**
```python
if "required_param" not in arguments:
    return [TextContent(type="text", text="Missing required parameter", isError=True)]
```

---

**End of MCP Builder Skill Guide**

For complete working examples, see the `examples/` directory.
