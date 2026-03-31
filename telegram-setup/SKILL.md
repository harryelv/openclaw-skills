---
name: telegram-setup
description: Guided, step-by-step Telegram bot setup for OpenClaw using Telegram account keys (not Telegram numeric IDs), including BotFather creation, account-to-agent binding, OpenClaw config, and safe group policy setup.
---

# Telegram Setup Skill

## Purpose
Help the user fully set up a Telegram bot inside OpenClaw with minimal friction:
1. Guide bot creation in BotFather.
2. Collect and validate required inputs (bot token, account key/name, target agent, approval code if needed).
3. Configure OpenClaw `channels.telegram.accounts` correctly.
4. Guide Telegram privacy mode setup for group support.
5. Configure OpenClaw group policy safely (`allowlist` or `open`).

## Important Source of Truth
For account setup behavior, follow OpenClaw Telegram docs:
- https://docs.openclaw.ai/channels/telegram

When this skill conflicts with old assumptions, prefer docs behavior:
- OpenClaw Telegram setup uses an **account key/name** under `channels.telegram.accounts`.
- Do **not** require Telegram numeric user ID or bot ID for normal setup.

## Use this skill when
- User asks to set up/connect Telegram with OpenClaw.
- User wants step-by-step Telegram bot onboarding.
- User needs bot group-chat compatibility.
- User wants account key ↔ agent routing behavior clarified.

## Do not use this skill when
- User asks only for advanced Telegram bot API design (handlers/architecture).
- User asks for unrelated messaging channels.

## Conversation Style Rules
- Guide setup in **small steps** (one action at a time).
- Wait for confirmation after each step.
- Do not dump everything at once unless asked.
- Be explicit about exactly what to click/copy/paste.
- If token appears, acknowledge without echoing full token.

## Language Simplicity (Important)
- Always guide in simple, plain language.
- Use short sentences and clear actions.
- Avoid jargon unless user asks for technical detail.
- If technical terms are needed, explain in one line.

## Required Inputs (Target)
Collect these in order:

1. **Bot Token** (from BotFather)
2. **Telegram Account Key/Name** (example: `john`, `marketing_manager`) used in OpenClaw config under `channels.telegram.accounts.<accountKey>`
3. **Target Agent ID** for this account (or default `main` with confirmation)
4. **Group Policy Preference** (`allowlist` recommended, `open` only with warning + confirmation)
5. **Approval Code** only if the environment flow requires one

Optional:
- Group/chat usernames or IDs to add into allowlist

## Input Qualification Rules

### Bot Token
Qualified if pattern matches `<digits>:<token-part>`.
If not qualified, ask user to re-copy from BotFather.

### Telegram Account Key/Name
Qualified if:
- Non-empty slug-like key (letters/numbers/_/-)
- Safe to use as config object key

If not qualified:
- Ask user to provide a simple key like `mybot` or `john`.

### Agent ID
Qualified if it maps to an existing/accessible target agent/session.
If unclear, offer fallback to default `main` and require explicit confirmation.

## Binding Resolution (Account Key ↔ Agent)

This is mandatory before writing config.

### Goal
Create clear mapping from OpenClaw Telegram account key to handling agent.

### Resolution Order
1. If user provides qualified `accountKey` + `agentId`, use them.
2. If `accountKey` is missing/unqualified, ask:
   - “I couldn’t confirm the Telegram account key/name. Use `default` account key instead? (yes/no)”
3. If `agentId` is missing/unqualified, ask:
   - “I couldn’t confirm the target agent. Route this account to default agent `main` instead? (yes/no)”
4. If both are missing/unqualified, ask both confirmations in sequence.
5. If user declines either fallback, pause and ask for qualified input.

### Safety Rule
Never silently assume default account key or default agent. Always ask first.

## Group Policy Setup (OpenClaw)

### Default recommendation
Recommend `groupPolicy: allowlist` for safety.

### If user chooses `allowlist`
- Help user add allowed groups explicitly in OpenClaw config (format depends on current config schema).
- Ask for each group identifier needed and confirm before writing.
- Explain briefly: bot only responds in approved groups.

### If user chooses `open`
Before applying, show warning and require explicit confirmation:
- “Warning: `groupPolicy=open` allows broader group interaction and can expose the bot to unwanted group traffic. This is risky. Do you still want to continue? (yes/no)”
- Proceed only on explicit yes.

## Step-by-Step Flow

### Step 1) Create bot via BotFather
1. Open Telegram → `@BotFather`
2. Send `/newbot`
3. Set name and username
4. Copy token and paste into chat

### Step 2) Configure basic bot settings in BotFather
- `/setdescription`
- `/setabouttext`
- `/setcommands` (optional)

### Step 3) Disable bot privacy mode for group message access
1. `@BotFather` → `/mybots`
2. Choose bot
3. `Bot Settings` → `Group Privacy`
4. Tap **Turn off**

Also ensure bot can be added to groups.

### Step 4) Collect account key + target agent + policy
Collect:
- account key/name for `channels.telegram.accounts.<key>`
- target agent id
- group policy preference (`allowlist`/`open`)
- approval code only if needed

Run **Binding Resolution** and **Group Policy Setup** rules before continuing.

### Step 5) Validate bot token
Use Telegram `getMe` and confirm success (`ok: true`).
If invalid, guide token regeneration and retry.

### Step 6) Configure OpenClaw
Apply configuration idempotently:
- `channels.telegram.enabled = true`
- Ensure `channels.telegram.accounts` exists
- Upsert selected `accountKey` with at least:
  - `enabled: true`
  - `botToken: <token>`
  - `dmPolicy` (as chosen/default)
  - `groupPolicy` (from confirmed choice)
  - `streaming` (as chosen/default)
- Preserve unrelated config keys
- Apply account→agent binding according to current OpenClaw schema

If using `allowlist`, also write/update group allowlist entries.

Then:
- restart if required: `openclaw gateway restart`
- verify: `openclaw gateway status`

If command/path uncertain, check local docs/help first.

### Step 7) End-to-end test
1. DM test (`/start`)
2. Group test (normal non-command message)
3. Confirm behavior matches policy:
   - `allowlist`: only approved groups respond
   - `open`: broader group response expected

## Error Handling Rules
- **401 Unauthorized**: token invalid/revoked
- **429 Too Many Requests**: backoff and retry
- **No updates**: verify channel config + gateway status
- **No group response**: re-check privacy off and group policy settings
- **Unexpected group response**: verify allowlist/open config and routing

## Security Rules
- Never expose full token in logs/replies.
- Mask token in confirmations.
- Do not post secrets to groups.
- Confirm before risky policy changes (`groupPolicy=open`).

## Completion Criteria
Done only when all are true:
- Token validated
- Account key selected and configured
- Account-to-agent binding resolved (explicit or confirmed default)
- Group policy configured (and warning acknowledged if `open`)
- Allowlist configured when using `allowlist`
- Gateway healthy after apply/restart
- DM + group tests pass

## Suggested First Reply Template
“Great — I’ll guide this step by step in simple language.
Step 1: create your bot with `@BotFather` using `/newbot`, then paste the bot token here.
After that, I’ll help you set the OpenClaw account key and routing safely.”
