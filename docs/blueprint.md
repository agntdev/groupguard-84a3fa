# GroupGuard — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

GroupGuard is a Telegram group moderation bot that automates verification of new members, detects spam/flood patterns, and provides admin controls for warnings, mutes, kicks, and bans. It maintains a transparent action log with configurable thresholds and trusted-user exceptions, ensuring operational clarity while minimizing noise for admins and members.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- group admins
- Telegram community moderators

## Success criteria

- Automated verification of new members within 3 minutes
- Spam/flood actions logged with human-readable explanations
- Admin commands executed with privilege validation
- Action log retention of 90 days by default

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main admin menu for configuration options
- **I'm human** (button, actor: user, callback: verify:confirm) — Confirm human verification for new members
  - inputs: user_id, join_time
  - outputs: verification status, group access permissions
- **/warn** (command, actor: admin, command: /warn) — Record warning and post public notice
- **/mute** (command, actor: admin, command: /mute) — Restrict user message sending for specified duration
- **/kick** (command, actor: admin, command: /kick) — Remove user from group with reason
- **/ban** (command, actor: admin, command: /ban) — Ban user from group with reason
- **/trust** (command, actor: admin, command: /trust) — Add user to trusted list
- **/untrust** (command, actor: admin, command: /untrust) — Remove user from trusted list
- **/set_greeting** (command, actor: admin, command: /set_greeting) — Configure greeting and rules text
- **/set_thresholds** (command, actor: admin, command: /set_thresholds) — Adjust spam/flood detection sensitivity
- **/report** (command, actor: admin, command: /report) — Generate moderation activity summary
- **/log** (command, actor: admin, command: /log) — View or export action log entries

## Flows

### new_member_verification
_Trigger:_ user_join

1. Send greeting with verification button
2. Enforce message restriction until confirmation
3. Process button click or timeout

_Data touched:_ Member, Verification session

### spam_detection
_Trigger:_ incoming_message

1. Analyze message content and user metadata
2. Check against active moderation rules
3. Apply configured action and post explanation

_Data touched:_ Member, Moderation rule set, Action log entry

### admin_command_processing
_Trigger:_ /warn /mute /kick /ban /trust /untrust /set_greeting /set_thresholds /report /log

1. Validate admin privileges
2. Execute action
3. Create log entry
4. Notify admin of outcome

_Data touched:_ Member, Moderation rule set, Action log entry, Trusted list

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Member** _(retention: persistent)_ — Telegram user with metadata for moderation tracking
  - fields: user_id, display_name, join_time, account_age, admin_flag
- **Verification session** _(retention: session)_ — Pending state for new members requiring human confirmation
  - fields: user_id, join_time, timeout
- **Moderation rule set** _(retention: persistent)_ — Active configuration for spam/flood detection
  - fields: link_account_age_threshold, repetition_window, flood_rate_threshold, preset_mode
- **Action log entry** _(retention: persistent)_ — Record of moderation actions with evidence
  - fields: timestamp, target_user, action_type, reason, actor_type, evidence_snippet
- **Trusted list** _(retention: persistent)_ — User_ids exempt from automated moderation
  - fields: user_id, added_by, timestamp

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure greeting/rules text
- Adjust moderation rule thresholds
- Manage trusted user list
- Generate activity reports
- Set admin notification preferences

## Notifications

- Admin alerts for automated actions
- Verification timeout notifications
- Periodic summary reports
- Action log updates

## Permissions & privacy

- Admin-only access to moderation commands
- Trusted user list stored securely
- Action logs retain minimal metadata
- User account age checked via Telegram API

## Edge cases

- Users joining and leaving before verification timeout
- Multiple simultaneous rule triggers on single message
- Admin commands without required parameters
- Message rate spikes during group events

## Required tests

- Verify new-member greeting with button interaction
- Test spam detection rule combinations
- Validate admin command privilege checks
- Confirm action log retention policy

## Assumptions

- Default verification timeout of 3 minutes is sufficient for human users
- Balanced preset thresholds provide adequate protection without over-moderation
- Admins will manually review complex moderation cases
