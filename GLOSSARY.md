# Glossary

Plain-language definitions for terms used throughout the FitFoundry documentation. If you are new to AI tools or developer concepts, start here.

---

**Agent**
An AI that does things, not just answers questions. A regular AI conversation responds to what you type. An agent reads files, opens websites, runs code, and takes sequences of actions on your behalf to complete a task. FitFoundry runs as an agent — you tell it to search a job board, and it opens the board, extracts listings, scores them, and writes the results without you doing anything in between.

---

**Chat**
The standard way most people use Claude — a conversation in a browser tab or the Claude mobile app. You type a message and Claude responds. Chat does not have access to your computer or files. FitFoundry does not run in Chat.

---

**Code**
Short for Claude Code — a developer tool that lets programmers run Claude from a command line terminal. Not required for FitFoundry. Mentioned here because it sometimes comes up when searching for Claude-related help online.

---

**Connector**
A pre-built integration that gives Claude access to an external service. Connectors are installed through Customize → Connectors in Claude Desktop. FitFoundry uses the Indeed and Dice connectors to search those job boards through their official access, rather than scraping them through a browser.

---

**Cowork**
The mode of Claude Desktop that lets Claude act as an agent on your computer. Unlike Chat, Cowork can read and write files, run code, control a browser, and complete multi-step tasks. FitFoundry runs inside a Cowork session.

---

**GitHub**
A website where developers store and share code projects. FitFoundry is hosted on GitHub at github.com/ecodad/fitfoundry. You do not need a GitHub account to download and use FitFoundry — the Releases page is publicly accessible.

---

**Markdown**
A simple way of formatting plain text files. Headings start with `#`, bold text is wrapped in `**asterisks**`, and lists use `-` or `1.`. All FitFoundry's documentation and output files are written in Markdown. Most text editors and GitHub display Markdown in a readable formatted view automatically.

---

**MCP (Model Context Protocol)**
A standard that lets Claude connect to external tools and services. Think of it as a plug format — any tool that speaks MCP can be plugged into Claude. Puppeteer is added to FitFoundry as an MCP server; Indeed and Dice connect as MCP-based connectors. You do not need to understand MCP to use FitFoundry — the setup steps handle everything.

---

**Mounted directory (workspace folder)**
The folder on your computer that Cowork is given access to at the start of a session. When Cowork opens, it runs inside a small isolated environment on your machine. Mounting a directory creates a direct connection between that environment and a real folder on your hard drive, so files Claude writes there appear in your normal file system. In FitFoundry, this is your job search folder — where your profile, results, and resumes live.

---

**Node.js**
A program that lets JavaScript code run on your computer, outside of a browser. Puppeteer is written in JavaScript and requires Node.js to run. You install it once from nodejs.org and do not need to interact with it again — it works silently in the background.

---

**npx**
A command-line tool that comes bundled with Node.js. It runs JavaScript packages without requiring you to install them permanently. When Claude starts the Puppeteer MCP server, it uses `npx` to launch it automatically. You do not run `npx` yourself — it is called behind the scenes.

---

**Prompt**
The instruction or question you give to Claude. In FitFoundry, your prompt is how you tell Claude which board to search, what kind of roles to look for, and what constraints to apply. A well-formed prompt gives Claude enough context to run an entire search session without needing to stop and ask clarifying questions.

---

**Repository (repo)**
A project hosted on GitHub containing all its files and their full history of changes. The FitFoundry repository contains the skill files, documentation, setup guides, and release assets. When someone says "check the repo" they mean visiting the project on GitHub.

---

**Skill**
A packaged set of instructions that teaches Claude how to run a specific workflow. Skills are installed into Claude Desktop and loaded automatically when relevant. FitFoundry is distributed as a `.skill` file — a single file you upload through Customize → Skills. Once installed, Claude knows how to run the full FitFoundry workflow without you pasting in instructions manually.
