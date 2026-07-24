# CryptoAlert Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that delivers configurable crypto alerts (price movements, volume spikes, on-chain activity, and social/news mentions) to a public channel and optional direct messages. Users subscribe per-coin with customizable alert types and thresholds.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- crypto traders
- crypto community members

## Success criteria

- Users receive accurate alerts in public channel
- Users can opt into per-coin DM alerts
- Subscriptions and preferences persist across sessions
- Admin can manage alert rules and delivery logs

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open onboarding menu to link account and set DM preferences
- **/subscribe** (command, actor: user, command: /subscribe) — Initiate subscription flow for a new coin
- **/subscriptions** (command, actor: user, command: /subscriptions) — List active subscriptions and alert preferences
- **/unsubscribe** (command, actor: user, command: /unsubscribe) — Remove subscription for a specific coin
- **Enable DM alerts** (button, actor: user, callback: opt_in:dm) — Toggle direct message notifications for subscribed coins

## Flows

### Onboarding
_Trigger:_ /start

1. Display welcome message
2. Request channel linkage
3. Ask for DM opt-in preference

_Data touched:_ User

### Subscription management
_Trigger:_ /subscribe <coin>

1. Normalize coin symbol
2. Display alert type options
3. Set default thresholds
4. Store subscription

_Data touched:_ Subscription, Alert rule

### Alert delivery
_Trigger:_ Alert event triggered

1. Format alert message
2. Post to configured channel
3. Send DM to opted-in users

_Data touched:_ Alert event, User

### Admin controls
_Trigger:_ Owner command

1. Silence specific alert rule
2. Send test alert
3. View delivery logs

_Data touched:_ Alert rule, Alert event

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user account with subscription preferences
  - fields: telegram_id, dm_opt_in_coins, last_alert_time
- **Subscription** _(retention: persistent)_ — User-coin alert preferences
  - fields: user_id, coin_symbol, alert_types, thresholds
- **Alert rule** _(retention: persistent)_ — Monitored market condition parameters
  - fields: coin, alert_type, threshold, time_window
- **Alert event** _(retention: persistent)_ — Generated alert instance with metadata
  - fields: timestamp, coin, alert_type, score, excerpt, link

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Silence specific alert rules
- Send test alerts to channel
- View alert delivery logs
- Configure target channel/group

## Notifications

- Formatted crypto alerts in public channel
- Direct message notifications for opted-in users

## Permissions & privacy

- Store user subscriptions with opt-in consent
- Only send DMs to users who explicitly enable them
- Anonymize user data when purging

## Edge cases

- Invalid/unknown coin symbols
- Duplicate alerts within short time window
- Channel post formatting failures
- DM opt-in without active subscriptions

## Required tests

- End-to-end alert triggering to channel and DMs
- Subscription management flow validation
- Admin control command execution

## Assumptions

- Default thresholds reduce setup friction
- External APIs handle price/on-chain data
- Users know basic crypto symbols
