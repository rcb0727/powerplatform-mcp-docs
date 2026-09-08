# Installation Guide

**Docs:** [Overview](https://github.com/rcb0727/powerplatform-mcp-docs/blob/main/README.md) · **Installation & Upgrading** · [Changelog](https://github.com/rcb0727/powerplatform-mcp-docs/blob/main/CHANGELOG.md) · [Report an issue](https://github.com/rcb0727/powerplatform-mcp-docs/issues)

This guide gets you from zero to "ask your AI app to build a flow." Pick the path that sounds like you — you don't need to read the whole page.

## Choose your path

For large tool catalogs, see [Token-efficient client setup](docs/TOKEN-EFFICIENCY.md).
All tools remain available; supported clients discover full schemas on demand.

| You are… | Go to |
|----------|-------|
| 🟢 **Not very technical** — you just want it working | [Easy Path](#easy-path-3-steps) |
| 🔵 **A developer** — comfortable in a terminal | [Fast Path](#fast-path-developers) |
| 🟣 **An IT admin** — setting this up for yourself or your org | [Admin & enterprise setup](#admin--enterprise-setup) |
| ❓ Stuck on something | [Troubleshooting](#troubleshooting) · [Glossary](#glossary) |

New to any of this? The [Glossary](#glossary) explains MCP, app registration, tenant, and consent in one line each.

---

## Before you start

**This takes about 10 minutes.** One step (admin consent) may need a quick approval from your IT department — skim [Step 2](#step-2--run-setup) before you begin so nothing catches you off guard.

You need three things. The Easy Path checks all of them for you, but here's the full list:

1. **A Microsoft 365 *work* account** with Power Automate access — the same email and password you use for Outlook, Teams, and Office at work. (Not a personal @outlook.com / @gmail account. You don't need to create anything new.)
2. **An AI app that supports MCP** — Claude Desktop, Claude Code, Cursor, VS Code (Copilot), Gemini CLI, Windsurf, or ChatGPT. If you have one open right now, you're set.
3. **Node.js 22.19 or newer** — the engine the server runs on. Install steps below.

### Step A — Open a terminal

The terminal is the app where you type the commands in this guide.

- **Windows:** click **Start**, type **PowerShell**, press Enter.
- **macOS:** press **Cmd + Space**, type **Terminal**, press Enter.
- **Linux:** press **Ctrl + Alt + T**, or search **Terminal** in your apps.

To run any command below: click in the terminal, **paste** it, press **Enter**, and wait for it to finish before the next one.

### Step B — Install Node.js (one-time)

**First check if you already have it.** In the terminal, paste this and press Enter:

```bash
node --version
```

- If it prints **`v22.19.0` or newer** (for example `v22.19.x`, `v24.x`, or later), you're done — go to [Easy Path Step 1](#step-1--install-the-server).
- If it says **"command not found"**, `v18`, `v20`, or a `v22` version below `v22.19.0`, install Node:
  - **Windows & macOS:** go to [nodejs.org](https://nodejs.org) and click the big green button labeled **LTS** (the "Recommended for most users" one — *not* "Current"). Run the installer and click through the defaults.
  - **macOS, only if you already use Homebrew:** `brew install node`
  - **Linux:** [nodejs.org](https://nodejs.org), your distro's package manager, or [nvm](https://github.com/nvm-sh/nvm).

  Then close and reopen the terminal and run `node --version` again to confirm.

> **Linux only — secure password storage:** install libsecret so your sign-in token is stored safely. (Windows and macOS users: skip this.)
> ```bash
> sudo apt-get install libsecret-1-0 gnome-keyring   # Ubuntu/Debian
> sudo dnf install libsecret gnome-keyring           # Fedora/RHEL
> ```
> If setup later says `libsecret-1.so.0: cannot open shared object file`, this package is missing.

---

## Easy Path (3 steps)

In your terminal, paste each command, press **Enter**, and let it finish before the next one. When the tool *asks* you something, type your answer (or the number next to your choice) and press Enter.

### Step 1 — Install the server

```bash
npm install -g powerautomate-mcp
```

✅ **You should see** a few lines ending in something like `added 1 package`. Yellow `WARN` lines are normal and safe to ignore — only red `ERR!` means a real problem.

> **npm 12 or newer?** npm now blocks dependency install scripts by default, and you'll see a warning naming `keytar` and `@azure/msal-node-extensions`. **You don't have to do anything** — setup detects this and repairs it automatically. To skip the repair step entirely, install with the scripts allowed:
> ```bash
> npm install -g --allow-scripts=@azure/msal-node-extensions,keytar powerautomate-mcp
> ```

> **If that fails** with `command not found`, `EACCES`, or a permission error, don't worry — you don't need the global install. Just use `npx` instead: everywhere below, replace `powerautomate-mcp` with `npx -y powerautomate-mcp@latest`. So Step 2 becomes `npx -y powerautomate-mcp@latest --setup`. That's the only change — skip the rest of this step.

### Step 2 — Run setup

```bash
powerautomate-mcp --setup
```

A wizard starts. It works out your situation first and only asks a question when it genuinely can't know the answer:

> **Already deployed at your organization?** If IT pushed the org file to your machine, there may be nothing to set up at all — your AI app's first request shows a sign-in code in the chat, and that's your entire onboarding. Running `--setup` still works and skips straight to sign-in.

The five steps:

1. **Find your app** — the wizard looks for an existing setup automatically: a saved config, your organization's `org.json`, or an app it can see through an already-signed-in Azure CLI. Found → nothing to type. Not found → one question: **paste the Client ID** your IT team gave you (it's checked against Microsoft on the spot, so a typo fails in seconds), or **press Enter to create a new app registration**. The create path is for IT or the first person at an org: it picks a permission preset (**Everyday automation** recommended — press Enter to accept), signs into Azure, creates the app with only those permissions, and grants org-wide admin consent when your account is allowed to.
2. **Sign in** — a device code is shown; open the link on any device and log in with your **work** account. Admin consent is verified by *doing*: only if Microsoft reports the app isn't approved yet do you see the approval link (or a ready-to-send IT request). Approved apps never show a consent step.
3. **Pick your environment** — if you only have one (most people), it's selected automatically. Otherwise type the number next to the one you want (the recommended one is marked ⭐).
4. **Save** — it writes your settings automatically, including which feature scopes are enabled.
5. **Connect your AI app** — the wizard looks for AI apps installed on your machine. If it finds exactly one, it's a single yes/no; otherwise detected apps are listed first (marked "detected") — type a number. It wires itself up — no JSON editing.

Setup ends by handing IT the rollout: `powerautomate-mcp --emit-org-config` generates the org file that makes every other machine's setup automatic — see [Mass deployment](#mass-deployment-intune--gpo--jamf).

Setup finishes by **verifying everything end to end** — config, persisted sign-in, and a live Power Automate call — before showing the banner.

✅ **You should see** `Verified — config valid, sign-in persisted, Power Automate reachable` and a green **Setup Complete!** banner.

> Want to skip the app menu? Name it on the command: `powerautomate-mcp --setup --client claude` (or `cursor`, `vscode`, `gemini`, `claude-code`, `codex`, `windsurf`). Either way works — without `--client`, the wizard detects what's installed.

#### Admin consent: what to do

Approval ("admin consent") is a one-time, organization-wide action — and the wizard never quizzes you about it up front. It simply attempts sign-in; only if Microsoft reports the app isn't approved does the approval link appear. Admins: open the link, approve, press Enter to retry. Everyone else: type `q` and the wizard prints a ready-to-send message for your IT helpdesk with the link included — your progress is saved, and re-running `powerautomate-mcp --setup` picks up where you left off.

(Only a Global, Application, Cloud Application, or Privileged Role admin can approve — IT will know who that is.)

#### Getting a Client ID from IT

If the wizard finds nothing and you don't have a **Client ID**, type `q` at the prompt — it prints a ready-to-send message for your IT department covering everything they need to do (create the app registration, send you the Client ID, grant consent), with a pointer to [Admin & enterprise setup](#admin--enterprise-setup). Once IT sends you the ID, run `powerautomate-mcp --setup` again and paste it — the wizard verifies the ID against Microsoft immediately, so a typo fails on the spot instead of at the sign-in screen.

### Step 3 — Confirm it works

```bash
powerautomate-mcp --doctor
```

✅ **You should see** green checks for Node, version, config, sign-in, "Power Platform reachable," and your AI app "connected." If anything is red, `--doctor` tells you the exact fix.

**Then fully restart your AI app** so it loads the new tools:
- **macOS:** closing the window isn't enough — press **Cmd + Q** with the app focused (or right-click its Dock icon → **Quit**), then reopen it.
- **Windows/Linux:** close the app completely and reopen it.

Now just ask it:

> *"List my Power Automate flows."*

If it lists your flows — or says you don't have any yet — it's working. 🎉 (If it says it has no Power Automate tool, redo the restart above.)

---

## Fast Path (developers)

```bash
npm install -g powerautomate-mcp          # or: npx -y powerautomate-mcp@latest --setup
powerautomate-mcp --setup --client cursor # runs the wizard + writes your client config
powerautomate-mcp --doctor                # verify config + auth + connectivity + wiring
```

`--client` accepts: `claude`, `claude-code`, `cursor`, `vscode`, `gemini`, `windsurf`. Add `--npx` to write a config that runs the server via `npx -y powerautomate-mcp@latest` (no global install). Prefer to wire the client yourself? See [manual client configs](#connect-your-ai-app-manually).

**No global install at all:** point your client at `npx` (see the [npx option](#option-b-no-global-install-npx)) and run setup with `npx -y powerautomate-mcp@latest --setup`.

---

## Connecting your AI app

`--setup` connects your app for you. You only need this section if you skipped that step, use a second app, or want to do it by hand.

### Auto-connect (recommended)

Run any time — no re-auth needed, it only writes the client's config file:

```bash
powerautomate-mcp --client claude     # Claude Desktop
powerautomate-mcp --client cursor     # Cursor
powerautomate-mcp --client vscode     # VS Code (Copilot)
powerautomate-mcp --client gemini     # Gemini CLI
powerautomate-mcp --client claude-code # Claude Code (uses `claude mcp add`)
powerautomate-mcp --client windsurf   # Windsurf
```

It **merges** into any existing config — your other MCP servers are preserved. Add `--npx` for the no-global-install form. Restart the app afterward.

### Connect your AI app manually

Prefer to edit the files yourself? Use these.

<details>
<summary><strong>Claude Desktop</strong></summary>

Edit the config file (create it if missing):

| OS | Path |
|----|------|
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |
| Linux | `~/.config/Claude/claude_desktop_config.json` |

```json
{
  "mcpServers": {
    "powerautomate": {
      "command": "powerautomate-mcp"
    }
  }
}
```

If the file already has content, add the `powerautomate` block **inside** the existing `mcpServers` object — don't create a second one. Restart Claude Desktop.
</details>

<details>
<summary><strong>Claude Code (CLI)</strong></summary>

```bash
claude mcp add powerautomate -- powerautomate-mcp
```

Or add to your project's `.mcp.json`:

```json
{
  "mcpServers": {
    "powerautomate": { "command": "powerautomate-mcp" }
  }
}
```
</details>

<details>
<summary><strong>VS Code (GitHub Copilot)</strong></summary>

Open the Command Palette (`Ctrl/Cmd+Shift+P`) → **MCP: Open User Configuration**, or edit the file directly:

| OS | Path |
|----|------|
| macOS | `~/Library/Application Support/Code/User/mcp.json` |
| Windows | `%APPDATA%\Code\User\mcp.json` |
| Linux | `~/.config/Code/User/mcp.json` |

```json
{
  "servers": {
    "powerautomate": {
      "type": "stdio",
      "command": "powerautomate-mcp"
    }
  }
}
```

Note VS Code uses `servers` (not `mcpServers`) and needs `"type": "stdio"`.
</details>

<details>
<summary><strong>Cursor</strong></summary>

Edit `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (project):

```json
{
  "mcpServers": {
    "powerautomate": { "command": "powerautomate-mcp" }
  }
}
```

Restart Cursor.
</details>

<details>
<summary><strong>Google Gemini CLI</strong></summary>

Edit `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "powerautomate": { "command": "powerautomate-mcp" }
  }
}
```
</details>

<details>
<summary><strong>ChatGPT (advanced — needs a public HTTPS URL)</strong></summary>

ChatGPT can only reach a remote HTTPS MCP endpoint, so you expose the local server through a tunnel.

**1. Start in HTTP mode — with an access token:**
```bash
PA_MCP_HTTP_TOKEN="pick-a-long-random-secret" powerautomate-mcp --http --port 3000
```
Serves Streamable HTTP at `http://localhost:3000/mcp`.

**Why the token matters:** in the normal (stdio) mode there is nothing to protect — your AI app talks to the server through a private pipe on your machine. In `--http` mode the server is a web service, and **anyone who can reach the port can run every tool with your signed-in Power Platform account** — create, share, and delete flows as you. `PA_MCP_HTTP_TOKEN` locks the door: requests must send a matching `Authorization: Bearer <token>` header or they're rejected with 401. The server only listens on `127.0.0.1` (your own machine) by default — but the moment you tunnel it to the internet (step 2), the token is the *only* thing standing between the public URL and your tenant. Never expose the server without one.

(Windows PowerShell: `$env:PA_MCP_HTTP_TOKEN="pick-a-long-random-secret"; powerautomate-mcp --http --port 3000`)

**2. Expose it (pick one):**
```bash
ngrok http 3000
# or
cloudflared tunnel --url http://localhost:3000
```

**3. In ChatGPT:** Settings → MCP Servers → **Add Server** → enter `https://your-tunnel-url/mcp`, set the **Authorization / Bearer token** field to the same secret from step 1 → Save.

> **Security:** the tunnel exposes your local server to the internet. Always set `PA_MCP_HTTP_TOKEN` before tunneling, only run it while using ChatGPT, and stop the server and tunnel when done.
</details>

### Option B: no global install (npx)

If `npm install -g` causes permission headaches, skip it entirely. Configure your client to launch the server through `npx`, which downloads it on demand:

```json
{
  "mcpServers": {
    "powerautomate": {
      "command": "npx",
      "args": ["-y", "powerautomate-mcp@latest"]
    }
  }
}
```

`powerautomate-mcp --client <name> --npx` writes exactly this for you. Run setup the same way: `npx -y powerautomate-mcp@latest --setup`. Trade-off: the first launch each session is a little slower while npx fetches the package.

---

## Updating

> **Quit your AI apps first.** A running server keeps the old version loaded, and on Windows it can lock files in npm's folder and cause a half-finished, confusing upgrade.

1. Quit your AI apps and stop any `--http` servers.
2. Upgrade:
   ```bash
   npm install -g powerautomate-mcp@latest
   ```
3. Confirm:
   ```bash
   powerautomate-mcp --doctor
   ```
4. Reopen your AI app — it picks up the new version automatically.

`powerautomate-mcp --update` does the npm upgrade for you (same rule: close apps first). The [Changelog](https://github.com/rcb0727/powerplatform-mcp-docs/blob/main/CHANGELOG.md) lists what changed and any version-specific notes.

You don't have to check for updates yourself: when a new version is out, the server mentions it once per session right in the chat (set `PA_MCP_UPDATE_NOTICE=0` to turn that off). Machines set up through an organization's `org.json` stay quiet — updates there are IT's job.

---

## Rolling back

Every published version stays on npm permanently, so if an update misbehaves you can return to the version you were on with one command. Same rule as updating: quit your AI apps first.

1. Pick the version: the [Changelog](https://github.com/rcb0727/powerplatform-mcp-docs/blob/main/CHANGELOG.md) lists every release and what changed, or run `npm view powerautomate-mcp versions`.
2. Install it by number:
   ```bash
   npm install -g powerautomate-mcp@0.16.4
   ```
3. Confirm, then reopen your AI app:
   ```bash
   powerautomate-mcp --doctor
   ```

A rollback changes only the installed package — your configuration and sign-in are untouched, and there's nothing else to undo (no service, no database, no migrations). When the issue is resolved, `npm install -g powerautomate-mcp@latest` moves you forward again.

**Organizations:** deployment scripts can pin an exact version (`@0.16.4` instead of `@latest`) so every machine runs the version IT approved — rolling back then means pushing the deployment again with the prior pin. If a release forced you to roll back, please [report it](https://github.com/rcb0727/powerplatform-mcp-docs/issues) so it gets fixed for everyone.

---

## Signing in again

Sign-ins don't last forever — tokens expire after long inactivity, and IT policy changes (like new MFA rules) can invalidate them. When tools start reporting missing credentials, you don't need to re-run setup:

- **In chat:** ask your AI assistant to *"sign in to Power Platform"* — the `sign_in` tool returns a code and link right in the conversation.
- **In a terminal:** `powerautomate-mcp --login` — signs you in again using your existing setup, then verifies it worked.

Both use Microsoft's standard sign-in page in your own browser, MFA included. Nothing ever asks for your password outside microsoft.com.

---

## Troubleshooting

Run **`powerautomate-mcp --doctor`** first — it pinpoints most problems and prints the fix. Common cases:

### Sign-in works, then everything says "not authorized" (npm 12+)

npm 12 blocks dependency install scripts by default; the blocked `keytar` script leaves secure token storage without a native component, and the failure masquerades as permission/consent problems (sign-in appears fine, every later step finds no credentials). Run `powerautomate-mcp --setup` or `--doctor` — both detect and repair this automatically. Manual alternative: reinstall with `npm install -g --allow-scripts=@azure/msal-node-extensions,keytar powerautomate-mcp`.

### macOS keeps asking for your keychain password

macOS prompts whenever the token cache is read by a program that isn't on the keychain item's approval list — and Node upgrades change the program's identity, so an old approval stops counting. Two fixes, use both:

1. When the prompt appears, click **Always Allow** (not "Allow").
2. If prompts persist or come back after a Node upgrade, run:
   ```bash
   powerautomate-mcp --fix-keychain
   ```
   It asks for **one** final approval, then rebuilds the keychain entry so the current Node owns it — reads and writes stop prompting entirely. Rerun it if prompts ever return.

(The server also avoids re-reading the keychain unless your sign-in actually changed, so even unapproved setups see at most a prompt per session — never one per tool call. `PA_MCP_TOKEN_CACHE_SYNC=always` restores the old re-read-every-time behavior if you ever need it.)

### Install problems

| Symptom | Fix |
|---------|-----|
| `command not found: powerautomate-mcp` after install | npm's global folder isn't on your PATH. Easiest fix: use the [npx option](#option-b-no-global-install-npx) instead. Or check `npm config get prefix` and add its `bin` folder to PATH. |
| `npm ERR! code EACCES` during `npm install -g` | A permissions issue. **Don't use `sudo`.** Use the [npx option](#option-b-no-global-install-npx), or set npm's prefix to a folder you own, or install Node via [nvm](https://github.com/nvm-sh/nvm). |
| `EBUSY` / `EPERM` on Windows during upgrade | An app is still running the server. Quit all AI apps and `--http` servers, then upgrade again. |
| `npm warn deprecated prebuild-install@…: No longer maintained` during install | **Harmless — ignore it.** `prebuild-install` is an install-time helper that downloads prebuilt native modules; it comes from two upstream dependencies (the SQLite cache library and Microsoft's authentication library) and never runs as part of the server. Its author retired the package, so npm warns on every install until those upstreams migrate off it. Nothing on your machine is broken or insecure. |
| `Node.js … (need 22.19+)` from `--doctor` | Upgrade Node from [nodejs.org](https://nodejs.org) (install the LTS). |
| Linux: `libsecret-1.so.0: cannot open shared object file` | Install libsecret — see the Linux note under [Before you start](#before-you-start). |

### Setup & sign-in problems

| Symptom | Fix |
|---------|-----|
| `AADSTS65001` / "admin consent not granted" | An admin must approve the consent URL the wizard shows. Send it to your Global/Application/Cloud-App/Privileged-Role admin, then re-run `--setup`. |
| "No environments found" | The account you signed in with has no Power Automate access — sign in with your work account, or ask IT to grant access. |
| Wizard can't create an app registration | You're not an Entra admin and don't have Azure CLI. Ask an admin for a **Client ID** and paste it when prompted (or set `PA_MCP_CLIENT_ID`). See [Admin & enterprise setup](#admin--enterprise-setup). |
| Some tools fail with `AADSTS65001` after setup | A permission for that feature wasn't consented. Run `powerautomate-mcp --doctor` to see the exact API, then see [enterprise permissions](#admin--enterprise-setup). |

### "It installed but my AI app doesn't see the tools"

1. **Fully restart the app** — quit completely (not just close the window) and reopen.
2. Run `powerautomate-mcp --doctor` — it shows whether your app is "connected."
3. If it's not connected, run `powerautomate-mcp --client <your-app>` and restart again.
4. Confirm the command works on its own: `powerautomate-mcp --version` should print a number.

Still stuck? [Open an issue](https://github.com/rcb0727/powerplatform-mcp-docs/issues) — include your OS, AI app, and the output of `powerautomate-mcp --doctor`.

---

## Glossary

| Term | In plain English |
|------|------------------|
| **MCP** (Model Context Protocol) | A standard way for AI apps to use external tools. This server is one such tool. |
| **MCP client / AI app** | The app you chat with (Claude, Cursor, VS Code Copilot, etc.) that runs the tools. |
| **App registration** | An identity in Microsoft Entra that lets this server sign in to your Microsoft account on your behalf. The wizard usually creates it for you. |
| **Tenant** | Your organization's Microsoft 365 directory — basically "your company's Microsoft account." |
| **Admin consent** | A one-time approval from an IT admin that allows the app's permissions across your tenant. |
| **Environment** | A Power Platform workspace that holds your flows, apps, and data. You pick one during setup. |
| **stdio** | How the AI app talks to this server locally (over standard input/output). The default — nothing to configure. |
| **Client ID** | The ID of the app registration — a UUID like `1234abcd-…`. Only needed if you provide your own. |

---

## Admin & enterprise setup

If your tenant **requires admin consent for all applications** (most enterprises do), the per-user wizard can't self-approve. As an admin:

1. **Add only the API permissions for the feature set you selected** in Microsoft Entra. A Power Automate-only setup needs only the Flow Service permissions; add the optional APIs only when users need those tools:
   - Flow Service (`7df0a125-d3be-4c96-aa54-591f83ff541c`): `Flows.Read.All`, `Flows.Manage.All`, `Activity.Read.All`, `Approvals.Manage.All`
   - Optional SharePoint/Excel helpers: Microsoft Graph `User.Read`, `Sites.ReadWrite.All`, `Files.ReadWrite.All`
   - Optional connections/connectors/Power Apps: PowerApps Service (`475226c6-020e-4fb2-8a90-7a972cbfc1d4`) `User`
   - Optional Dataverse/admin/Power Pages config: BAP Admin API (`0e0bf3cc-3078-4fd4-9ef3-cb6dc0245b10`) `user_impersonation`
   - Optional Dataverse/Power Pages config: Dynamics CRM (`00000007-0000-0000-c000-000000000000`) `user_impersonation`
   - Power Platform API (`8578e004-a5c6-46e7-913e-12f58912df43`): `ResourceQuery.Resources.Read` for tenant-wide Copilot inventory; `CopilotStudio.Copilots.Invoke` for authenticated execution; `CopilotStudio.MakerOperations.Read` and `CopilotStudio.MakerOperations.ReadWrite` for evaluations; `CopilotStudio.AdminActions.Invoke` for governance; `PowerPages.Websites.Read` and `PowerPages.Websites.Write` for optional Power Pages site-management tools. Automatic setup resolves the tenant-local scope IDs; manually managed registrations must add the delegated scopes and have an Entra administrator grant consent. Refresh the signed-in session afterward; re-running setup cannot grant admin consent.

2. **Grant admin consent** for the selected permissions:
   ```
   https://login.microsoftonline.com/{tenant-id}/adminconsent?client_id={your-client-id}
   ```

3. Have users run `powerautomate-mcp --setup`. Distribute the Client ID via an `org.json` file (see [Mass deployment](#mass-deployment-intune--gpo--jamf) below — users then type nothing at all) or the `PA_MCP_CLIENT_ID` environment variable.

Skipped feature scopes are recorded in `features.enabled`; their tools are hidden and their auth checks are skipped. Without the PowerApps Service permission, connector and Power Apps tools are unavailable. Without the BAP Admin API permission, admin tools and Dataverse URL auto-discovery are unavailable.

## Mass deployment (Intune / GPO / Jamf)

Rolling out to a whole team means IT does everything once, and every other user's onboarding is a single sign-in from chat — no terminal, no wizard, no Client ID.

**One time, on your own machine (IT):**

1. Run `powerautomate-mcp --setup` and press **Enter** at the app prompt to create the registration — the wizard signs into Azure, creates the app with only the permissions you pick, and grants organization-wide admin consent (your admin role permitting). Pick the environment your team uses.
2. Run `powerautomate-mcp --emit-org-config > org.json`. The file carries the app's Client ID, your tenant, and the chosen environment — everything a fresh machine needs, and nothing secret (identifiers only, no credentials).

**Per machine, through your management tool:**

| Artifact | How |
|----------|-----|
| Runtime | Node.js 22+ plus `npm install -g powerautomate-mcp` (machine-wide script step) |
| `org.json` | Windows: `%ProgramData%\powerautomate-mcp\org.json` · macOS: `/Library/Application Support/powerautomate-mcp/org.json` · Linux: `/etc/powerautomate-mcp/org.json` |
| AI app wiring | Per user: `powerautomate-mcp --client claude` (or `cursor`, `vscode`, …) in a login script — writes the app's MCP config only, needs no sign-in — or push the config file itself |

**Per user: nothing.** With the org file present, the MCP server starts without any per-user setup. The first time they ask their AI app to do something real, it shows a Microsoft sign-in code right in the chat (the `sign_in` tool); they enter it on any device and they're working. Running `powerautomate-mcp --setup` also works — it finds the org file and skips straight to sign-in.

**Before rollout, check Conditional Access.** Sign-in uses Microsoft's device-code flow. If your tenant's Conditional Access policies block device-code grants, allow them for this app first — otherwise every user's sign-in fails identically on day one.

**Expect periodic re-sign-in under sign-in frequency policies.** If your tenant enforces a Conditional Access sign-in frequency, Microsoft expires every user's session on that schedule and no client can renew it silently — that's the policy working, not the tool failing. When it happens, the error names the policy and the fix is one `sign_in` from chat. If the cadence is disruptive, IT can scope the sign-in frequency policy to exclude this app.

**Operations afterward:**

- **Kill switch** — disable the app registration in Microsoft Entra: the entire deployment stops accepting sign-ins instantly.
- **Offboard one user** — disable their account; their tokens die with it.
- **Update** — `npm install -g powerautomate-mcp@latest` via your management tool (or users run `powerautomate-mcp --update`). With `org.json` present, users are NOT nagged about new versions in chat (updates are yours to roll out); set `PA_MCP_UPDATE_NOTICE=1` if you want them notified anyway.
- **Change environments** — push an updated `org.json`.
- **Custom file location** — set `PA_MCP_ORG_CONFIG` to the file's path.
