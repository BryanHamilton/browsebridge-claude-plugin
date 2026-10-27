# Security

- Report vulnerabilities in Browse Bridge infrastructure or this plugin config to **support@browsebridge.com**.
- Abuse / legal process: **abuse@browsebridge.com** (also listed on browsebridge.com).
- Never commit real `bbk_` API keys. The plugin header uses `${user_config.api_key}`. That value is sent only to `https://mcp.browsebridge.com/mcp`.
- Prefer least-privilege keys and rotate on suspicion of leak.
- Anthropic directory listing issues: see https://claude.com/docs/directory/publish
