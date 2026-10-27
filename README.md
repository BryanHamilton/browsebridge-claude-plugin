# Browse Bridge for Claude Code

Claude Code plugin that connects the **Browse Bridge** remote MCP server so Claude can observe and control Chrome tabs you have explicitly **Enabled** in the Browse Bridge Chrome extension.

- **MCP endpoint:** `https://mcp.browsebridge.com/mcp` (Streamable HTTP)
- **Site:** https://browsebridge.com
- **Chrome extension:** [Chrome Web Store](https://chromewebstore.google.com/detail/browse-bridge/fbnpefbmmljjgnlagkcbnnfknpomhidb)
- **License:** MIT

## Prerequisites

1. A Browse Bridge account: https://browsebridge.com/sign-up (plans: https://browsebridge.com/plans; paid tier $29.97/mo).
2. The Browse Bridge Chrome extension, signed in with the same account.
3. An API key from https://browsebridge.com/dashboard/connections (starts with `bbk_`). Keep it secret. When you enable the plugin, Claude prompts for that key and stores it securely. Do not export it for the plugin.

## Install

```text
/plugin marketplace add BryanHamilton/browsebridge-claude-plugin
/plugin install browse-bridge@browse-bridge
```

When you enable the plugin, Claude prompts for the API key and stores it securely. Then run `/mcp` to confirm `browse-bridge` is connected and the `browser_*` tools are listed.

### Without the plugin

Connect Claude Code to the remote server using the steps at https://browsebridge.com/docs/mcp.

## Usage

1. Open a normal website tab (not `chrome://`).
2. Click the Browse Bridge toolbar icon and **Enable** the tab. Use **Take control** (Ctrl+Shift+Period) to take it back at any time.
3. Ask Claude to work in that tab. The bundled `browse-bridge` skill teaches the observe → act → verify loop.
4. Sensitive sites and cross-origin navigations return `APPROVAL_REQUIRED`; Approve or Deny in the Chrome side panel.

## Tools (19)

| Tool | What it does |
| --- | --- |
| `browser_list_sessions` | List Enabled shared tabs. |
| `browser_get_status` | Control mode, URL, `deviceOnline`, pending approvals, remaining monthly quota (`actionsRemaining`, `screenshotsRemaining`, `quotaExceeded`, `quotaResetAt`). Tools return `QUOTA_EXCEEDED` when a monthly quota is exhausted. |
| `browser_observe` | Semantic page tree + `observationId` / `elementId`s. |
| `browser_screenshot` | Visible-tab JPEG as MCP image content. |
| `browser_click` | Click an observed element. Optional `button`: `"left"` (default), `"right"`, or `"middle"`. |
| `browser_type` | Set a control's value (not a password). |
| `browser_select` | Choose a listed `<option>`. |
| `browser_press_key` | Key in the focused (or named) element. |
| `browser_hover` | `mouseover` / `mouseenter` on an observed element. |
| `browser_scroll` | Scroll the page or an element into view. |
| `browser_navigate` | Go to an allowed http(s) URL. |
| `browser_back` | History back. |
| `browser_forward` | History forward. |
| `browser_reload` | Reload the tab. |
| `browser_wait` | Pause 0–30000 ms. |
| `browser_release_control` | Give the tab back to the human (`controlMode: "human"`). |
| `browser_handle_dialog` | Accept or dismiss a pending JavaScript `alert` / `confirm` / `prompt` (`action`: accept\|dismiss; optional `promptText` when accepting a prompt). |
| `browser_cancel_approval` | Withdraw a pending agent approval (`approvalId`). |
| `get_profile` | Read-only Browse Bridge user id for the signed-in account (no email, tokens, cookies, or page content). |

### Dialogs

While the agent has control, a page `alert` / `confirm` / `prompt` stays **pending** until `browser_handle_dialog` (or about a 30-second timeout dismiss). Other click, type, and navigate calls while a dialog is open return **`DIALOG_PENDING`**. Native HTTP basic or digest sign-in prompts are not JavaScript dialogs (`HTTP_AUTH_REQUIRED`). The human completes or cancels them. Browse Bridge never types those credentials.

### Approvals

Sensitive-site clicks/types/navigations and cross-origin navigations (and cross-origin link clicks) return **`APPROVAL_REQUIRED`**. The human Approves or Denies in the Chrome side panel. The agent can withdraw its own request with **`browser_cancel_approval`** (`APPROVAL_CANCELLED` — that is not a human Deny). While an approval is pending, observe/screenshot/wait/get_status/list_sessions/release_control still run.

## Configuration

| Variable | Required | Description |
| --- | --- | --- |
| `userConfig.api_key` | Yes | Sensitive plugin setting, not an environment variable. Claude prompts at enable and stores it securely. Sent only to `https://mcp.browsebridge.com/mcp` as `Authorization: Bearer …`. |

## Privacy Policy

This plugin only configures a remote MCP connection; it ships no code and collects no telemetry. Page content, screenshots, and action results from tabs you Enable are processed by Browse Bridge to fulfil tool calls. Full policy: https://browsebridge.com/privacy · Terms: https://browsebridge.com/terms · See also [PRIVACY.md](./PRIVACY.md).

## Support

support@browsebridge.com · https://browsebridge.com/faq · Security: [SECURITY.md](./SECURITY.md)
