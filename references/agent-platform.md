# GoDaddy CLI, MCP, and agent platform

Verified 2026-10-25 using the checksum-verified official gddy v0.2.32 release, its default and staged command trees, built-in guides, API catalog, and the public Domains MCP documentation. Re-run discovery when using a newer release. Coverage here means workflow guidance, not that every product is enabled on the user's account.

## Select the surface

| Task | Surface |
| --- | --- |
| Public domain search and availability | Public Domains MCP, or gddy domain |
| Account domains, quotes, purchase, DNS, nameservers, auto-renew, transfer lock | Authenticated gddy domain / dns |
| Buy GoDaddy products (hosting plans, email), review orders | gddy shopping |
| Commerce stores, catalogs, orders, payments, taxes, subscriptions | gddy api REST/GraphQL catalog; inspect operation scopes and schemas |
| Node.js apps, source, deployments, secrets, logs, domains, plans | gddy hosting |
| Mailbox list/get/create/eligibility | gddy email |
| MySQL access to a Node.js or Managed WordPress app | Experimental gddy db tunnel |
| On-demand URL/domain malware and reputation scan | Beta gddy malware scan run |
| Platform apps, extensions, actions, subscriptions, releases | gddy platform |
| Agent identity, discovery, verification, certificates, events | ANS official REST reference and SDKs |

## Official agent integrations

Source: https://github.com/godaddy/cli

The official CLI skill can also be installed for a supported agent:

```bash
npx skills add godaddy/cli --skill gddy --agent <agent>
```

Claude Code integration:

```bash
claude plugin marketplace add godaddy/cli
claude plugin install gddy@godaddy
```

These are optional host installations. Choose the user's actual agent and inspect installer help before changing host configuration. CI workflows can use the official `godaddy/cli/action` GitHub Action to install gddy. The older TypeScript `godaddy` CLI lives on the repository's `original` branch; do not mix its command syntax with gddy.

## Public Domains MCP

Documentation: https://developer.godaddy.com/en/docs/api-users/mcp

Claude Desktop can add it from Settings → Connectors → Browse connectors → GoDaddy. For other hosts, configure a Streamable HTTP MCP connection using this documented example; adapt transport syntax to that host:

```json
{
  "mcpServers": {
    "godaddy": {
      "url": "https://api.godaddy.com/v1/domains/mcp",
      "transport": "streamable-http"
    }
  }
}
```

No authentication is needed or accepted for account access. Initialize the connection and discover tools with tools/list; inspect their actual input schemas. The server is read-only public discovery, not DNS editing, purchase, transfer, or account administration. Route those operations to gddy. Recheck availability before buying.

The former godaddy/ai and godaddy/commerce-agent-plugin GitHub URLs still returned 404 on 2026-10-25. Do not install them or infer a working Commerce MCP endpoint from stale search snippets. Use the current authenticated Commerce CLI/API catalog until an accessible official MCP contract is verified.

## Complete CLI discovery, including staged features

```bash
gddy --version
gddy tree
gddy guide
gddy flags list
gddy api domain list
GDDY_MIN_STAGE=beta gddy tree
GDDY_MIN_STAGE=experimental gddy tree
```

Use the minimum stage required by the requested feature and keep the environment variable scoped to that command. Stage visibility does not grant account access. Inspect command --help before execution; a listed command is not proof the remote service is enabled.

The v0.2.32 default tree covers auth, API/schema/GraphQL discovery and calls, DNS, domains, Email, Hosting, Shopping, environments, PATs, payment methods, Platform, updates, flags, guides, search, and completion. Beta adds `malware`; experimental adds `db` and Platform native Android extensions (`native-apps` flag). Plural aliases such as `domain nameservers` still resolve, but the canonical nouns are singular.

### Hosting

```bash
gddy guide hosting
gddy hosting --help
```

Groups: `app` list/get/create/update/delete/status/restart; `deployment` list/get/publish; `source` upload/status/github; `secret` list/create/update/delete/sync; `log list`; `runtime get`; `domain` list/get/attach/detach; `subscription` list/get/attach; `operation get`.

