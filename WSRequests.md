# WebSocket Requests

WebSocket requests on the YServer are executed over the imaxa protocol as messages with type 'request'. See the protocol documentation section for more information.

## Universal Player Requests

Regardless of the type of game or casino provider, these requests are always available to the game client.
They are used for game client logic that is not sensitive to game-specific or provider-specific behavior.

### action-enabled

Change the enabled status of a registered action (landbase integration)

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
{
   "action" : "", // <string> [optional] The name of the subaction to set enabled (or empty for the root action)
   "enabled" : , // <bool> [REQUIRED] True/false if this (sub)action is executable
   "type" :  // <string> [REQUIRED] The type of this action [The type of this action]
} // <object> [REQUIRED] 
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
[] // <array> The balance of the player
```

---

### chat

Post a message in chat [request is unavailable if player is locked in UserLock or Blocked or Handpay or LockProvider or LockHost or System or Closing]

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
{
   "message" : , // <string> [REQUIRED] The message to send
   "to" : "" // <string> [optional] The ID of the player this is meant for (ommit this for broadcast messages)
} // <object> [REQUIRED] 
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
{
   "flags" : 
   [
       // <string> [REQUIRED] A flag describing game state
   ], // <array> [optional] A list of flags
   "state" : , // <string> [REQUIRED] The state of the game right now [The state of the game right now]
   "subState" : 0 // <uint> [optional] The sub-state ID
} // <object> [REQUIRED] 
```

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
[
   // <object> 
   {
      "aid" : , // <string> The accounting round ID of the round this game is in
      // <object> 
      "balance" : 
      {
         "after" : [], // <array> The balance after the round
         "before" : [] // <array> The balance before the round
      },
      "data" : , // <any> [optional] Extra data attributed to the round
      "endT" : , // <uint> The end timestamp
      "endpoint" : , // <string> Endpoint of this round
      "gameType" : , // <string> The type of game this round is for [The type of game this round is for]
      "gid" : , // <string> Game round ID
      "host" : , // <string> Game host UID
      "randoms" : 
      [
          // <real> Floating point or integer which was drawn
      ], // <array> List of random numbers which were drawn in this round
      "startT" : , // <uint> The start timestamp
      "totalBet" : [], // <array> The total bet in this round
      "totalWin" : [], // <array> The total win in this round
      "transactions" : 
      [
         // <object> 
         {
            "aid" : , // <string> The accounting round ID in which this transaction occured
            "credits" : [], // <array> The transacted amount
            "data" : , // <any> [optional] Extra data attributed to this transaction
            "gid" : , // <string> The game round ID in which this transaction occured
            "id" : , // <string> Transaction id
            "result" : , // <string> The result of the transaction [The result of the transaction]
            "source" :  // <string> What is responsible for this transaction [What is responsible for this transaction]
         }
      ], // <array> Array of transactions in this round
      "void" :  // <bool> True if the game was void, false otherwise
   }
] // <array> List of actions
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
{
   "msg" : , // <string> [REQUIRED] The message to log
   "verbosity" : "Normal" // <string> [optional] The verbosity of this log message [The verbosity of this log message]
} // <object> [REQUIRED] 
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
{
   "body" : , // <any> [optional] A value describing help screen layout (forwarded to clients asking for the help screen information of this game)
   "resources" : 
   [
       // <string> [REQUIRED] The name of this resource, as used in 'body'
   ] // <array> [optional] A list of resource names to be hashed (prepare for upload)
} // <object> [REQUIRED] 
```

#### Return Value

Returns a map of registered resource names (name:resourceID) as a JSON object

```javascript
{
   "exampleMember" :  // <string> The resourceID that can be used to upload resources to the server, see HTTP requests section
} // <object> Map of registered resources
```

---

### station-lock

Lock this play station [request is unavailable if player is locked in Blocked or AwaitAuth or Handpay or LockProvider or LockHost or System or Closing]

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
{
   "lock" : , // <bool> [REQUIRED] True to lock the station, false to request an unlock
   "message" : "" // <string> [optional] The message to lock the client with
} // <object> [REQUIRED] 
```

