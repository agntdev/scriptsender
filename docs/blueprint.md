# ScriptSender — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that allows authorized senders to deliver a predefined script to a preconfigured recipient via private message. Senders trigger delivery through the bot, while admins manage script content, recipient ID, and sender permissions.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- authorized senders
- admin (owner)

## Success criteria

- Recipient receives script message successfully
- Delivery records are logged with timestamp and sender ID
- Admin can update script content and sender permissions

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with sender/admin options
- **/send_script** (command, actor: sender, command: /send_script) — Initiate script delivery workflow
- **Edit Script** (button, actor: admin, callback: admin:edit_script) — Open script editing interface
- **Manage Senders** (button, actor: admin, callback: admin:manage_senders) — Add/remove authorized senders

## Flows

### Sender Delivery Flow
_Trigger:_ /send_script

1. Show script preview with confirmation dialog
2. Send script to recipient if confirmed
3. Notify sender of delivery status

_Data touched:_ script, delivery_record

### Admin Setup Flow
_Trigger:_ admin:edit_script

1. Display current script content
2. Accept new script text input
3. Save updated script

_Data touched:_ script

### Recipient Configuration Flow
_Trigger:_ admin:set_recipient

1. Request new recipient ID
2. Validate and store ID

_Data touched:_ recipient

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **sender** _(retention: persistent)_ — Authorized user who can trigger script delivery
  - fields: telegram_user_id, display_name
- **recipient** _(retention: persistent)_ — Fixed user who receives script messages
  - fields: telegram_user_id
- **script** _(retention: persistent)_ — Text content to deliver
  - fields: content, last_modified
- **delivery_record** _(retention: persistent)_ — Audit log of deliveries
  - fields: timestamp, sender_id, script_version, status

## Integrations

- **Telegram** (required) — Bot API messaging and user authentication
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Manage allowed senders list
- Edit script content
- Change recipient ID
- View delivery logs

## Notifications

- Admin confirmation message on script update
- Delivery success/failure notification to sender
- Optional admin delivery log summary

## Permissions & privacy

- Store authorized sender Telegram IDs securely
- Recipient ID is owner-controlled and private
- Script content is owner-controlled text

## Edge cases

- Recipient has blocked the bot
- Sender not in allowlist
- Empty script content
- Invalid Telegram user IDs

## Required tests

- End-to-end sender delivery workflow with confirmation
- Admin script update workflow
- Recipient ID validation and change handling

## Assumptions

- Admin provides recipient ID during setup
- Admin manages sender allowlist
- Script content remains plain text only
