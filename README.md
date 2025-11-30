# 🚀 NeoKEX - FCA

[![npm version](https://img.shields.io/npm/v/neokex-fca.svg)](https://www.npmjs.com/package/neokex-fca)
[![npm downloads](https://img.shields.io/npm/dm/neokex-fca.svg)](https://www.npmjs.com/package/neokex-fca)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node.js Version](https://img.shields.io/node/v/neokex-fca.svg)](https://nodejs.org)

💁 **NeoKEX - FCA** (neokex-fca) is an advanced Facebook Chat API (FCA) client built for **reliable**, **real-time**, and **modular** interaction with Facebook Messenger. Inspired by **ws3-fca**, this project is designed with modern bot development in mind, offering full control over Messenger automation through a clean, stable interface.

---

## 📚 Documentation

- **[Theme Features](THEME_FEATURES.md)** - Comprehensive guide to theme management
- **[Changelog](CHANGELOG.md)** - Version history and updates
- **[Examples](examples/)** - Code examples and usage patterns

### Support & Issues

For documentation and support, please visit:
[https://github.com/NeoKEX](https://github.com/NeoKEX)

If you encounter issues or want to contribute, feel free to open an issue on GitHub:
[https://github.com/NeoKEX/neokex-fca/issues](https://github.com/NeoKEX/neokex-fca/issues)

---

## ✨ Features

* 🔐 **Precise Login Mechanism**
  Dynamically scrapes Facebook's login form and submits tokens for secure authentication.

* 💬 **Real-time Messaging**
  Send and receive messages (text, attachments, stickers, replies).

* 📝 **Message Editing**
  Edit your bot’s messages in-place.

* ✍️ **Typing Indicators**
  Detect and send typing status.

* ✅ **Message Status Handling**
  Mark messages as delivered, read, or seen.

* 📂 **Thread Management**

  * Retrieve thread details
  * Load thread message history
  * Get lists with filtering
  * Pin/unpin messages

* 👤 **User Info Retrieval**
  Access name, ID, profile picture, and mutual context.

* 🖼️ **Sticker API**
  Search stickers, list packs, fetch store data, AI-generated stickers.

* 💬 **Post Interaction**
  Comment and reply to public Facebook posts.

* ➕ **Follow/Unfollow Users**
  Automate social interactions.

* 🌐 **Proxy Support**
  Full support for custom proxies.

* 🧱 **Modular Architecture**
  Organized into pluggable models for maintainability.

* 🛡️ **Robust Error Handling**
  Retry logic, consistent logging, and graceful failovers.

---

## ⚙️ Installation

> **Requirements:** Node.js v18.0.0 or higher

```bash
npm install neokex-fca
```

Or use the latest version:

```bash
npm install neokex-fca@latest
```

---

## 🔒 Security Warning

**IMPORTANT:** `appstate.json` contains your Facebook session credentials and should be treated as sensitive information.

- ⚠️ **Never commit `appstate.json` to version control**
- ⚠️ **Never share your `appstate.json` file publicly**
- ⚠️ **Keep it in `.gitignore`** (already configured in this project)
- ⚠️ **Use environment-specific credentials** for production deployments

Your `appstate.json` gives full access to your Facebook account. Treat it like a password!

---

## 🚀 Getting Started

### 1. Generate `appstate.json`

This file contains your Facebook session cookies. Follow these steps:

1. **Install a cookie export extension** for your browser:
   - Chrome/Edge: "C3C FbState" or "CookieEditor"
   - Firefox: "Cookie-Editor"

2. **Log in to Facebook** in your browser

3. **Export cookies** using the extension and save them in this format:

```json
[
  {
    "key": "c_user",
    "value": "your-user-id"
  },
  {
    "key": "xs",
    "value": "your-xs-value"
  }
]
```

4. **Save the file** as `appstate.json` in your project root

**Tutorial:** If you need detailed instructions, follow this tutorial: **[Getting AppState](https://appstate-tutorial-ws3.pages.dev)**

---

### 2. Basic Usage Example

```js
const fs = require("fs");
const path = require("path");
const { login } = require("neokex-fca");

let credentials;
try {
  credentials = { appState: JSON.parse(fs.readFileSync("appstate.json", "utf8")) };
} catch (err) {
  console.error("❌ appstate.json is missing or malformed.", err);
  process.exit(1);
}

console.log("Logging in...");

login(credentials, {
  online: true,
  updatePresence: true,
  selfListen: false,
  randomUserAgent: false
}, async (err, api) => {
  if (err) return console.error("LOGIN ERROR:", err);

  console.log(`✅ Logged in as: ${api.getCurrentUserID()}`);

  const commandsDir = path.join(__dirname, "modules", "commands");
  const commands = new Map();

  if (!fs.existsSync(commandsDir)) fs.mkdirSync(commandsDir, { recursive: true });

  for (const file of fs.readdirSync(commandsDir).filter(f => f.endsWith(".js"))) {
    const command = require(path.join(commandsDir, file));
    if (command.name && typeof command.execute === "function") {
      commands.set(command.name, command);
      console.log(`🔧 Loaded command: ${command.name}`);
    }
  }

  api.listenMqtt(async (err, event) => {
    if (err || !event.body || event.type !== "message") return;

    const prefix = "/";
    if (!event.body.startsWith(prefix)) return;

    const args = event.body.slice(prefix.length).trim().split(/ +/);
    const commandName = args.shift().toLowerCase();

    const command = commands.get(commandName);
    if (!command) return;

    try {
      await command.execute({ api, event, args });
    } catch (error) {
      console.error(`Error executing ${commandName}:`, error);
      api.sendMessageMqtt("❌ An error occurred while executing the command.", event.threadID, event.messageID);
    }
  });
});
```

---

## 🤖 AI Features

**NeoKEX - FCA** includes support for AI-generated stickers and themes:

### AI Stickers
```js
const aiStickers = await api.getAiStickers({ limit: 10 });
console.log(aiStickers);
```

### AI Themes (✨ Now Fixed!)
Generate and apply custom AI-powered chat themes with text prompts:

```js
// Generate an AI theme
const aiThemes = await api.createAITheme("vibrant purple pink ocean sunset");

if (aiThemes && aiThemes.length > 0) {
  console.log(`Generated theme ID: ${aiThemes[0].id}`);
  
  // Apply the theme to a conversation
  await api.setThreadThemeMqtt(threadID, aiThemes[0].id);
  console.log("AI theme applied successfully!");
}
```

**Note:** AI theme generation requires account-level access from Facebook. Not all accounts/regions have this feature enabled. If unavailable, you can still use 90+ standard themes via `api.getTheme(threadID)`.

### Standard Themes
Browse and apply from 90+ pre-made themes:

```js
// Get all available themes
const themes = await api.getTheme(threadID);
console.log(`Found ${themes.length} themes`);

// Apply a theme
await api.setThreadThemeMqtt(threadID, themes[0].id);
```

### Check Current Theme
```js
const currentTheme = await api.getThemeInfo(threadID);
console.log(`Thread: ${currentTheme.threadName}`);
console.log(`Color: ${currentTheme.color}`);
console.log(`Emoji: ${currentTheme.emoji}`);
```

---

## 🧪 Examples

Check the **[examples/](examples/)** directory for comprehensive usage examples:
- `simple-bot.js` - Basic bot setup and message handling
- `theme-usage-example.js` - Theme management examples
- `test-bot.js` - Full-featured test bot
- And more!

---

## 📦 Publishing

Before publishing to npm, validate your package:

```bash
npm pack --dry-run
```

---

## 🙌 Credits

* 🚀 **Credits to NeoKEX** – [https://github.com/NeoKEX](https://github.com/NeoKEX)
* 🔧 **NeoKEX Team** – Development, maintenance, and feature contributions
* 🔮 **Inspired by ws3-fca** – Originally developed by @NethWs3Dev and @CommunityExocore

> Copyright (c) 2025 NeoKEX

---

## 📊 License

**MIT** – Free to use, modify, and distribute. Attribution appreciated.

See [LICENSE](LICENSE) for full license text.

---

## 🔗 Links

- **npm Package:** [https://www.npmjs.com/package/neokex-fca](https://www.npmjs.com/package/neokex-fca)
- **GitHub Repository:** [https://github.com/NeoKEX/neokex-fca](https://github.com/NeoKEX/neokex-fca)
- **Issue Tracker:** [https://github.com/NeoKEX/neokex-fca/issues](https://github.com/NeoKEX/neokex-fca/issues)
