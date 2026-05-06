# MCP Integration Guide

## Table of Contents
- [MCP Integration Guide](#mcp-integration-guide)
  - [Table of Contents](#table-of-contents)
  - [Preparation](#preparation)
    - [mcp-proxy](#mcp-proxy)
  - [ChatWise](#chatwise)
  - [Cherry Studio](#cherry-studio)
  - [Claude Desktop](#claude-desktop)
  - [Monica Code](#monica-code)


## Preparation

Run `chatlog`, finish data decryption, and start the HTTP service.

### mcp-proxy
If you encounter a client that does not support `SSE`, you can use `mcp-proxy` to convert `stdio` requests into `SSE`.

Project: https://github.com/sparfenyuk/mcp-proxy

Installation:
```shell
# Install via uv tool; see project docs for other installation methods
uv tool install mcp-proxy

# Find the mcp-proxy binary path for later use
which mcp-proxy
/Users/sarv/.local/bin/mcp-proxy
```

## ChatWise

- Website: https://chatwise.app/
- Transport: MCP SSE
- Note: ChatWise MCP features require a Pro account

1. Create a new `SSE Request` tool under Settings -> Tools

![chatwise-1](https://github.com/user-attachments/assets/87e40f39-9fbc-4ff1-954a-d95548cde4c2)

2. Set the tool URL to `http://127.0.0.1:5030/sse`, enable "Auto Execute Tool", and click "Inspect Tool" to verify the connection with chatlog.

![chatwise-2](https://github.com/user-attachments/assets/8f98ef18-8e6c-40e6-ae78-8cd13e411c36)

3. Return to the main page, choose an MCP-capable model, and enable the `chatlog` tool option.

![chatwise-3](https://github.com/user-attachments/assets/ea2aa178-5439-492b-a92f-4f4fc08828e7)

4. Test the integration.

![chatwise-4](https://github.com/user-attachments/assets/8f82cb53-8372-40ee-a299-c02d3399403a)

## Cherry Studio

- Website: https://cherry-ai.com/
- Transport: MCP SSE

1. In Settings -> MCP Servers click Add Server, name it `chatlog`, select "Server-Sent Events (sse)" as the type, set the URL to `http://127.0.0.1:5030/sse`, and click Save. (Do not click the enable switch before saving.)

![cherry-1](https://github.com/user-attachments/assets/93fc8b0a-9d95-499e-ab6c-e22b0c96fd6a)

2. Choose an MCP-capable model and enable the `chatlog` tool option.

![cherry-2](https://github.com/user-attachments/assets/4e5bf752-2eab-4e7c-b73b-1b759d4a5f29)

3. Test the integration.

![cherry-3](https://github.com/user-attachments/assets/c58a019f-fd5f-4fa3-830a-e81a60f2aa6f)

## Claude Desktop

- Website: https://claude.ai/download
- Transport: mcp-proxy
- Reference: https://modelcontextprotocol.io/quickstart/user#2-add-the-filesystem-mcp-server

1. Install `mcp-proxy` (see the mcp-proxy section above).

2. In Claude Desktop, go to Settings -> Developer and click "Edit Config". This will create and open `claude_desktop_config.json` for editing.

3. Edit `claude_desktop_config.json` to add a server named `chatlog`. Set `command` to the path of `mcp-proxy` and `args` to `http://127.0.0.1:5030/sse`, for example:

```json
{
  "mcpServers": {
    "chatlog": {
      "command": "/Users/sarv/.local/bin/mcp-proxy",
      "args": [
        "http://localhost:5030/sse"
      ]
    }
  },
  "globalShortcut": ""
}
```

4. Save `claude_desktop_config.json` and restart Claude Desktop. You should see `chatlog` added.

![claude-1](https://github.com/user-attachments/assets/f4e872cc-e6c1-4e24-97da-266466949cdf)

5. Test the integration.

![claude-2](https://github.com/user-attachments/assets/832bb4d2-3639-4cbc-8b17-f4b812ea3637)


## Monica Code

- Website: https://monica.im/en/code
- Transport: mcp-proxy
- Reference: https://github.com/Monica-IM/Monica-Code/blob/main/Reference/config.md#modelcontextprotocolserver

1. Install `mcp-proxy` (see mcp-proxy section above).

2. Find the Monica Code extension folder in the VSCode extensions directory (`~/.vscode/extensions`), and edit `config_schema.json`. In `experimental.modelContextProtocolServer`, set `transport` to `stdio`, `command` to the path of `mcp-proxy`, and `args` to `http://localhost:5030/sse`, for example:

```json
{
  "experimental": {
    "type": "object",
    "title": "Experimental",
    "description": "Experimental properties are subject to change.",
    "properties": {
      "modelContextProtocolServer": {
        "type": "object",
        "properties": {
          "transport": {
            "type": "stdio",
            "command": "/Users/sarv/.local/bin/mcp-proxy",
            "args": [
              "http://localhost:5030/sse"
            ]
          }
        },
        "required": [
          "transport"
        ]
      }
    }
  }
}
```

3. Restart VSCode; you should see `chatlog` added.

![monica-1](https://github.com/user-attachments/assets/8d0a96f2-ed05-48aa-a99a-06648ae1c500)

4. Test the integration.

![monica-2](https://github.com/user-attachments/assets/054e0a30-428a-48a6-9f31-d2596fb8f743)

