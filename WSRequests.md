# WebSocket Requests

WebSocket requests on the YServer are executed over the imaxa protocol as messages with type 'request'. See the protocol documentation section for more information.

## Universal Player Requests

Regardless of the type of game or casino provider, these requests are always available to the game client.
They are used for game client logic that is not sensitive to game-specific or provider-specific behavior.

### accounting-round-history

Get the history of player accounting rounds [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Returns a list of accounting round snapshots - played rounds of this player

```javascript
 // <array> List of actions
```

---

### action-enabled

Change the enabled status of a registered action (landbase integration)

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
 // <object> [REQUIRED] 
```

#### Return Value

Does not return anything (null)

---

### balance

Get the current balance

#### Parameters

No request parameters required.

#### Return Value

Returns the current credit balance of the player

```javascript
 // <array> The balance of the player
```

---

### cancel-exit

Cancel a pending user close

#### Parameters

No request parameters required.

#### Return Value

Does not return anything (null)

---

### chat

Post a message in chat [request is unavailable if player is locked in UserLock or UserClose or Blocked or Handpay or LockProvider or LockHost or GameInProgress or System or SystemClose]

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
 // <object> [REQUIRED] 
```

#### Return Value

Returns true if the message was sent, false if it was not (if it's a targeted message and the target player could not be found or has chat disabled)

```javascript
 // <bool> True/false indicating if the message was sent
```

---

### client-status

Update the state of this client

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
 // <object> [REQUIRED] 
```

#### Return Value

Does not return anything (null)

---

### exit

Signal a user-initiated close

#### Parameters

No request parameters required.

#### Return Value

Does not return anything (null)

---

### game-round-history

Get the history of player game rounds [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Returns a list of game round snapshots - played rounds of this player

```javascript
 // <array> List of actions
```

---

### goodbye

Signal that the client will leave the server

#### Parameters

No request parameters required.

#### Return Value

Does not return anything (null)

---

### log

Write a log [request is unavailable if player is locked in Blocked]

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
 // <object> [REQUIRED] 
```

#### Return Value

Does not return anything (null)

---

### logout

Log out of this account (try to invalidate this session on the provider side) [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Does not return anything (null)

---

### payout

Make a payout request on the provider (if possible) [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Does not return anything (null)

---

### set-help-screens

Set the help screen information (landbase integration)

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
 // <object> [REQUIRED] 
```

#### Return Value

Returns a map of registered resource names (name:resourceID) as a JSON object

```javascript
 // <object> Map of registered resources
```

---

### set-param

Persistently save a player setting [request is unavailable if player is locked in any lock level]

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
 // <object> [REQUIRED] 
```

#### Return Value

Returns the new value of the parameter

```javascript
 // <any> Value of the parameter (same type as provided in request parameters)
```

---

### station-lock

Lock this play station [request is unavailable if player is locked in UserClose or Blocked or AwaitAuth or Handpay or LockProvider or LockHost or GameInProgress or System or SystemClose]

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
 // <object> [REQUIRED] 
```

#### Return Value

Does not return anything (null)

---

### support-actions

Register supported actions for this game (landbase integration)

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
 // <array> [REQUIRED] List of actions supported by the game
```

#### Return Value

Returns the total number of actions successfully parsed

```javascript
 // <uint> Number of actions [ | values must be in intervals of 1.000000]
```

---

### authenticate

Attempt to authenticate with the provider [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Returns the authentication result

```javascript
 // <object> 
```

---

### bet

Place a new bet or change an existing one (game-specific, see game specific section for more information)

#### Parameters

No request parameters required.

#### Return Value

Different game types return different values, see the game-speciic requests section

```javascript
 // <any> [optional] 
```

---

### player-list

Get the list of players on this game host [request is unavailable if player is locked in any lock level]

#### Parameters

Request parameters are **optional** and are parsed with the following schema:

```javascript
0 // <uint> [optional] The i-th page of players (or get all players if == 0) [ | values must be in intervals of 1.000000]
```

#### Return Value

Returns a list of players on this host (keys are player IDs)

```javascript
 // <object> List of players
```

---

### players

Get the number of players on this game host [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Returns the current number of players on this game host

```javascript
 // <uint> Current number of players on the host [ | values must be in intervals of 1.000000]
```

---

### state

Get the current game state [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

If successfull

```javascript
 // <any> [optional] The state of the game
```

---

## Game-Specific Player Requests

Each game type is a little different, so there are some requests which are only accessible by that game type.
Additionally, the type of the response to the ***bet*** request (see above) is also determined by the type of game.

## Roulette

The parameters of **bet** requests on this game host should conform with the following schema:
```javascript
 // <object> [REQUIRED] 
```

The return value of **bet** requests on this game host look like this:

```javascript
 // <any> [optional] 
```

### config

Return the game configuration for this host [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

If successfull

```javascript
 // <object> The config of this roulette game
```

---

### history

Return the history of winning numbers [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

If successful

```javascript
 // <array> List of games played on this host
```

---

### select-stake

Select a stake [request is unavailable if player is locked in UserLock or UserClose or Blocked or AwaitAuth or Handpay or LockProvider or GameInProgress or System or SystemClose]

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
 // <uint> [REQUIRED] The index of the stake to select [ | values must be in intervals of 1.000000]
```

#### Return Value

If successfull

```javascript
 // <object> The configuration of the stake
```

---

### vote-start

Vote for the early start of a game [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Does not return anything (null)

---

## GameartSlotGame

The parameters of **bet** requests on this game host should conform with the following schema:
```javascript
 // <array> [REQUIRED] List of actions to execute
```

The return value of **bet** requests on this game host look like this:

```javascript
 // <any> [optional] 
```

### action

Perform a Gameart action

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
 // <array> [REQUIRED] An array of actions to perform
```

#### Return Value

If actions were executed successfully

```javascript
 // <array> Array of triggered events
```

---

### config

Get the game configuration [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Does not return anything (null)

---

## VirtualRoulette

The parameters of **bet** requests on this game host should conform with the following schema:
```javascript
 // <object> [REQUIRED] 
```

The return value of **bet** requests on this game host look like this:

```javascript
 // <any> [optional] 
```

No special requests for this host

## GameartTableGame

The parameters of **bet** requests on this game host should conform with the following schema:
```javascript
 // <array> [REQUIRED] List of actions to execute
```

The return value of **bet** requests on this game host look like this:

```javascript
 // <any> [optional] 
```

### action

Perform a Gameart action

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
 // <array> [REQUIRED] An array of actions to perform
```

#### Return Value

If actions were executed successfully

```javascript
 // <array> Array of triggered events
```

---

### config

Get the game configuration [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Does not return anything (null)

---

## Blackjack

The parameters of **bet** requests on this game host should conform with the following schema:
```javascript
 // <object> [REQUIRED] 
```

The return value of **bet** requests on this game host look like this:

```javascript
 // <any> [optional] 
```

No special requests for this host

## Provider-Specific Player Requests

Each casino provider works differently and can have special features, so there are some requests which are only accessible when using that provider type.

## nano

No special requests for this provider

## HG4

No special requests for this provider

## berghain

No special requests for this provider

## GameHub

No special requests for this provider

