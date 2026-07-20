# TopDuka Storefront Installation

This guide installs the storefront, connects it to a TopDuka store, and optionally installs the included Codex skill.

## Requirements

- Node.js 20 or newer
- npm
- A running TopDuka backend
- A TopDuka public API key beginning with `ak_`

If TopDuka is hosted on ScaleNodes, use the public service endpoint for port `8080`. Port `3000` serves the TopDuka dashboard and is not the API endpoint for this storefront.

## 1. Get the template

Clone the repository and enter the template directory:

```bash
git clone <your-topduka-template-repository-url>
cd topduka-template
```

If you downloaded an archive instead, extract it and open a terminal in the extracted `topduka-template` directory.

## 2. Prepare TopDuka

Before starting the storefront:

1. Add at least one product and category.
2. Configure delivery zones and delivery methods.
3. Configure VAT and the payment methods you want to accept.
4. Create a public API key in **API Keys**.
5. Optionally create a customer agent and copy its UUID.

The API key is scoped to its TopDuka store. Treat it as a server secret even though it accesses the public storefront API.

## 3. Install dependencies

```bash
npm install
```

For repeatable CI or production installs after a lockfile has been committed, use `npm ci`.

## 4. Configure the environment

Create a local environment file:

```bash
cp .env.example .env.local
```

On Windows PowerShell:

```powershell
Copy-Item .env.example .env.local
```

Set the following values in `.env.local`:

```env
TOPDUKA_API_URL=https://your-app-port-8080.go.scalenodes.app
TOPDUKA_API_KEY=ak_your_store_api_key
TOPDUKA_AGENT_ID=optional_agent_uuid
```

`TOPDUKA_AGENT_ID` is optional. `TOPDUKA_API_URL` may be the API origin or a URL ending in `/pb/v1`; the template normalizes both.

Never rename the API key variable to `NEXT_PUBLIC_TOPDUKA_API_KEY`. The browser must not receive this key.

## 5. Start the storefront

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000). Confirm that products load, then add a product to the cart and load checkout to verify the API key, shipping configuration, and server-side cart bridge.

## 6. Validate a production build

```bash
npm run lint
npm run build
npm start
```

The production server listens on port `3000` by default. Configure the three environment variables as secrets in your hosting platform; do not upload `.env.local`.

## Connect an MCP client

The same TopDuka API origin exposes its MCP server at `/mcp`:

```text
URL: https://your-app-port-8080.go.scalenodes.app/mcp
Authorization: Bearer ak_your_store_api_key
```

After connecting, an MCP-compatible coding agent can:

- Read `topduka://api/reference` for the complete public API contract.
- Use the `implement_topduka_api` prompt when adding an integration.
- Inspect store-scoped products, categories, shipping, and orders with read-only tools.

Use the same server-only API key boundary described above. Do not put the MCP authorization header in browser code.

## Install the included Codex skill

The repository works without installing the skill. Installing it gives Codex reusable TopDuka-specific implementation and security guidance.

From the `topduka-template` directory, run the following in PowerShell:

```powershell
$skillPath = Join-Path $env:USERPROFILE ".codex\skills\topduka-template"
New-Item -ItemType Directory -Force (Join-Path $skillPath "agents") | Out-Null
Copy-Item .\SKILL.md (Join-Path $skillPath "SKILL.md") -Force
Copy-Item .\agents\openai.yaml (Join-Path $skillPath "agents\openai.yaml") -Force
```

On macOS or Linux:

```bash
mkdir -p "$HOME/.codex/skills/topduka-template/agents"
cp SKILL.md "$HOME/.codex/skills/topduka-template/SKILL.md"
cp agents/openai.yaml "$HOME/.codex/skills/topduka-template/agents/openai.yaml"
```

Restart Codex after installing it. Invoke it explicitly with `$topduka-template`, or ask Codex to build or extend a TopDuka storefront while working inside this repository.

## Common installation problems

### Products do not load

- Confirm the URL points to the TopDuka backend on port `8080`, not the dashboard on port `3000`.
- Confirm the API key starts with `ak_` and belongs to the intended store.
- Restart `npm run dev` after changing `.env.local`.

### Checkout has no delivery options

Create and enable delivery zones and methods in TopDuka. The storefront reads live rates and does not invent fallback prices.

### MCP returns `401 Unauthorized`

Send the API key as `Authorization: Bearer ak_...`. The `x-api-key: ak_...` header is also supported.

### The Codex skill is not listed

Confirm the installed file is exactly `~/.codex/skills/topduka-template/SKILL.md`, then restart Codex.
