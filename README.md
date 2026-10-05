# Snapgent

> Turn any AI chat into an agent that drives your editor.

Snapgent is a browser extension plus a small local bridge that lets a normal AI chat (DeepSeek, Z.ai, and Gemini) **read, edit, run, and inspect your project directly in VS Code** through the official MCP (Model Context Protocol) server. You describe what you want in plain language — the AI writes Snapgent commands into its reply, the extension executes them on your machine, and the result is fed straight back to the AI. No API key. No terminal. No copy-pasting code.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![Manifest V3](https://img.shields.io/badge/Chrome-Manifest%20V3-blue)
![Version](https://img.shields.io/badge/version-1.5.5-green)

---

## Table of contents

- [What it does](#what-it-does)
- [How it works](#how-it-works)
- [Supported AI providers](#supported-ai-providers)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Architecture](#architecture)
- [Project structure](#project-structure)
- [Development](#development)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## What it does

Snapgent gives a web-based AI chat real hands inside your editor. With it, the AI can:

- **Read and edit files** in your workspace (create, modify, move, delete).
- **Run shell commands** inside the workspace folder.
- **Execute code** (e.g. Luau) and read the returned result.
- **Inspect and search** your project — game tree, scripts, instances, console output.
- **Generate assets and models** when a connected server supports it.
- **Talk to other MCP servers** alongside VS Code, not just the editor itself.

Everything happens through the connected MCP server. You never leave the chat window, and you never touch a terminal.

---

## How it works

Snapgent has three parts that talk to each other over a local WebSocket:

```
┌─────────────┐     plain-text      ┌──────────────────┐    WebSocket    ┌────────────┐    MCP    ┌───────────┐
│  AI chat    │ ───  commands  ───▶ │  Snapgent        │ ──────────────▶ │  Bridge    │ ────────▶ │  VS Code  │
│  (browser)  │ ◀──  results   ─── │  extension       │ ◀────────────── │ (local)    │ ◀──────── │  + MCP    │
└─────────────┘                     └──────────────────┘                 └────────────┘           └───────────┘
```

1. **You** type a request into the AI chat.
2. The **AI** replies with a Snapgent command — a plain-text JSON object in a fenced code block (or a `###LUA###` block for code).
3. The **extension** watches the reply, detects the command, and forwards it to the local bridge.
4. The **bridge** runs the command against the connected MCP server(s) — VS Code by default.
5. The **result** (success or a formatted error) is sent back into the chat as the next message, and the AI keeps going on its own.

Because commands are just text in the AI's reply, they work on any chat site Snapgent supports — no vendor plugins required.

---

## Supported AI providers

| Provider | URL | Status | Notes |
|---|---|---|---|
| **DeepSeek** | `chat.deepseek.com` | ✅ Recommended | Most stable, best tool adherence |
| **Z.ai (GLM)** | `chat.z.ai` | ✅ Supported | Svelte DOM, code-block wrapper masking |
| **Gemini** | `gemini.google.com` | ✅ Supported | Angular DOM, Quill composer; can stop using tools in long sessions |

> Which sites the extension activates on is controlled by `manifest.json` (`content_scripts` + `host_permissions`) and `PROVIDER_URLS` in `background.js`.

---

## Requirements

- **Google Chrome** or **Microsoft Edge** (Manifest V3).
- **VS Code** (or a compatible editor exposing an MCP server).
- **Windows** or **macOS** for the bridge.
- The MCP server enabled in your editor (first time only — see below).

---

## Installation

### 1. Load the extension

1. Open `edge://extensions` (Edge) or `chrome://extensions` (Chrome).
2. Enable **Developer mode** (toggle in the top-right corner).
3. Click **Load unpacked**.
4. Select the `snapgent-extension` folder.
5. The Snapgent icon appears in your toolbar — the extension is active.

### 2. Set up the bridge

1. Grab the bridge (`bridge.exe`, `start.bat`, `MacOS_Start.command`) from this repo or the releases page.
2. Open VS Code and load a project/place.
3. **Enable the MCP server** (first time only): click the AI assistant button in the top bar, then **… → Manage MCP Servers → Enable as MCP Server**.
4. **Run the bridge**:
   - **Windows** — double-click `start.bat`.
   - **macOS** — run `MacOS_Start.command`.
   A small window opens and stays open while the bridge is running.
   > On macOS the first launch shows a Gatekeeper warning (normal for downloaded scripts): click **Done**, then **System Settings → Privacy & Security**, scroll down and click **Open Anyway**.

### 3. Start a session

1. Go to a supported chat site (e.g. `https://chat.deepseek.com`).
2. Open a new chat — Snapgent only activates on the exact supported addresses.
3. Click **Start session** in the Snapgent panel.
4. Describe what you want to build. The AI takes it from there.

---

## Usage

Once a session is running, just talk to the AI normally:

> "Read `src/index.js` and add a function that validates emails."

> "Run the test suite and fix anything that fails."

The AI will emit commands, Snapgent executes them, and you watch the results appear in the chat. You can step in at any time with a new instruction.

**Tips**

- Keep the bridge window open while you work — closing it stops the connection.
- On **Gemini**, start fresh chats for long tasks; it can stop using tools in long sessions.

---

## Architecture

The extension is split into a **provider-agnostic core** and **per-site providers**. The core never touches a host site's DOM directly — it only talks to the `ZSProvider` interface.

```
core/config.js        system prompt, feedback strings, tool categories   (global ZS)
core/parser.js        Snapgent command parsing — pure string logic      (global ZSParse)
core/main.js          agentic loop, UI, camouflage, session state        (uses ZSProvider)
providers/deepseek.js DeepSeek-specific: DOM selectors, generation
                      detection, send mechanics, composer modes         (global ZSProvider)
providers/glm.js      Z.ai / GLM: Svelte DOM, code-block wrapper masking (global ZSProvider)
providers/gemini.js   Google Gemini: Angular DOM, Quill composer,
                      code-block masking                               (global ZSProvider)
background.js         WebSocket to the local bridge (provider-agnostic)
```

### Adding a new provider

No core changes required:

1. Write `providers/<site>.js` exporting the same `ZSProvider` interface.
2. Add the site's URL pattern to `manifest.json` (`content_scripts` + `host_permissions`).
3. Add it to `PROVIDER_URLS` in `background.js`.

---

## Project structure

```
snapgent-rilis/
├── snapgent-extension/        # The browser extension (load this folder unpacked)
│   ├── core/                  # Provider-agnostic logic
│   │   ├── config.js          # System prompt, strings, tool categories
│   │   ├── parser.js          # Command parser
│   │   └── main.js            # Agentic loop + UI
│   ├── providers/             # Per-AI-site adapters
│   ├── background.js          # WebSocket service worker
│   ├── manifest.json          # MV3 manifest
│   ├── overlay.css
│   ├── popup.html / popup.js
│   └── README.md              # Extension-level docs
├── assets/                    # Icons and images
├── config.json                # MCP server config (bridge)
├── start.bat                  # Windows bridge launcher
├── MacOS_Start.command        # macOS bridge launcher
├── bridge.exe                 # Local bridge (Windows)
└── vscode_mcp.exe             # MCP server binary
```

---

## Development

### Smoke tests

Plain Node, no dependencies. Run from `snapgent-extension/`:

```bash
node test-parser.js     # command parser (core/parser.js)
node test-chatgpt.js    # ChatGPT reply reading (providers/chatgpt.js)
```

Both print `PASS`/`FAIL` per case and exit non-zero on failure.

### Notes

- `core/main.js` only calls the `ZSProvider` interface — keep DOM logic inside providers.
- Commands are detected as **plain text** in the AI reply, so formatting rules matter (one command per reply, inside a fenced code block; `###LUA###` blocks for code).
- The bridge listens on local port **17613**. `start.bat` frees the port from any previous instance before launching.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| "Bridge offline" error | Bridge not running, or VS Code closed | Start `start.bat` / `MacOS_Start.command`; ensure VS Code is open |
| "No VS Code instance connected" | MCP server disabled | Enable it: assistant → … → Manage MCP Servers |
| "Extension was reloaded" | Tab running a stale extension version | Reload the page (F5) |
| Commands never run | Wrong site or address | Use an exact supported URL, open a new chat |

---

## Contributing

Contributions are welcome — especially new provider adapters.

1. Fork the repo and create a feature branch.
2. Keep DOM logic inside `providers/`; keep the core provider-agnostic.
3. Run the smoke tests before opening a PR.
4. Open a pull request with a clear description of the change.

---

## License

Released under the [MIT License](LICENSE).

Copyright (c) 2026 herdo dimas pratirto
