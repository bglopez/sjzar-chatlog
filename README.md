<div align="center">

![chatlog](https://github.com/user-attachments/assets/e085d3a2-e009-4463-b2fd-8bd7df2b50c3)

_A chat log tool that helps you easily use and analyze your own chat data_

[![ImgMCP](https://cdn.imgmcp.com/imgmcp-logo-small.png)](https://imgmcp.com)

[![Go Report Card](https://goreportcard.com/badge/github.com/sjzar/chatlog)](https://goreportcard.com/report/github.com/sjzar/chatlog)
[![GoDoc](https://godoc.org/github.com/sjzar/chatlog?status.svg)](https://godoc.org/github.com/sjzar/chatlog)
[![GitHub release](https://img.shields.io/github/release/sjzar/chatlog.svg)](https://github.com/sjzar/chatlog/releases)
[![GitHub license](https://img.shields.io/github/license/sjzar/chatlog.svg)](https://github.com/sjzar/chatlog/blob/main/LICENSE)

</div>

## Features

- Extract chat data from local database files
- Supports Windows and macOS, compatible with WeChat 3.x/4.x versions
- Fetch data and image keys (Windows < 4.0.3.36 / macOS < 4.0.3.80)
- Supports decryption of images, audio, and other multimedia content, including wxgf format parsing
- Automatic database decryption and webhook callback for new messages
- Provides a Terminal UI, as well as command line tools and Docker images
- Offers an HTTP API service for easy access to chat logs, contacts, groups, and recent conversations
- Supports MCP Streamable HTTP protocol for seamless AI assistant integration
- Multi-account management, allowing you to switch between different accounts

## Quick Start

### Basic Steps

1. **Install Chatlog:** [Download prebuilt binaries](#download-prebuilt-binaries) or [install with Go](#install-from-source)
2. **Run the app:** Launch by running `chatlog` for the Terminal UI
3. **Decrypt data:** Select the `Decrypt Data` menu item
4. **Start HTTP service:** Select the `Start HTTP Service` menu item
5. **Access your data:** Use the [HTTP API](#http-api) or [MCP integration](#mcp-integration) to access your chat logs

> 💡 **Tip:** If the chat history on your desktop WeChat is incomplete, [migrate your data from mobile](#migrating-chat-history-from-phone)

### Troubleshooting Quick Reference

- **macOS users:** Temporarily [disable SIP](#macos-notes) before obtaining keys
- **Windows users:** For UI issues, [use Windows Terminal](#windows-notes)
- **AI assistant integration:** See [MCP Integration Guide](#mcp-integration)
- **Can't get keys:** See the [FAQ](https://github.com/sjzar/chatlog/issues/197)

## Installation Guide

### Install from Source

```bash
go install github.com/sjzar/chatlog@latest
```
> 💡 **Tip:** Some features require cgo; make sure you have a C build environment.

### Download Prebuilt Binaries

Download the release for your OS from the [Releases](https://github.com/sjzar/chatlog/releases) page.

## Usage Guide

### Terminal UI Mode

The easiest way is to use the Terminal UI:

```bash
chatlog
```
Navigation:
- Use `↑` `↓` keys to move through menu items
- Press `Enter` to select
- Press `Esc` to return
- Press `Ctrl+C` to exit

### Command Line Mode

For advanced users, use these commands:

```bash
# Get WeChat data keys
chatlog key

# Decrypt database files
chatlog decrypt

# Start HTTP service
chatlog server
```

### Docker Deployment

Due to isolation, you can't obtain keys within Docker. Get key data on your host first.

Common for NAS or server deployment — see the [Docker Deployment Guide](docs/docker.md) for details.

**0. Get your keys**
```shell
$ chatlog key
Data Key: [c0163e***ac3dc6]
Image Key: [38636***653361]
```

**1. Pull the image**
```shell
docker pull sjzar/chatlog:latest
# or
docker pull ghcr.io/sjzar/chatlog:latest
```
> 💡 **Images:**  
> - Docker Hub: https://hub.docker.com/r/sjzar/chatlog  
> - GitHub Container Registry: https://ghcr.io/sjzar/chatlog

**2. Run the container**
```shell
$ docker run -d \
  --name chatlog \
  -p 5030:5030 \
  -v /path/to/your/wechat/data:/app/data \
  sjzar/chatlog:latest
```

### Migrating Chat History from Phone

If your desktop chat logs are incomplete, migrate from your phone:

1. On your phone, go to `Me - Settings - General - Chat Log Migration & Backup`
2. Select `Migrate - Migrate to Computer` and follow the instructions
3. After migration, rerun `chatlog` to get the keys and decrypt your data

> This operation does NOT remove anything from your phone; it just copies to your PC.

## Platform-Specific Notes

### Windows Notes

If you encounter display issues (garbled UI, etc.), use [Windows Terminal](https://github.com/microsoft/terminal).

### macOS Notes

macOS users must temporarily disable SIP (System Integrity Protection) before obtaining keys:

1. **Disable SIP:**
    ```shell
    # Enter Recovery Mode
    # Intel Mac: Restart and hold Command + R
    # Apple Silicon: Restart and hold the power button

    # In Recovery, open Terminal and run:
    csrutil disable

    # Reboot
    ```
2. **Install required tools**
    ```shell
    xcode-select --install
    ```
3. **After fetching keys:** You can re-enable SIP (`csrutil enable`); this won't affect usage

> Apple Silicon users: Ensure WeChat, chatlog, and Terminal are *not* running under Rosetta.

## HTTP API

After starting the HTTP service (`http://127.0.0.1:5030` by default), use these APIs:

### Querying Chat Logs

```
GET /api/v1/chatlog?time=2023-01-01&talker=wxid_xxx
```
Parameters:
- `time`: Date range, e.g. `YYYY-MM-DD` or `YYYY-MM-DD~YYYY-MM-DD`
- `talker`: Chat target (wxid, group ID, remark, nickname, etc.)
- `limit`: Number of records to return
- `offset`: Pagination offset
- `format`: Output format: `json`, `csv`, or plain text

### Other API Endpoints

- **Contact list**: `GET /api/v1/contact`
- **Group list**: `GET /api/v1/chatroom`
- **Session list**: `GET /api/v1/session`

### Multimedia Content

Media files are served over HTTP:

- **Images**: `GET /image/<id>`
- **Videos**: `GET /video/<id>`
- **Files**: `GET /file/<id>`
- **Audio**: `GET /voice/<id>`
- **Raw data**: `GET /data/<relative path>`

Image/video/file URLs will return a 302 redirect; audio files are transcoded to MP3 on the fly. Encrypted images are decrypted in real-time when requested.

## Webhook

Enable auto-decrypt. When specific new messages are received, they can be pushed to a specified URL via HTTP POST.

For local services, callback latency is about 13s; remote sync callback ~45s.

**Example configuration:**
In `$HOME/.chatlog/chatlog.json` (on Windows, `%USERPROFILE%/.chatlog/chatlog.json`):
```json
{
  "history": [],
  "last_account": "wxuser_x",
  "webhook": {
    "host": "localhost:5030",
    "items": [
      {
        "url": "http://localhost:8080/webhook",
        "talker": "wxid_123",
        "sender": "",
        "keyword": ""
      }
    ]
  }
}
```
Or, set via environment variables in server mode.

## MCP Integration

Chatlog supports the MCP (Model Context Protocol) and can be seamlessly integrated with MCP-compatible AI assistants.

Start HTTP service, then access the Streamable HTTP endpoint:

```
GET /mcp
```

**Example integrations:**

- **ChatWise**: Add `http://127.0.0.1:5030/mcp` in tool settings
- **Cherry Studio**: Add `http://127.0.0.1:5030/mcp` in MCP server settings

For clients that don't support Streamable HTTP, use [mcp-proxy](https://github.com/sparfenyuk/mcp-proxy):

- **Claude Desktop**: Supported via mcp-proxy; requires `claude_desktop_config.json`
- **Monica Code**: Supported via mcp-proxy; configure in VSCode plugin

See the [MCP Integration Guide](docs/mcp.md) for more.

## Prompt Examples

We've compiled some sample prompts to help you get the most out of Chatlog and your AI assistant. See the [Prompt Guide](docs/prompt.md) for more examples.

You are welcome to share feedback and your own prompts in [Discussions](https://github.com/sjzar/chatlog/discussions)!

## Disclaimer

⚠️ **Important: Before using Chatlog, please carefully read the full [Disclaimer](./DISCLAIMER.md).**

This project is for learning, research, and personal legal use only; unauthorized or illegal use of others' data is strictly forbidden. Downloading, installing, or using this tool means you accept all the terms of the disclaimer and assume full responsibility and risk.

**This project is entirely free and open source; any attempt to charge for it is unrelated to the project.**

## License

Project is open source under [Apache-2.0 License](./LICENSE).

## Privacy Policy

This project does not collect any user data. All data processing is local to your device. For external services, see their privacy policy.

## Thanks

- [@0xlane](https://github.com/0xlane) for [wechat-dump-rs](https://github.com/0xlane/wechat-dump-rs)
- [@xaoyaoo](https://github.com/xaoyaoo) for [PyWxDump](https://github.com/xaoyaoo/PyWxDump)
- [@git-jiadong](https://github.com/git-jiadong) for [go-lame](https://github.com/git-jiadong/go-lame) and [go-silk](https://github.com/git-jiadong/go-silk)
- [Anthropic MCP](https://github.com/modelcontextprotocol)
- And all the contributors of Go open source libraries!
