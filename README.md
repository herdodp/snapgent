<div align="center">

<a href="https://github.com/herdodp/snapgent">
<img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=800&size=52&duration=3000&pause=1000&color=FACC15&center=true&vCenter=true&width=600&height=90&lines=Snapgent;%3E+AI+agent+for+VS+Code" alt="Snapgent" />
</a>

<p><b>Turn any AI chat into an agent that drives your editor.</b></p>

[![License](https://img.shields.io/badge/LICENSE-MIT-FACC15?style=flat-square&labelColor=111111&logo=opensourceinitiative&logoColor=FACC15)](LICENSE)
[![Manifest](https://img.shields.io/badge/MANIFEST-V3-FACC15?style=flat-square&labelColor=111111&logo=googlechrome&logoColor=FACC15)](https://developer.chrome.com/docs/extensions/mv3/)
[![Version](https://img.shields.io/badge/VERSION-1.0-FACC15?style=flat-square&labelColor=111111&logo=semanticrelease&logoColor=FACC15)](../../releases)
[![MCP](https://img.shields.io/badge/PROTOCOL-MCP-FACC15?style=flat-square&labelColor=111111&logo=modelcontextprotocol&logoColor=FACC15)](#)

</div>

```console
herdo@snapgent:~$ npm run snapgent

  ███████╗███╗   ██╗ █████╗ ██████╗  ██████╗ ███████╗███╗   ██╗████████╗
  ██╔════╝████╗  ██║██╔══██╗██╔══██╗██╔════╝ ██╔════╝████╗  ██║╚══██╔══╝
  ███████╗██╔██╗ ██║███████║██████╔╝██║  ███╗█████╗  ██╔██╗ ██║   ██║
  ╚════██║██║╚██╗██║██╔══██║██╔═══╝ ██║   ██║██╔══╝  ██║╚██╗██║   ██║
  ███████║██║ ╚████║██║  ██║██║     ╚██████╔╝███████╗██║ ╚████║   ██║
  ╚══════╝╚═╝  ╚═══╝╚═╝  ╚═╝╚═╝      ╚═════╝ ╚══════╝╚═╝  ╚═══╝   ╚═╝

  > browser extension + local bridge for VS Code
  > no API key · no terminal · no copy-paste
```

Snapgent is a browser extension plus a small local bridge that lets a normal AI chat (DeepSeek, Z.ai, and Gemini) **read, edit, run, and inspect your project directly in VS Code** through the official MCP (Model Context Protocol) server. You describe what you want in plain language — the AI writes Snapgent commands into its reply, the extension executes them on your machine, and the result is fed straight back to the AI.

---

<table>
<tr>
<td width="33%" valign="top">

### 📖 Read & Edit
Create, modify, move and delete files in your workspace — all from the chat.

</td>
<td width="33%" valign="top">

### ⚡ Run
Run shell commands inside the workspace folder without touching a terminal.

</td>
<td width="33%" valign="top">

### 🔍 Inspect
Search files, folders, sizes, checksums and more — everything your project needs.

</td>
</tr>
</table>

---

## `01` &nbsp; How it works

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

## `02` &nbsp; Supported AI providers

| Provider | URL | Status | Notes |
|---|---|---|---|
| **DeepSeek** | `chat.deepseek.com` | ✅ Recommended | Most stable, best tool adherence |
| **Z.ai (GLM)** | `chat.z.ai` | ✅ Supported | Svelte DOM, code-block wrapper masking |
| **Gemini** | `gemini.google.com` | ✅ Supported | Angular DOM, Quill composer; can stop using tools in long sessions |

> Which sites the extension activates on is controlled by `manifest.json` (`content_scripts` + `host_permissions`) and `PROVIDER_URLS` in `background.js`.

---

## `03` &nbsp; Requirements

- **Google Chrome** or **Microsoft Edge** (Manifest V3).
- **VS Code** (or a compatible editor exposing an MCP server).
- **Windows**, **macOS**, or **Linux** for the bridge.
- The MCP server enabled in your editor (first time only — see below).

---

## `04` &nbsp; Installation

**① Load the extension**

1. Open `edge://extensions` (Edge) or `chrome://extensions` (Chrome).
2. Enable **Developer mode** (top-right toggle).
3. Click **Load unpacked**.
4. Select the `snapgent-extension` folder.
5. The Snapgent icon appears in your toolbar — the extension is active.

**② Set up the bridge**

1. Grab the bridge (`bridge.exe` / `bridge.py`, `vscode_mcp.exe` / `vscode_mcp.py`, `start.bat`, `start.sh`) from this repo or the releases page.
2. Open VS Code and open the folder you want the AI to work in.
3. Run the bridge:
   - **Windows** — double-click `start.bat`.
   - **macOS / Linux** — run `./start.sh`.

   A small window opens and stays open while the bridge runs.
   > On macOS the first launch shows a Gatekeeper warning (normal for downloaded scripts): click **Done**, then **System Settings → Privacy & Security**, scroll down and click **Open Anyway**.

**③ Start a session**

1. Go to a supported chat site (e.g. `https://chat.deepseek.com`).
2. Open a new chat — Snapgent only activates on the exact supported addresses.
3. Click **Start session** in the Snapgent panel.
4. Describe what you want to build. The AI takes it from there.

---

## `05` &nbsp; Usage

Once a session is running, just talk to the AI normally:

> "Read `src/index.js` and add a function that validates emails."

> "Run the test suite and fix anything that fails."

The AI emits commands, Snapgent executes them, and you watch results appear in the chat. You can step in at any time with a new instruction.

**Tips**

- Keep the bridge window open while you work — closing it stops the connection.
- On **Gemini**, start fresh chats for long tasks; it can stop using tools in long sessions.

---

## `06` &nbsp; Architecture

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

**Adding a new provider** — no core changes required:

1. Write `providers/<site>.js` exporting the same `ZSProvider` interface.
2. Add the site's URL pattern to `manifest.json` (`content_scripts` + `host_permissions`).
3. Add it to `PROVIDER_URLS` in `background.js`.

---

## `07` &nbsp; Project structure

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
├── start.sh                   # macOS / Linux bridge launcher
├── bridge.py                  # Local bridge (cross-platform)
├── bridge.exe                 # Local bridge (Windows, prebuilt)
├── vscode_mcp.py              # VS Code MCP server (cross-platform)
└── vscode_mcp.exe             # VS Code MCP server binary (Windows)
```

---

## `08` &nbsp; Development

Plain Node, no dependencies. Run from `snapgent-extension/`:

```bash
node test-parser.js     # command parser (core/parser.js)
node test-chatgpt.js    # ChatGPT reply reading (providers/chatgpt.js)
```

Both print `PASS`/`FAIL` per case and exit non-zero on failure.

**Notes**

- `core/main.js` only calls the `ZSProvider` interface — keep DOM logic inside providers.
- Commands are detected as **plain text** in the AI reply, so formatting rules matter (one command per reply, inside a fenced code block).
- The bridge listens on local port **17613**. `start.bat` frees the port from any previous instance before launching.

---

## `09` &nbsp; Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| "Bridge offline" error | Bridge not running, or VS Code closed | Start `start.bat` (Windows) or `./start.sh` (macOS/Linux); ensure VS Code is open |
| "No VS Code instance connected" | MCP server disabled | Enable it: assistant → … → Manage MCP Servers |
| "Extension was reloaded" | Tab running a stale extension version | Reload the page (F5) |
| Commands never run | Wrong site or address | Use an exact supported URL, open a new chat |

---

## `10` &nbsp; Contributing

Contributions are welcome — especially new provider adapters.

1. Fork the repo and create a feature branch.
2. Keep DOM logic inside `providers/`; keep the core provider-agnostic.
3. Run the smoke tests before opening a PR.
4. Open a pull request with a clear description of the change.

---

<div align="center">

**[ MIT License ](LICENSE)** &nbsp;·&nbsp; built with 🖤 and 💛

<sub>Snapgent — turn any AI chat into an agent that drives your editor.</sub>

</div>
