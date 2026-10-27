---
name: browse-bridge
description: Use when the user asks Claude to look at, read, click, fill, or navigate a page in their own Chrome browser via Browse Bridge (browser_* MCP tools). Covers the observe-then-act loop, approvals, and handing control back.
---

# Using Browse Bridge shared tabs

Browse Bridge only exposes Chrome tabs the user has explicitly **Enabled** in the Browse Bridge extension. You cannot open or enable tabs yourself.

## Loop

1. `browser_list_sessions` — find Enabled tabs. If none, ask the user to click the Browse Bridge toolbar icon and **Enable** the tab (normal http/https pages only, not `chrome://`).
2. `browser_get_status` — check `controlMode`, `deviceOnline`, and pending approvals before acting.
3. `browser_observe` — get the semantic page tree. Use the returned `observationId` / `elementId`s; re-observe after any navigation or significant page change rather than reusing stale IDs.
4. Act with `browser_click`, `browser_type`, `browser_select`, `browser_press_key`, `browser_hover`, `browser_scroll`, `browser_navigate`, `browser_back`, `browser_forward`, `browser_reload`.
5. Verify with `browser_observe` or `browser_screenshot`. Use `browser_wait` (0–30000 ms) for slow loads instead of rapid polling.
6. When done, or if the user wants the tab back, call `browser_release_control`.

## Rules

- Never type passwords; `browser_type` refuses password fields. Ask the user to enter credentials themselves.
- `APPROVAL_REQUIRED` means the human must Approve/Deny in the Chrome side panel (sensitive sites, cross-origin navigation). Tell the user and wait; do not retry in a loop. Use `browser_cancel_approval` with the `approvalId` to withdraw your own request (`APPROVAL_CANCELLED` is not a human Deny).
- CAPTCHAs/challenges: stop and ask the user to take over.
- The user can take control at any time (Take control / Ctrl+Shift+Period). If control mode is `human`, do not act until it is returned.
- Respect the terms of service of the sites involved; confirm before submitting forms, purchasing, or sending messages.
