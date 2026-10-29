---
name: browse-bridge
description: Use when the user asks Claude to look at, read, click, fill, or navigate a page in their own Chrome browser via Browse Bridge (browser_* MCP tools). Covers the observe-then-act loop, approvals, and handing control back.
---

# Using Browse Bridge shared tabs

Browse Bridge only exposes Chrome tabs the user has explicitly **Enabled** in the Browse Bridge extension. You cannot open or enable tabs yourself.

## Loop

1. `browser_list_sessions` — find Enabled tabs. If none, ask the user to click the Browse Bridge toolbar icon and **Enable** the tab (normal http/https pages only, not `chrome://`).
2. `browser_get_status` — check `controlMode`, `deviceOnline`, `sensitiveMode`, and pending approvals before acting. When `sensitiveMode` is true, click, type, key, select, and navigation wait for Approve.
3. `browser_observe` — get the semantic page tree. Use the returned `observationId` / `elementId`s; re-observe after any navigation or significant page change rather than reusing stale IDs.
4. Act with `browser_click`, `browser_type`, `browser_select`, `browser_press_key`, `browser_hover`, `browser_scroll`, `browser_navigate`, `browser_back`, `browser_forward`, `browser_reload`.
5. If an action returns `APPROVAL_REQUIRED`, tell the user and poll `browser_get_approval` (`sessionId`, `approvalId`). That call is read-only and does not include the action payload. Do not repeat the action just to check status (`APPROVAL_PENDING`). Status `expired`, or a later call that returns `APPROVAL_EXPIRED`, means the approval timed out after about 2 minutes. Retry the original action only if the user still wants it; that asks for a new approval. Withdraw a still-pending request with `browser_cancel_approval` (`APPROVAL_CANCELLED` is not a human Deny). Cancelling an expired approval returns `APPROVAL_EXPIRED` and does not cancel it.
6. Verify with `browser_observe` or `browser_screenshot`. Use `browser_wait` (0–30000 ms) for slow loads instead of rapid polling.
7. When done, or if the user wants the tab back, call `browser_release_control`.

## Rules

- Never type passwords; `browser_type` refuses password fields. Ask the user to enter credentials themselves.
- `APPROVAL_REQUIRED` means the human must Approve or Deny in the Chrome side panel. Sensitive site gates click, type, key, select, and navigation; cross-origin navigations and cross-origin link clicks do too. That wait is not `CHALLENGE_DETECTED`. Tell the user and wait. Poll with read-only `browser_get_approval` (`sessionId`, `approvalId`) instead of repeating the action; it does not return the action payload. Repeating the same pending action returns `APPROVAL_PENDING`. Status `expired`, or `APPROVAL_EXPIRED` on the next action or idempotency-key call, means the approval timed out after about 2 minutes and is recoverable: retry the action to request a new approval. `browser_cancel_approval` withdraws a still-pending request (`APPROVAL_CANCELLED` is not a human Deny). On an expired approval it returns `APPROVAL_EXPIRED` and does not cancel it. Deny is `APPROVAL_DENIED`.
- CAPTCHAs/challenges: stop and ask the user to take over.
- The user can take control at any time (Take control / Ctrl+Shift+Period). If control mode is `human`, do not act until it is returned.
- Respect the terms of service of the sites involved; confirm before submitting forms, purchasing, or sending messages.
