# TripSplit — Bot specification

**Archetype:** finance

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram group bot for tracking and settling shared trip expenses. Organizers create trips, add participants, and manage balances. Participants can record expenses with custom splits, and the bot suggests minimal settlement transfers. All data is private to participants, with immutable expense records and deterministic rounding to ensure accurate balances.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Friends
- Small travel groups
- Roommates
- Small event groups

## Success criteria

- Trip expenses are accurately tracked and settled with minimal payment transfers
- All participants can view real-time balances and expense history
- Settlements are confirmed securely with one-tap private confirmations
- Trip data remains private and immutable

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu
- **/newtrip** (command, actor: user, command: /newtrip) — Create a new trip with title and currency
- **/jointrip** (command, actor: user, command: /jointrip) — Join an existing trip
- **/leavetrip** (command, actor: user, command: /leavetrip) — Leave the current trip
- **/expense** (command, actor: user, command: /expense) — Record a new expense with amount, payer, description, and split method
- **/balance** (command, actor: user, command: /balance) — View current balances and suggested settlements
- **/detailedbalance** (command, actor: user, command: /detailedbalance) — View detailed expense history (organizer-only)
- **/settle** (command, actor: user, command: /settle) — Record a settlement between two participants
- **/closetrip** (command, actor: user, command: /closetrip) — Close the current trip (organizer-only)

## Flows

### Create Trip
_Trigger:_ /newtrip

1. Organizer provides title and currency
2. Bot records organizer and prepopulates participants with current group members
3. Organizer can add/remove participants

_Data touched:_ Trip

### Join/Leave Trip
_Trigger:_ /jointrip or /leavetrip

1. User sends command to join/leave
2. Bot updates participant list and status
3. Leaving marks participant as inactive for future expenses

_Data touched:_ Participant

### Record Expense
_Trigger:_ /expense

1. User provides amount, payer, description, and split method
2. Bot calculates shares and updates balances
3. Bot confirms expense in group chat with updated balances

_Data touched:_ Expense, Balance

### View Balances
_Trigger:_ /balance

1. Bot displays net balances per participant
2. Bot suggests minimal settlement transfers
3. Organizer can view detailed expense history

_Data touched:_ Balance

### Settle Payment
_Trigger:_ /settle

1. User records a settlement between two participants
2. Bot sends private confirmation to both parties
3. Settlement is applied after confirmation

_Data touched:_ Settlement, Balance

### Close Trip
_Trigger:_ /closetrip

1. Organizer confirms trip closure
2. Trip is marked as closed
3. Participants can still view data but no new expenses are allowed

_Data touched:_ Trip

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Trip** _(retention: persistent)_ — A group trip with shared expenses
  - fields: title, currency, organizer, participant list, creation time, status
- **Participant** _(retention: persistent)_ — A user in the trip
  - fields: Telegram user id, display name, join/leave timestamps, contact info
- **Expense** _(retention: persistent)_ — A recorded expense in the trip
  - fields: payer, amount, currency, timestamp, description, participant shares, immutable record id, receipt URL
- **Balance** _(retention: persistent)_ — Net balance for each participant
  - fields: participant id, net balance, trip id
- **Settlement** _(retention: persistent)_ — A recorded payment between participants
  - fields: payer, payee, amount, timestamp, note

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Create and manage trips
- Add/remove participants
- View detailed expense history
- Close trips
- Approve expense corrections

## Notifications

- Expense confirmation in group chat
- Balance updates in group chat
- Private settlement confirmations
- Trip closure notifications

## Permissions & privacy

- Only trip participants can view trip data
- Private confirmations are sent only to involved users
- Expense details are visible to all participants
- Organizer has full read access to trip data

## Edge cases

- Rounding of cents with deterministic assignment
- Handling mid-trip joins/leaves
- Immutable expense records with correction tracking
- Minimal settlement transfers calculation
- Private vs group chat message separation

## Required tests

- Verify expense recording and balance updates
- Test settlement confirmation flow
- Validate minimal payment calculation
- Confirm private message delivery for sensitive actions
- Test trip closure and reopening

## Assumptions

- Currency is set per trip by organizer
- Default split is equal among active participants
- Expense entries are immutable with append-only corrections
- One-tap private confirmations prevent accidental changes
- New joiners are not retroactively charged unless organizer specifies
