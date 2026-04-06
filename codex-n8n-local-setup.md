# Codex + n8n Installation Guide ( Windows )

This is a guide to setup n8n mcp ( both [community MCP](https://github.com/czlonkowski/n8n-mcp/) and official MCP ) with Codex on Windows.


This gets you:

- local `n8n` in Docker
- `n8n_docs` in Codex for node search and validation ( this is the [community MCP](https://github.com/czlonkowski/n8n-mcp/) )
- `n8n_live` in Codex for your real local instance ( this is the official MCP )
- an optional public tunnel for webhook testing
- a clean path to Hostinger later

## 1. Install These

Install all three first:

1. Docker Desktop: [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
2. Node.js LTS: [https://nodejs.org/en/download](https://nodejs.org/en/download)
3. Codex app: [https://openai.com/index/introducing-the-codex-app/](https://openai.com/index/introducing-the-codex-app/)

Verify in PowerShell:

```powershell
docker --version
node -v
npm -v
```

## 2. Run n8n Locally

Create the Docker volume in PowerShell and start n8n:

```powershell
docker volume create n8n_data

docker run -d --name n8n -p 5678:5678 `
  -e GENERIC_TIMEZONE="Asia/Singapore" `
  -e TZ="Asia/Singapore" `
  -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true `
  -e N8N_RUNNERS_ENABLED=true `
  -v n8n_data:/home/node/.n8n `
  docker.n8n.io/n8nio/n8n:stable
```

Check it in PowerShell:

```powershell
docker ps
```

Open in any browser:

- `http://localhost:5678`

Create your owner account if this is the first run.

## 3. Enable Official MCP in n8n

Inside n8n:

1. Open `Settings`
2. Open the instance-level MCP page
3. Enable MCP access
4. Copy the server URL
5. Copy the access token

For your current local setup, the working live MCP URL is:

```text
http://localhost:5678/mcp-server/http
```

## 4. Save the Live Token

Save the token at user scope in PowerShell so the Codex desktop app can read it after restart:

```powershell
[Environment]::SetEnvironmentVariable('N8N_MCP_TOKEN', '<paste-token-here>', 'User')
```

Verify it in PowerShell:

```powershell
[Environment]::GetEnvironmentVariable('N8N_MCP_TOKEN', 'User')
```

## 5. Create Codex MCP Config

Create the config file in PowerShell:

```powershell
mkdir $HOME\.codex -Force
notepad $HOME\.codex\config.toml
```

Paste this into the config.toml that is opened with the previous PowerShell command:

```toml
[mcp_servers.n8n_docs]
command = "npx"
args = ["n8n-mcp"]
env = { MCP_MODE = "stdio", LOG_LEVEL = "error", DISABLE_CONSOLE_OUTPUT = "true" }

[mcp_servers.n8n_live]
url = "http://localhost:5678/mcp-server/http"
bearer_token_env_var = "N8N_MCP_TOKEN"
enabled = true
```

If `npx` is blocked on Windows, replace the `n8n_docs` block with:

```toml
[mcp_servers.n8n_docs]
command = "C:\\Program Files\\nodejs\\npx.cmd"
args = ["n8n-mcp"]
env = { MCP_MODE = "stdio", LOG_LEVEL = "error", DISABLE_CONSOLE_OUTPUT = "true" }
```

## 6. Create Codex Routing Rules

Create the file in PowerShell:

```powershell
notepad $HOME\.codex\AGENTS.md
```

Paste this into AGENTS.md:

```md
## n8n workflow rules

- Always use `n8n_docs` FIRST for:
  * finding nodes
  * building workflows
  * validating structure

- ONLY use `n8n_live` for:
  * creating workflows
  * updating workflows
  * reading workflows
  * executing workflows

- NEVER touch live workflows until:
  1. A full plan is shown
  2. Inputs/credentials are clear

- Keep workflows INACTIVE by default

- Prefer simple nodes over code nodes

- If unsure, stop and ask for clarification before taking any action.
```

## 7. Restart Codex

After changing any of these:

- `config.toml`
- `AGENTS.md`
- `N8N_MCP_TOKEN`

fully close and reopen the Codex app.

## 8. Verify Both MCP Servers

In Codex, open:

- `Settings -> MCP servers`

You should see:

- `n8n_docs`
- `n8n_live`

Then run these smoke tests in Codex:

```text
Use n8n_live.

List workflows.
```

```text
Use n8n_docs.

Find the smallest no-credentials workflow pattern that uses Manual Trigger and Set.
Do not create anything yet.
```

## 9. Create a Tiny Live Smoke-Test Workflow

This is the safest first workflow:

- `Manual Trigger`
- `Set`
- no credentials
- inactive by default

Use this prompt in Codex:

```text
Use n8n_docs first.

Design and validate a tiny no-credentials workflow with:
- Manual Trigger
- Set

Then use n8n_live to create it in my local n8n instance as INACTIVE.
Name it "Codex Smoke Test".
```

## Tunnel Setup: Expose n8n Beyond Localhost

You only need this when something outside your PC must call your local n8n.

Examples:

- Stripe webhooks
- Telegram bot webhooks
- GitHub webhooks
- Typeform or Meta callbacks

You do not need a tunnel just for Codex talking to your local `n8n_live`.

### Option A: Cloudflare Tunnel

Recommended for quick local testing.

Download:

- [https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/)

Run in PowerShell:

```powershell
cloudflared tunnel --url http://localhost:5678
```

You will get a public URL like:

```text
https://something.trycloudflare.com
```

### Option B: ngrok

Alternative if you prefer ngrok.

Download:

- [https://ngrok.com/downloads/windows](https://ngrok.com/downloads/windows)

Run in PowerShell:

```powershell
ngrok http 5678
```

You will get a public URL like:

```text
https://random-subdomain.ngrok-free.app
```

### Important Tunnel Note

If you want webhook URLs generated by n8n to use the public tunnel URL instead of `localhost`, restart n8n with `WEBHOOK_URL` set to the tunnel URL.

Example in PowerShell:

```powershell
docker stop n8n
docker rm n8n

docker run -d --name n8n -p 5678:5678 `
  -e GENERIC_TIMEZONE="Asia/Singapore" `
  -e TZ="Asia/Singapore" `
  -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true `
  -e N8N_RUNNERS_ENABLED=true `
  -e WEBHOOK_URL="https://your-public-url/" `
  -v n8n_data:/home/node/.n8n `
  docker.n8n.io/n8nio/n8n:stable
```

Replace `https://your-public-url/` with your Cloudflare or ngrok URL.

If the URL changes, update it and restart n8n again.

## Hostinger Later

Use this model:

1. Build locally first
2. Move to a Hostinger VPS later
3. Keep `n8n_docs` local in Codex
4. Point `n8n_live` at the hosted n8n MCP URL when you are ready

Important:

- use a VPS, not normal shared hosting
- give n8n its own subdomain
- use HTTPS
- set `WEBHOOK_URL` to the real public URL
- enable official MCP on the hosted n8n instance too

Typical hosted shape:

- Codex app on your PC
- `n8n_docs` running locally on your PC
- hosted `n8n` on Hostinger VPS
- `n8n_live` updated to the hosted MCP URL

When you move to Hostinger, these are the main changes:

1. your `n8n_live` URL changes from local to hosted
2. your third-party OAuth redirect URLs change from `localhost` to your real domain
3. your tunnel is no longer needed for normal production webhooks

## Fast Troubleshooting

### `n8n_docs` missing in Codex

Check:

1. `C:\Users\<your-user>\.codex\config.toml` exists
2. the `n8n_docs` block is present
3. the `env = { ... }` line is present
4. Codex was fully restarted
5. if needed, use `C:\Program Files\nodejs\npx.cmd`

### `n8n_live` missing in Codex

Check:

1. `C:\Users\<your-user>\.codex\config.toml` exists
2. the URL is `http://localhost:5678/mcp-server/http`
3. `bearer_token_env_var = "N8N_MCP_TOKEN"` is present
4. `N8N_MCP_TOKEN` exists at user scope
5. Codex was fully restarted

### `n8n_live` returns `401`

Your token is wrong or stale.

Reset it in PowerShell:

```powershell
[Environment]::SetEnvironmentVariable('N8N_MCP_TOKEN', '<paste-token-here>', 'User')
```

Then fully restart Codex.

### `n8n` does not open

Run in PowerShell:

```powershell
docker ps
docker logs n8n
```

Restart it in PowerShell if needed:

```powershell
docker restart n8n
```

### Port `5678` is busy

Either free the port or run n8n on a different host port and update every URL that points to it.

Example in PowerShell:

```powershell
docker stop n8n
docker rm n8n

docker run -d --name n8n -p 5679:5678 `
  -e GENERIC_TIMEZONE="Asia/Singapore" `
  -e TZ="Asia/Singapore" `
  -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true `
  -e N8N_RUNNERS_ENABLED=true `
  -v n8n_data:/home/node/.n8n `
  docker.n8n.io/n8nio/n8n:stable
```

Then update the live MCP URL from `5678` to `5679`.
