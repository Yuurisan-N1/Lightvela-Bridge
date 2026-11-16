<div align="center">

<img width="100%" alt="header" src="https://capsule-render.vercel.app/api?type=waving&height=210&text=LightVela%20Bridge%20Bot&fontAlign=50&fontAlignY=36&fontSize=56&desc=OpenAI-Compatible%20Local%20Bridge%20for%20LightVela%20AI&descAlign=50&descAlignY=58"/>

<img alt="typing" src="https://readme-typing-svg.demolab.com?font=Inter&size=18&duration=3000&pause=650&center=true&vCenter=true&width=900&lines=LightVela+Account+Token+Login;WebSocket+Tunnel+to+Sandbox+Instance;OpenAI-Compatible+%2Fv1%2Fchat%2Fcompletions+Endpoint;Streaming+%26+Non-Streaming+Support;Multi-Token+%7C+Auto+Rotate+on+Credit+Exhaustion"/>

<p>
  <img alt="python" src="https://img.shields.io/badge/Python-3.12%2B-3776AB?logo=python&logoColor=white"/>
  <img alt="platform" src="https://img.shields.io/badge/Platform-LightVela%20AI-111111"/>
  <img alt="api" src="https://img.shields.io/badge/API-OpenAI%20Compatible-111111"/>
  <img alt="author" src="https://img.shields.io/badge/by-Yuurisandesu-111111"/>
</p>

<p>
  <b>LightVela Bridge Bot</b> is a local OpenAI-compatible bridge for LightVela AI.<br/>
  It reads LightVela account tokens from <code>data.txt</code>, opens a WebSocket tunnel into the account's sandbox instance, and exposes a local HTTP server on port 1234 that accepts <code>/v1/chat/completions</code> and <code>/v1/models</code> requests in the OpenAI format, with both streaming and non-streaming support. When a token's credit is exhausted it rotates to the next one automatically.<br/>
  Built and distributed by <b>Yuurisandesu</b>.
</p>

</div>

---

## Table of Contents

- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Running the Bot](#running-the-bot)
- [Connecting a Client](#connecting-a-client)
- [Features](#features)
- [File Structure](#file-structure)
- [Disclaimer](#disclaimer)

---

## Requirements

- Python 3.12+
- pip
- Git
- A LightVela account with at least one running sandbox instance

---

## Installation

### Install Python

**Windows:**

1. Download the installer from https://www.python.org/downloads/ and run it
2. On the first screen tick **Add python.exe to PATH**, then continue with the default options
3. Open Command Prompt and verify:

```cmd
python --version
```

**Linux (Ubuntu / Debian):**

```bash
sudo apt-get update
sudo apt-get install -y python3 python3-pip
python3 --version
```

**macOS:**

```bash
brew install python
python3 --version
```

> If you do not have Homebrew, install it first from https://brew.sh

**Termux (Android):**

```bash
pkg update && pkg upgrade -y
pkg install python git -y
python --version
```

---

### Clone and Install

**Clone the repository:**

```bash
git clone https://github.com/Yuurisan-N1/Lightvela-Bridge.git
cd Lightvela-Bridge
```

**Install dependencies:**

```bash
pip install aiohttp yuurisan
```

---

## Configuration

### 1. Account Tokens (data.txt)

Fill `data.txt` with one LightVela Bearer token per line. Blank lines are ignored:

```
a9311e544aae722da8d...
a9311e544aae722da8d...
```

To get your token, open LightVela in the browser, open DevTools, go to the Network tab, and copy the value of the `Authorization` header (without the `Bearer ` prefix) from any API request.

### 2. Bot Settings (config.json)

`sleep_seconds` controls how often the tunnel refreshes. The default is 3600 (one hour).

```json
{
  "settings": {
    "sleep_seconds": 3600
  }
}
```

---

## Running the Bot

**Linux / macOS:**

```bash
python bot.py
```

**Windows:**

```cmd
python bot.py
```

**Termux (Android):**

```bash
python bot.py
```

Once started, the bridge logs a green line and begins accepting requests:

```
The bridge is listening on port 1234 and waits for requests
```

Press `Ctrl+C` to stop the bridge cleanly.

**Run in background with screen (Linux / macOS / Termux):**

```bash
screen -S lightvela
python bot.py
```

Detach with `Ctrl+A` then `D`. Reattach anytime:

```bash
screen -r lightvela
```

---

## Connecting a Client

Point any OpenAI-compatible client at:

```
Base URL : http://127.0.0.1:1234/v1
API Key  : any non-empty string (not checked by the bridge)
Model    : yuuri-lightvela
```

**curl example:**

```bash
curl http://127.0.0.1:1234/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer dummy" \
  -d '{
    "model": "yuuri-lightvela",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

**Streaming example:**

```bash
curl http://127.0.0.1:1234/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer dummy" \
  -d '{
    "model": "yuuri-lightvela",
    "stream": true,
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

The bridge works with any client that supports a custom base URL, including Open WebUI, SillyTavern, and Chatbox.

---

## Features

### WebSocket Tunnel
The bridge opens a WebSocket connection directly into the LightVela sandbox instance. Terminal commands are run through the PTY channel and a proxy channel is used to forward HTTP traffic from the local server to the sandbox gateway, with no extra tooling required.

### OpenAI-Compatible Endpoint
The local server on port 1234 accepts POST requests to `/v1/chat/completions` and GET requests to `/v1/models` in the standard OpenAI format. The model name reported is `yuuri-lightvela`.

### Streaming and Non-Streaming
Both `stream: true` and `stream: false` are supported. Streaming responses send SSE chunks in the OpenAI delta format and close with a `[DONE]` marker. A keep-alive comment is sent every 15 seconds to prevent proxy timeouts.

### Session Reuse
Agent sessions inside the sandbox are reused across multiple requests to avoid the overhead of creating a new session for every call. A session is rotated after a configured number of uses.

### Multi-Token and Auto Rotation
Multiple tokens can be loaded from `data.txt`. When a token's credit is exhausted or the account is unauthorized, the bridge rotates to the next token and reconnects the tunnel automatically.

### Tunnel Auto-Reconnect
If the tunnel drops between requests the bridge reconnects before serving the next call without returning an error to the client.

### Proxy Support
The bot reads proxy configuration from `proxy.txt`. HTTP and SOCKS5 are both supported. Leave the file empty to connect directly.

---

## File Structure

```text
Lightvela-Bridge/
├── utils/
│   ├── __init__.py                      # Package marker
│   └── banner.py                        # Banner and terminal title via yuurisan
├── bot.py                               # Bridge server, tunnel, RPC, and OpenAI handler
├── config.json                          # Tunnel refresh interval setting
├── data.txt                             # LightVela Bearer tokens, one per line
├── proxy.txt                            # Proxy list (optional)
├── LICENSE                              # License file
└── README.md                            # This file
```

---

## Disclaimer

This tool is built for educational and technical exploration purposes. Use it wisely and at your own responsibility.

---

<div align="center">
<img width="100%" alt="footer" src="https://capsule-render.vercel.app/api?type=waving&height=120&section=footer"/>
</div>