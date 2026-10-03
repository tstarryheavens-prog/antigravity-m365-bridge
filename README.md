# Antigravity M365 Bridge ☁️

> **Break through macOS Files-On-Demand barriers.** Connect Microsoft OneDrive (unsynced cloud files) and Microsoft OneNote to Google Antigravity & LLM agents via Microsoft Graph API.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![MCP Compatible](https://img.shields.io/badge/MCP-Compatible-emerald.svg)](https://modelcontextprotocol.io/)
[![Astro Portal](https://img.shields.io/badge/Live%20Portal-Smart%20AI%20Life-sky.svg)](https://ai-living-lab.pages.dev/m365-cloud-integration)
[![Blog Guide](https://img.shields.io/badge/Blog-Hatena%20Diary-orange.svg)](https://kurashi-ai.hatenadiary.com/entry/2026/10/04/021338)

---

## 💡 The Problem: macOS Files-On-Demand & OneNote API Limits

When pairing local AI agents (such as Google Antigravity, Claude Code, Cursor) with your local filesystem on macOS:

1. **APFS Dataless Files (OneDrive Files-On-Demand):**
   OneDrive for Mac keeps unsynced cloud files as dataless placeholders (`0-byte` stubs) to preserve SSD space. Standard CLI search tools (`grep`, `find`) and agent file viewers cannot index or read file contents without manually downloading each file first.
2. **OneNote for Mac Limitations:**
   OneNote on macOS stores notes in encrypted caches without rich AppleScript/Apple Events DOM query support. AI agents cannot parse notebook sections or page contents locally.

### The Solution
Connect the agent directly to the **Microsoft Graph API** using the lightweight `@softeria/ms-365-mcp-server` configured with the `--preset files,onenote` filter (reduced from 300+ tools down to 34 essential tools to protect token context).

---

## 🚀 Key Features

- **Cloud-Native Full-Text Search:** Search all OneDrive files—including files only in the cloud—using Microsoft Graph cloud search indexing.
- **OneNote Hierarchy Exploration:** Traverse all notebooks, sections, and pages, retrieving rich content in Markdown/HTML.
- **Token Economy (34 Tools):** Surface only the tools needed for files and notes, eliminating hundreds of unused corporate tools (Teams, Planner, etc.).
- **Encrypted Credential Cache:** MSAL access & refresh tokens are encrypted and cached in `~/Library/Application Support/ms-365-mcp-server`.
- **Personal Account Fix:** Bypasses the Microsoft MSAL `invalid_grant` bug on personal Microsoft accounts (`@hotmail.com`, `@outlook.jp`) by enforcing `MS365_MCP_TENANT_ID=consumers`.

---

## 🛠️ Setup Guide (Step-by-Step)

### Step 1: Register an App in Microsoft Entra ID
1. Navigate to the [Microsoft Entra Admin Center (App registrations)](https://entra.microsoft.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade).
2. Click **New registration**:
   - **Name:** `Antigravity-M365-Bridge` (or any custom name)
   - **Supported account types:** Select *Accounts in any organizational directory and personal Microsoft accounts*.
3. Note your **Application (client) ID**.
4. Go to **Manage > Authentication**:
   - Click **Add a platform** > **Mobile and desktop applications**.
   - Check `https://login.microsoftonline.com/common/oauth2/nativeclient` and `http://localhost`.
   - Under **Advanced settings**, set **Allow public client flows** to **Yes**.
   - Click **Save**.
5. Go to **Manage > API permissions**:
   - Click **Add a permission** > **Microsoft Graph** > **Delegated permissions**.
   - Add:
     - `Files.ReadWrite`
     - `Notes.Create`
     - `Notes.Read`
     - `Notes.ReadWrite`

---

### Step 2: Configure Antigravity / MCP Client

Add the following to your `mcp_config.json` (e.g. `~/.gemini/config/mcp_config.json`):

```json
{
  "mcpServers": {
    "ms-365": {
      "command": "/opt/homebrew/bin/npx",
      "args": [
        "-y",
        "@softeria/ms-365-mcp-server",
        "--preset",
        "files,onenote"
      ],
      "env": {
        "MS365_MCP_CLIENT_ID": "YOUR_ENTRA_CLIENT_ID",
        "MS365_MCP_TENANT_ID": "consumers"
      }
    }
  }
}
```

> **Important:** Set `MS365_MCP_TENANT_ID` to `"consumers"` if you are using a personal Microsoft account. Omitting this causes an `invalid_grant` error during MSAL token renewal.

---

### Step 3: Initial Authentication (Device Code Flow)

Run the one-time interactive login in your terminal:

```bash
MS365_MCP_CLIENT_ID="YOUR_ENTRA_CLIENT_ID" \
MS365_MCP_TENANT_ID="consumers" \
npx -y @softeria/ms-365-mcp-server --preset files,onenote --login
```

Follow the prompt:
1. Open [https://www.microsoft.com/link](https://www.microsoft.com/link) in your browser.
2. Enter the 8-character code displayed in the terminal.
3. Sign in and grant permission.

Your credentials will be securely cached in your OS Application Support folder. Subsequent calls will automatically refresh tokens in the background.

---

## 🔍 Verified Working Tools

### OneDrive
- `search-onedrive-files`: Search files across your entire personal cloud storage by query keyword.
- `list-folder-files`: List directory contents directly from the cloud.
- `get-drive-item`: Retrieve file metadata.
- `download-bytes`: Download raw binary files on-demand.

### OneNote
- `list-onenote-notebooks`: List all personal and shared OneNote notebooks.
- `list-onenote-notebook-sections`: List sections inside a target notebook.
- `list-onenote-section-pages`: List pages in a section.
- `get-onenote-page-content`: Read page content for summarization or research.

---

## 🗺️ Knowledge Ecosystem Integration

| Service | MCP Provider | Capabilities |
| :--- | :--- | :--- |
| **Google Cloud** | `google-multi` | Multi-account Gmail, Google Drive, Google Calendar |
| **Microsoft 365** | `ms-365` (this repo) | Cloud OneDrive search & OneNote notebook extraction |
| **Local macOS Office** | `excel-live` & `ppt-mcp` | Direct Apple Events Excel manipulation & PowerPoint generation |

---

## 📄 License

MIT License. Developed for the Smart AI Life Project.
