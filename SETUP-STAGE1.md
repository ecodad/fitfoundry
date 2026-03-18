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
| **Google Chrome** | Required for Claude in Chrome. |

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

   **Windows:** Press **Win+R**, paste the following into the Run box, and click **OK**:
   ```
   %APPDATA%\Claude\claude_desktop_config.json
   ```
   This opens the file directly in your default text editor. If Windows asks you to choose an app, select **Notepad**.

   **Mac:** Press **Cmd+Space**, type **Terminal**, press Enter, then run:
   ```
   open -e ~/Library/Application\ Support/Claude/claude_desktop_config.json
   ```
   This opens the file in TextEdit.

   > **Note:** The "Add custom connector" button in Claude Desktop settings is for remote MCP servers only and cannot be used for Puppeteer. The config file edit is the only way to add a local MCP server.

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

1. Open Cowork and click **Customize** in the left sidebar.
2. Click **Connectors**.
3. Click the **+** button at the top right of the Connectors panel.
4. Search for **Indeed** and select it.
5. A browser window will open — log in to your Indeed account. Complete any two-factor authentication if prompted.
6. Return to Cowork. Indeed should appear under **Web** and show as connected.

---

## Step 4 — Connect the Dice Connector

1. Still in Customize → Connectors, click the **+** button again.
2. Search for **Dice** and select it.
3. Log in to your Dice account in the browser window that opens.
4. Return to Cowork — Dice should appear under **Web** and show as connected.

Dice is a tech-focused job board specialising in software, engineering, and IT roles — worth including if your background is technical. If you do not have a Dice account or do not plan to search it, skip this step.

---

## Step 5 — Install Claude in Chrome

Claude in Chrome is used by FitFoundry in three situations: ghost job checks on company career sites that block Puppeteer, fetching full job descriptions from boards that are Cloudflare-protected (such as ClimateBase detail pages), and searching login-gated boards like LinkedIn and Wellfound. Most job search runs will use it at some point.

1. Open **Google Chrome** (not Edge, Firefox, or other browsers).
2. Visit [claude.ai/chrome](https://claude.ai/chrome) or search for "Claude in Chrome" in the Chrome Web Store.
3. Click **Add to Chrome** and confirm the installation.
4. Click the **puzzle piece icon** in Chrome's toolbar, then click the **thumbtack** next to Claude to pin it.
5. Click the Claude icon to open the side panel and sign in with your Claude account.

Claude in Chrome will prompt you to click **Connect** the first time it is used in a session. Make sure Chrome is open and you may be prompted to log into certain sites during a run that uses them.

---

## Step 6 — Restart Claude Desktop

Restart Claude Desktop for the Puppeteer MCP server to become active.

- **Windows:** Right-click the Claude icon in the system tray (bottom-right corner of the taskbar — click the **^** arrow if the icon is hidden) → **Quit**. Reopen from the Start menu or your desktop shortcut.
- **Mac:** Click the Claude icon in the menu bar (top-right of the screen) → **Quit Claude**. Reopen from Applications or your Dock.

After restarting, open Cowork → Customize → Connectors and confirm Indeed and Dice appear under **Web**. Puppeteer will appear under **Desktop** — it downloads and runs automatically the first time FitFoundry uses it, so you will not see it listed until then.

---

## Troubleshooting

**A connector shows as disconnected after restart**

In Cowork, go to Customize → Connectors, click the connector, and reconnect it. Log in again if prompted. A second restart is not needed — the connector activates in the current session.

**Puppeteer is not listed under Desktop connectors after a run**

Check that `claude_desktop_config.json` was saved correctly and that the JSON is valid (no missing commas or brackets). A common mistake is opening the file in Word or a rich-text editor that adds invisible formatting — use Notepad (Windows) or TextEdit in plain text mode (Mac). Restart Claude Desktop after fixing the file.

**Indeed or Dice connector is connected but returns no results**

The connectors must be active when the Cowork session starts — connecting mid-session does not inject the tools. Start a new Cowork session after confirming the connectors are connected.

---

## Next Step

Once Claude Desktop has restarted and connectors are confirmed, proceed to [SETUP-STAGE2.md](SETUP-STAGE2.md) to set up your workspace, build your career profile, and run your first search.
