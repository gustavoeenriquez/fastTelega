# Demo B — Claude Desktop Integration Guide

A step-by-step walkthrough for connecting **TelegramMCPServer** (StdIO) to
**Claude Desktop**, so you can control your Telegram bot through natural language.

---

## How it works

```
Claude Desktop ──StdIO (child process)──► TelegramMCPServer.exe ──HTTPS──► Telegram
```

Claude Desktop launches `TelegramMCPServer.exe` as a child process and communicates
with it over standard input/output. The server translates Claude's tool calls into
Telegram Bot API requests.

---

## Prerequisites

| What | Where |
|------|-------|
| Claude Desktop | [claude.ai/download](https://claude.ai/download) |
| `TelegramMCPServer.exe` | Compile `Examples/TelegramMCP/TelegramMCPServer.dpr` |
| Bot token | Talk to [@BotFather](https://t.me/BotFather) on Telegram |

**Important:** The StdIO server runs as a local process only — it is never exposed
to the network, making it safe for local use.

---

## Step 1 — Compile TelegramMCPServer

1. Open `Examples/TelegramMCP/TelegramMCPServer.dpr` in the Delphi IDE.
2. Compile as **Console Application** (Win32 or Win64).
3. Note the full path to the generated `.exe`.

---

## Step 2 — Edit claude_desktop_config.json

Open the Claude Desktop configuration file:

```
Windows: %APPDATA%\Claude\claude_desktop_config.json
macOS:   ~/Library/Application Support/Claude/claude_desktop_config.json
```

Add the `telegram` server entry inside `"mcpServers"`:

```json
{
  "mcpServers": {
    "telegram": {
      "command": "C:\\path\\to\\TelegramMCPServer.exe",
      "env": {
        "TELEGRAM_TOKEN": "123456:ABC-your-token"
      }
    }
  }
}
```

> **Windows paths:** Use double backslashes (`\\`) or forward slashes (`/`).
> `"command": "C:/Users/you/Desktop/TelegramMCPServer.exe"` also works.

---

## Step 3 — Restart Claude Desktop

Close and reopen Claude Desktop. When the server loads correctly:

- A **hammer icon** (🔨) appears in the bottom-left of the chat input box.
- Clicking it shows a list of available Telegram tools.

If the icon does not appear, check:

1. The path in `command` is correct and the `.exe` exists.
2. The `TELEGRAM_TOKEN` value is valid (test it with `getMe`).
3. Run `TelegramMCPServer.exe` manually in a terminal — diagnostic messages
   go to stderr (visible in the terminal) to help diagnose connection issues.

---

## Step 4 — Get your chat ID

To send messages, Claude needs a **chat ID**. The easiest ways to get yours:

**Method A — via browser:**
1. Send any message to your bot.
2. Open: `https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates`
3. Find `"chat":{"id": 123456789}` in the response.

**Method B — ask Claude:**
> "What is my bot's username and who has chatted with it recently?"

Claude will use `telegram_get_me` and can help you find the chat ID if you
forward it the Telegram update information.

**Group IDs** are negative (e.g. `-1001234567890`). Add your bot to the group
and send a message before using `getUpdates`.

---

## Example prompts

### Basic messaging

> "Send 'Hello team! 👋' to chat ID 123456789"

> "Send an HTML-formatted message to chat 987654321:
> **Build completed** ✅ at 14:32 — all tests passed."

### Sharing content

> "Send the file C:\Reports\Q4_Summary.pdf to chat 123456789 with the caption
> 'Q4 report — please review by Friday'"

> "Send a photo C:\screenshots\error.png to my chat with caption 'Bug found on login page'"

### Location and contacts

> "Send the location of the Eiffel Tower to chat 123456789"

> "Send a contact card: name 'Support Team', phone '+1 800 555 0000', to chat 123456789"

### Fun

> "Roll a dice and send it to chat 123456789 — let's see what we get!"

> "Send three different dice to chat 123456789: a regular die, a dartboard, and a slot machine"

### Automation workflows

> "Send a Telegram message to chat 123456789 saying the build just finished
>  successfully. Include the current time."

> "Forward message number 42 from chat -100987654321 to chat 123456789"

> "Show me the typing indicator in chat 123456789 for a few seconds, then send
> 'Processing complete — your report is ready.'"

---

## Available tools

| Tool | What you can ask |
|------|-----------------|
| `telegram_get_me` | "What is my bot's username?" |
| `telegram_send_message` | "Send a message to chat X saying..." |
| `telegram_send_photo` | "Send photo at path... to chat X" |
| `telegram_send_document` | "Send file at path... to chat X" |
| `telegram_send_location` | "Send coordinates / place name to chat X" |
| `telegram_send_contact` | "Send contact card name/phone to chat X" |
| `telegram_send_dice` | "Roll a dice in chat X" |
| `telegram_send_chat_action` | "Show typing in chat X" |
| `telegram_forward_message` | "Forward message M from chat A to chat B" |
| `telegram_get_chat` | "Get info about chat X" |

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Hammer icon missing | Verify the exe path in the config; check file exists |
| "Token error" on first use | Verify `TELEGRAM_TOKEN` is correct; test with `getMe` |
| "Chat not found" | Your bot must have been messaged at least once by that user/group |
| "Bot can't send to this chat" | Add the bot to the group or the user must start the bot first |
| Tools listed but calls fail | Run the exe manually; stderr shows diagnostic messages |

---

## Security notes

- The bot token is passed via `env` in the config — it is never stored in the executable.
- The StdIO server is a **child process of Claude Desktop** — it has no open network ports.
- The bot can only interact with chats where a user has previously messaged it,
  or groups where it has been added.