Flow: `app create --app-type NODEJS` → poll `operation get` → `source upload --file app.zip` (lands on PREVIEW) → poll `source status` → verify the preview → `subscription attach` → `deployment publish` (PUBLISH/production) → verify live status. The zip needs a top-level `package.json` with name, version, main, `build` and `start` scripts, runtime packages in `dependencies`, and must listen on `process.env.PORT`; exclude `node_modules`. If no plan exists, buy one through `gddy shopping` with the user's approval. `app delete` needs a sensitive scope and is irreversible. Keep secret values out of output and preserve the preview versus live distinction.

### Email

`gddy guide email`; commands are list, get, create, and check-eligibility. Check eligibility first; if a mailbox needs a plan, the guide routes purchase through `gddy shopping`. Confirm any price/account effect from the current response.

### Shopping

```bash
gddy guide shopping
gddy shopping catalog category
gddy shopping catalog search <query>
gddy shopping catalog get <product-id>
gddy shopping checkout create --help
gddy shopping checkout get <checkout-id>
gddy shopping checkout complete <checkout-id>
gddy shopping order get <order-id>
```

Creating or updating a checkout session does not charge. `checkout complete` places a paid order with the selected saved payment method (`--payment-instrument` to choose another); `--agree` accepts every required agreement. Before completing, show the final total including fees, each agreement's title and link, and the important links, then obtain explicit approval for that session. Add a payment method with `gddy payment-method add` (opens the browser).

### Database tunnel (experimental)

```bash
GDDY_MIN_STAGE=experimental gddy db tunnel --app-id <id> --product nodejs|mhwp
```

Opens a local MySQL port forwarded over the app agent's WebSocket. It needs the sensitive `hosting.database.tunnel:execute` scope, runs until interrupted, and never handles database credentials; the user logs in with their own DB user and should connect with TLS required. A Managed WordPress site allows one tunnel at a time and may take minutes to open. Run it as a background process and stop it when done.

### Malware scan (beta)

```bash
GDDY_MIN_STAGE=beta gddy malware scan run https://example.com
```

Returns malware detection, blocklist reputation, software fingerprinting, security headers, and TLS health synchronously. Scan only sites the user owns or is authorized to assess.

### Platform apps

```bash
gddy guide platform-overview
gddy guide platform-settings
gddy platform app --help
gddy platform action --help
gddy platform webhook --help
```

App commands cover init, import (pull an existing app's config and webhook subscriptions into godaddy.toml), list, info, update, local `config validate`, remote validate, enable/disable on a store, enablement (apps enabled on a store), archive, release, and deploy. Manifest additions include embed/checkout/blocks extensions, actions, webhook subscriptions, settings placements, and OAuth redirect URIs. Discover action contracts with `action list/describe` and event types with `webhook event`. Validate godaddy.toml locally and remote state before releasing; inspect deploy progress and resulting state. Do not equate local manifest edits with a deployed or store-enabled app.

## Agent Name Service (ANS)

Start with https://developer.godaddy.com/en/docs/references/rest/ans/registration and the official reference navigation for resolution, search, validation, agents, certificate management, events, and revocation. The open-source implementation and SDK links live at https://github.com/agentnameservice/ans.

ANS concerns agent names, identity certificates, domain ownership, discovery, and verifiable lifecycle history. It is separate from installing an agent skill or connecting the Domains MCP. Use the exact production authentication and schema for the chosen API/SDK; do not assume a Domains PAT or local development API key is accepted. Keep private keys local, send only required CSRs/public material, complete returned DNS/ACME challenges, then verify registry/certificate status. A submitted registration is not a verified identity. Never promote local demo defaults, no-op DNS verification, or development keys to production.

## Coverage limits

The v0.2.32 CLI catalog exposes 24 API domains, including Commerce, Hosting, Email, Bulk Operations, and App Registry, but it is not every GoDaddy REST API. For Certificates, Aftermarket, Shoppers, Agreements, Countries, Abuse, Parking, or ANS absent from that catalog, open https://developer.godaddy.com/en/docs/references/rest and inspect the exact current contract. An empty CLI search is not proof the product has no API. Do not invent paths or reuse the obsolete custom MCP server from skill v1.