#### Return Value

Does not return anything (null)

---

### support-actions

Register supported actions for this game (landbase integration)

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
[
   // <object> [REQUIRED] 
   {
      "actions" : 
      [
         // <object> [REQUIRED] 
         {
            "enabled" : true, // <bool> [optional] True/false if this subaction is executable right now
            "name" : , // <string> [REQUIRED] The name of this subaction
            // <object> [optional] The schema which action trigger contexts should abide by
            "requiredContext" : 
         }
      ], // <array> [optional] List of sub-actions
      "enabled" : true, // <bool> [optional] True/false if this action is executable right now
      "onlySubactions" : false, // <bool> [optional] True to disallow the root action
      // <object> [optional] The schema which action trigger contexts should abide by
      "requiredContext" : ,
      "type" :  // <string> [REQUIRED] The type of this action [The type of this action]
   }
] // <array> [REQUIRED] List of actions supported by the game
```

#### Return Value

Returns the total number of actions successfully parsed

```javascript
 // <uint> Number of actions
```

---

### authenticate

Attempt to authenticate with the provider [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Returns the authentication result

```javascript
{
   "sid" : , // <string> The created session for the player
   "success" :  // <bool> True/false indicating if the authentication succeeded
} // <object> 
```

---

### set-param

Persistently save a player setting [request is unavailable if player is locked in any lock level]

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
{
   "name" : , // <string> [REQUIRED] The name of the parameter to set
   "value" :  // <string> [REQUIRED] The value of the parameter
} // <object> [REQUIRED] 
```

#### Return Value

Returns the new value of the parameter

```javascript
 // <any> Value of the parameter (same type as provided in request parameters)
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
0 // <uint> [optional] The i-th page of players (or get all players if == 0)
```

#### Return Value

Returns a list of players on this host (keys are player IDs)

```javascript
{
   // <object> 
   "exampleMember" : 
   {
      "chat" : , // <bool> True/false if you can chat with this player
      "nickname" : , // <string> The display name of this player
      "number" :  // <uint> ID of this player
   }
} // <object> List of players
```

---

### players

Get the number of players on this game host [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Returns the current number of players on this game host

```javascript
 // <uint> Current number of players on the host
```

---

### state

Get the current game state [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Does not return anything (null)

---

## Game-Specific Player Requests

Each game type is a little different, so there are some requests which are only accessible by that game type.
Additionally, the type of the response to the ***bet*** request (see above) is also determined by the type of game.

## Roulette

### config

Return the game configuration for this host [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Does not return anything (null)

---

### history

Return the history of winning numbers [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Does not return anything (null)

---

### select-stake

Select a stake [request is unavailable if player is locked in UserLock or Blocked or AwaitAuth or Handpay or LockProvider or System or Closing]

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
 // <uint> [REQUIRED] The index of the stake to select
```

#### Return Value

Does not return anything (null)

---

### vote-start

Vote for the early start of a game [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Does not return anything (null)

---

## GameartSlotGame

### action

Perform a Gameart action

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
[
   // <object> [REQUIRED] 
   {
      "action" : , // <string> [REQUIRED] Name of the action
      "context" :  // <any> [REQUIRED] The context of this action
   }
] // <array> [REQUIRED] An array of actions to perform
```

#### Return Value

If actions were executed successfully

```javascript
[
   // <object> 
   {
      "context" : , // <any> Data of the event
      "event" :  // <string> Name of the triggered event
   }
] // <array> Array of triggered events
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

No special requests for this host

## GameartTableGame

### action

Perform a Gameart action

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
[
   // <object> [REQUIRED] 
   {
      "action" : , // <string> [REQUIRED] Name of the action
      "context" :  // <any> [REQUIRED] The context of this action
   }
] // <array> [REQUIRED] An array of actions to perform
```

#### Return Value

If actions were executed successfully

```javascript
[
   // <object> 
   {
      "context" : , // <any> Data of the event
      "event" :  // <string> Name of the triggered event
   }
] // <array> Array of triggered events
```

---

### config

Get the game configuration [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Does not return anything (null)

---

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

