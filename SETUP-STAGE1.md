# FitFoundry Setup — Stage 1: Claude Desktop Configuration

Before FitFoundry can run, Claude Desktop needs three things configured: the Puppeteer MCP server for browser automation, the Indeed and Dice job board connectors, and optionally the Claude in Chrome extension for LinkedIn and Wellfound.

This stage is done once, entirely in Claude Desktop settings. No Cowork session is needed yet.

---

## Prerequisites

| Requirement | Notes |
|---|---|
| **Claude Desktop** | Download from anthropic.com if not already installed. Cowork mode is included. |
| **Node.js (LTS)** | Required for Puppeteer. Download from nodejs.org if not installed. See Step 1. |
| **Indeed account** | Free at indeed.com. Required to search Indeed. |
| **Dice account** | Free at dice.com. Required to search Dice. Skip if not planning to use it. |
| **Google Chrome** | Required only if you plan to search LinkedIn or Wellfound. |

---

## Step 1 — Install Node.js

Puppeteer requires Node.js. If you already have it, skip to Step 2.

1. Go to **nodejs.org** in your browser.
2. Download the **LTS** version (labeled "Recommended For Most Users").
3. Run the installer — defaults are fine. It includes Node.js, npm, and npx automatically.

To verify the install, open a terminal (Command Prompt on Windows, Terminal on Mac) and run:

```
node --version
```

You should see a version number like `v20.11.0`.

---

## Step 2 — Add the Puppeteer MCP Server

Puppeteer is what allows FitFoundry to control a browser to scrape job listings. It is registered in Claude Desktop as an MCP server.

1. Open your Claude Desktop configuration file in a text editor:
   - **Windows:** `C:\Users\<YourName>\AppData\Roaming\Claude\claude_desktop_config.json`
   - **Mac:** `~/Library/Application Support/Claude/claude_desktop_config.json`

   The `AppData` folder on Windows is hidden. Paste the path directly into the File Explorer address bar and press Enter.

2. Add the Puppeteer entry to the `"mcpServers"` object. If the file is empty, use the full structure below. If other servers are already listed, add just the `"puppeteer"` block alongside them — do not replace existing entries.

   ```json
   {
     "mcpServers": {
       "puppeteer": {
         "command": "npx",
         "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
       }
     }
   }
   ```

3. Save the file. Puppeteer will download and run automatically on first use — no separate install step is needed.

---

## Step 3 — Connect the Indeed Connector

1. Open Claude Desktop and click the **gear icon** in the bottom-left corner to open Settings.
2. Click **Connectors** in the left sidebar.
3. Find **Indeed** and click **Connect**.
4. A browser window will open — log in to your Indeed account. Complete any two-factor authentication if prompted.
5. Return to Claude Desktop. Indeed should show as Connected.

---

## Step 4 — Connect the Dice Connector

1. Still in Settings → Connectors, find **Dice** and click **Connect**.
2. Log in to your Dice account in the browser window that opens.
3. Return to Claude Desktop — Dice should show as Connected.

If you do not have a Dice account or do not plan to search Dice, skip this step.

---

## Step 5 — Install Claude in Chrome (Optional)

Required only if you plan to search LinkedIn or Wellfound. Both boards require an authenticated browser session that Puppeteer cannot access due to login walls and CAPTCHA.

1. Open **Google Chrome** (not Edge, Firefox, or other browsers).
2. Visit [claude.ai/chrome](https://claude.ai/chrome) or search for "Claude in Chrome" in the Chrome Web Store.
3. Click **Add to Chrome** and confirm the installation.
4. Click the **puzzle piece icon** in Chrome's toolbar, then click the **thumbtack** next to Claude to pin it.
5. Click the Claude icon to open the side panel and sign in with your Claude account.

Before running a LinkedIn or Wellfound search, make sure you are already logged into those sites in Chrome. Claude in Chrome will ask you to click **Connect** when a search begins.

---

## Step 6 — Restart Claude Desktop

Restart Claude Desktop for the Puppeteer MCP server and connectors to become fully active.

- **Windows:** Right-click the Claude icon in the system tray (bottom-right corner of the taskbar — click the ^ arrow if the icon is hidden) → **Quit**. Reopen from the Start menu or your desktop shortcut.
- **Mac:** Click the Claude icon in the menu bar (top-right of the screen) → **Quit Claude**. Reopen from Applications or your Dock.

After restarting, open Settings → Connectors and confirm Indeed and Dice show as Connected.

---

## Troubleshooting

**The Settings gear icon or Connectors section is not where described**

Claude Desktop's interface changes between versions. Look for a gear or cog icon anywhere on the left sidebar, or check the application menu at the top of the screen under Preferences or Settings. Once in Settings, look for a section called Connectors, MCP Servers, or Developer.

**A connector shows as disconnected after restart**

Open Settings → Connectors and click Connect next to the affected connector. Log in again. A second restart is not needed — the connector activates in the current session.

**Puppeteer is not listed under MCP Servers**

Check that the `claude_desktop_config.json` file was saved correctly and that the JSON is valid (no missing commas or brackets). Then restart Claude Desktop again.

---

## Next Step

Once Claude Desktop has restarted and connectors are confirmed, proceed to [SETUP-STAGE2.md](SETUP-STAGE2.md) to set up your workspace, build your career profile, and run your first search.
