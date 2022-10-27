# YServer Events

Events are sent to the client by messages with type 'event'. See the yserver documentation for more information.

## Universal Player Events

Regardless of the type of game or casino provider, these events are common to all game clients.
They are used for game client logic that is not sensitive to game-specific or provider-specific behavior.

### accounting-round-end

Triggered when an accounting round ends

#### Event Data

No data passed with this event.

---

### accounting-round-start

Triggered when a new accounting round is started

#### Event Data

No data passed with this event.

---

### action

Triggered when an action should be executed on the game client

#### Event Data

No data passed with this event.

---

### action-enabled

Triggered when the enabled state of an action or subaction changes

#### Event Data

No data passed with this event.

---

### authenticated

Triggered when the player successfully authenticates

#### Event Data

No data passed with this event.

---

### authentication-required

Triggered when authentication is required

#### Event Data

No data passed with this event.

---

### balance-update

Triggered when your balance changes

#### Event Data

No data passed with this event.

---

### bet

Triggered when a bet is made to the accounting service

#### Event Data

No data passed with this event.

---

### bet-status

Triggered when the bet status of a game client changes, only forwarded to any spectators, not sent to the client that had the bet status changed

#### Event Data

No data passed with this event.

---

### chat

Triggered when a new chat message should be displayed

#### Event Data

No data passed with this event.

---

### client-closing

Triggered a moment before the game client would close - only forwarded to any spectators, not triggered for the game client itself

#### Event Data

No data passed with this event.

---

### client-event

Called for every event that occurs on a spectated client

#### Event Data

No data passed with this event.

---

### connection-lost

Triggered when the server loses connection with the game client

#### Event Data

No data passed with this event.

---

### game-round-end

Triggered when a game round ends

#### Event Data

No data passed with this event.

---

### game-round-start

Triggered when a new game round is started

#### Event Data

No data passed with this event.

---

### help-screen-info

Triggered when the game client has defined what information is visible on the help screens

#### Event Data

No data passed with this event.

---

### jackpot-won

Triggered when you win a jackpot!

#### Event Data

No data passed with this event.

---

### lock

Triggered when the game client should lock the play area

#### Event Data

No data passed with this event.

---

### log

Triggered when the game client logs something

#### Event Data

No data passed with this event.

---

### payout

Triggered when funcs are withdrawn from the player account

#### Event Data

No data passed with this event.

---

### reconnected

Triggered when a game client reconnects to an existing player

#### Event Data

No data passed with this event.

---

### resource-loaded

Triggered when a resource is finished uploading to the server

#### Event Data

No data passed with this event.

---

### save-point

Triggered when the saved game round snapshot changes

#### Event Data

No data passed with this event.

---

### session-expired

Triggered when your session expires

#### Event Data

No data passed with this event.

---

### state-changed

Triggered when the state of a game client changes, only forwarded to any spectators, not sent to the client that had the state changed

#### Event Data

No data passed with this event.

---

### support-actions

Triggered when supported actions were registered by the game client - only forwarded to any spectators, not triggered for the game client itself

#### Event Data

No data passed with this event.

---

### unlock

Triggered when game client should unlock the play area

#### Event Data

No data passed with this event.

---

### void

Triggered the game round you are in is voided

#### Event Data

No data passed with this event.

---

### welcome

Triggered when everything has been initialized and you are safe to do requests

#### Event Data

No data passed with this event.

---

### win

Triggered when a win is confirmed by the accounting service

#### Event Data

No data passed with this event.

---

### module-event

Trigered for every event on this host - only forwarded to and viewers of this module

#### Event Data

No data passed with this event.

---

### player-joined

Triggered when a new player joins this game host

#### Event Data

No data passed with this event.

---

### player-left

Triggered when a pleyer leaves this game host

#### Event Data

No data passed with this event.

---

## Game-Specific Player Events

Each game type is a little different, so there are some events which are only trigegred by that game host type.

## Roulette

### config-changed

If the configuration of the host changes, this is fired

#### Event Data

No data passed with this event.

---

### game-state-changed

Sent when a variable describing the state of the game changes

#### Event Data

No data passed with this event.

---

### history

Triggered when the list of history numbers changes

#### Event Data

No data passed with this event.

---

### jackpot-update

Sent when the pot value of the lucky number jackpot changes

#### Event Data

No data passed with this event.

---

### jackpot-was-won

Triggered when some other player (not you) won the jackpot

#### Event Data

No data passed with this event.

---

### stakeChanged

Triggered when the selected stake changes

#### Event Data

No data passed with this event.

---

### win-report

Triggered when a game ends to report wins that have occured

#### Event Data

No data passed with this event.

---

## GameartSlotGame

No special events for this host

## VirtualRoulette

### stakeChanged

Triggered when the selected stake changes

#### Event Data

No data passed with this event.

---

## GameartTableGame

No special events for this host

## Provider-Specific Player Events

Each casino provider works differently and can have special features, so there are some events which are only triggered by that provider type.

## nano

No special events for this provider

## HG4

No special events for this provider

## berghain

No special events for this provider

## GameHub

No special events for this provider

