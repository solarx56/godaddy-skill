# GoDaddy API Skill

An agent skill for GoDaddy's current developer platform, built around the official `gddy` CLI for dynamic REST and GraphQL discovery, domains, registration, nameservers, DNS, authentication, and account operations.

Version 2.1.0 tracks gddy v0.2.32 (verified October 25, 2026): generally available Hosting, Email, and Shopping commands; domain auto-renew and transfer-lock settings; HTTPS, SVCB, and TLSA DNS records; scope-bundle login with sensitive step-up; and the staged `db tunnel` and `malware scan` commands. It also documents the public Domains MCP, official agent integrations, Platform apps/actions/webhooks/extensions, and Agent Name Service. See [coverage and setup](references/agent-platform.md), including current availability limits. The skill supplies instructions; CLI installation and MCP connection setup are separate host operations.

## ClawHub

[View GoDaddy API on ClawHub](https://clawhub.ai/solarx56/skills/godaddy-api)

```bash
openclaw skills install @solarx56/godaddy-api
```

The skill's executable instructions are in [`SKILL.md`](SKILL.md). Detailed API discovery, safety, and retry guidance lives under [`references/`](references/).
