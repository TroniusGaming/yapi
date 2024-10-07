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
[
   {
      "aid": "",
      "balance": {
         "after": [],
         "before": []
      },
      "data": null,
      "denom": 0.0,
      "endT": 0,
      "endpoint": "",
      "grounds": [
         ""
      ],
      "startT": 0,
      "totalBet": [],
      "totalWin": [],
      "transactions": [
         {
            "aid": "",
            "credits": [],
            "data": null,
            "gid": "",
            "id": "",
            "result": "",
            "source": ""
         }
      ]
   }
]
```

---

### action-enabled

Change the enabled status of a registered action (landbase integration)

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
{
   "action": "",
   "enabled": false,
   "type": ""
}
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
[]
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
{
   "message": "",
   "to": ""
}
```

#### Return Value

Returns true if the message was sent, false if it was not (if it's a targeted message and the target player could not be found or has chat disabled)

```javascript
false
```

---

### client-status

Update the state of this client

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
{
   "flags": [
      ""
   ],
   "state": "",
   "subState": 0
}
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
[
   {
      "aid": "",
      "balance": {
         "after": [],
         "before": []
      },
      "data": null,
      "endT": 0,
      "endpoint": "",
      "gameType": "",
      "gid": "",
      "host": "",
      "randoms": [
         0.0
      ],
      "startT": 0,
      "totalBet": [],
      "totalWin": [],
      "transactions": [
         {
            "aid": "",
            "credits": [],
            "data": null,
            "gid": "",
            "id": "",
            "result": "",
            "source": ""
         }
      ],
      "void": false
   }
]
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
   "msg": "",
   "verbosity": "Normal"
}
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
   "body": null,
   "resources": [
      ""
   ]
}
```

#### Return Value

Returns a map of registered resource names (name:resourceID) as a JSON object

```javascript
{
   "exampleMember1": ""
}
```

---

### set-param

Persistently save a player setting [request is unavailable if player is locked in any lock level]

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
{
   "name": "",
   "value": ""
}
```

#### Return Value

Returns the new value of the parameter

```javascript
null
```

---

### station-lock

Lock this play station [request is unavailable if player is locked in UserClose or Blocked or AwaitAuth or Handpay or LockProvider or LockHost or GameInProgress or System or SystemClose]

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
{
   "lock": false,
   "message": ""
}
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
   {
      "actions": [
         {
            "enabled": true,
            "name": "",
            "requiredContext": null
         }
      ],
      "enabled": true,
      "onlySubactions": false,
      "requiredContext": null,
      "type": ""
   }
]
```

#### Return Value

Returns the total number of actions successfully parsed

```javascript
0
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
   "sid": "",
   "success": false
}
```

---

### bet

Place a new bet or change an existing one (game-specific, see game specific section for more information)

#### Parameters

No request parameters required.

#### Return Value

Different game types return different values, see the game-specific requests section

```javascript
null
```

---

### player-list

Get the list of players on this game host [request is unavailable if player is locked in any lock level]

#### Parameters

Request parameters are **optional** and are parsed with the following schema:

```javascript
0
```

#### Return Value

Returns a list of players on this host (keys are player IDs)

```javascript
{
   "exampleMember1": {
      "chat": false,
      "nickname": "",
      "number": 0
   }
}
```

---

### players

Get the number of players on this game host [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Returns the current number of players on this game host

```javascript
0
```

---

### state

Get the current game state [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

If successful

```javascript
null
```

---

## Game-Specific Player Requests

Each game type is a little different, so there are some requests which are only accessible by that game type.
Additionally, the type of the response to the ***bet*** request (see above) is also determined by the type of game.

## Roulette

The parameters of **bet** requests on this game host should conform with the following schema:
```javascript
{
   "betPresets": {
      "exampleMember1": 0
   },
   "bets": [
      {
         "X": 0,
         "Y": 0,
         "amount": 0
      }
   ],
   "gameID": 0
}
```

The return value of **bet** requests on this game host look like this:

```javascript
null
```

### config

Return the game configuration for this host [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

If successful

```javascript
{}
```

---

### history

Return the history of winning numbers [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

If successful

```javascript
[
   [
      0,
      0
   ]
]
```

---

### select-stake

Select a stake [request is unavailable if player is locked in UserLock or UserClose or Blocked or AwaitAuth or Handpay or LockProvider or GameInProgress or System or SystemClose]

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
0
```

#### Return Value

If successful

```javascript
{
   "betTypes": {
      "12to1": {
         "max": 0,
         "min": 0,
         "multiple": 1
      },
      "18to1": {
         "max": 0,
         "min": 0,
         "multiple": 1
      },
      "2to1": {
         "max": 0,
         "min": 0,
         "multiple": 1
      },
      "36to1": {
         "max": 0,
         "min": 0,
         "multiple": 1
      },
      "3to1": {
         "max": 0,
         "min": 0,
         "multiple": 1
      },
      "6to1": {
         "max": 0,
         "min": 0,
         "multiple": 1
      },
      "7to1": {
         "max": 0,
         "min": 0,
         "multiple": 1
      },
      "9to1": {
         "max": 0,
         "min": 0,
         "multiple": 1
      },
      "le-finali-3": {
         "max": 0,
         "min": 0,
         "multiple": 1
      },
      "le-finali-4": {
         "max": 0,
         "min": 0,
         "multiple": 1
      },
      "le-finali-5": {
         "max": 0,
         "min": 0,
         "multiple": 1
      },
      "les-figures": {
         "max": 0,
         "min": 0,
         "multiple": 1
      },
      "tout-va": {
         "multiplier": 81
      },
      "tout-va-3": {
         "multiplier": 145
      },
      "tout-va-5": {
         "multiplier": 50
      }
   },
   "chipValues": [
      1,
      2,
      5,
      10,
      20
   ],
   "maxBet": 0,
   "maxNumBets": 0,
   "minBet": 0,
   "tableWinLimit": 0
}
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
[
   {
      "action": "bet",
      "context": []
   },
   {
      "action": "play",
      "context": null
   }
]
```

The return value of **bet** requests on this game host look like this:

```javascript
null
```

### action

Perform a Gameart action

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
[
   {
      "action": "",
      "context": null
   }
]
```

#### Return Value

If actions were executed successfully

```javascript
[
   {
      "context": null,
      "event": ""
   }
]
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
{
   "betPresets": {
      "exampleMember1": 0
   },
   "bets": [
      {
         "X": 0,
         "Y": 0,
         "amount": 0
      }
   ]
}
```

The return value of **bet** requests on this game host look like this:

```javascript
null
```

No special requests for this host

## GameartTableGame

The parameters of **bet** requests on this game host should conform with the following schema:
```javascript
[
   {
      "action": "",
      "context": null
   }
]
```

The return value of **bet** requests on this game host look like this:

```javascript
null
```

### action

Perform a Gameart action

#### Parameters

Request parameters are **required** and are parsed with the following schema:

```javascript
[
   {
      "action": "",
      "context": null
   }
]
```

#### Return Value

If actions were executed successfully

```javascript
[
   {
      "context": null,
      "event": ""
   }
]
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
{
   "type": ""
}
```

The return value of **bet** requests on this game host look like this:

```javascript
null
```

No special requests for this host

## VirtualBaccarat

The parameters of **bet** requests on this game host should conform with the following schema:
```javascript
{
   "betPresets": {
      "exampleMember1": 0
   },
   "bets": [
      0
   ]
}
```

The return value of **bet** requests on this game host look like this:

```javascript
null
```

### free-game

Play game without bet [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Does not return anything (null)

---

### shuffleCards

Player requested reshuffeling of cards [request is unavailable if player is locked in any lock level]

#### Parameters

No request parameters required.

#### Return Value

Does not return anything (null)

---

## Baccarat

The parameters of **bet** requests on this game host should conform with the following schema:
```javascript
{
   "betPresets": {
      "exampleMember1": 0
   },
   "bets": [
      0
   ]
}
```

The return value of **bet** requests on this game host look like this:

```javascript
null
```

No special requests for this host

## Evolution

The parameters of **bet** requests on this game host should conform with the following schema:
```javascript
null
```

The return value of **bet** requests on this game host look like this:

```javascript
null
```

### credit

Credit result of a game (win or loss - loss has credit amout 0)

#### Parameters

No request parameters required.

#### Return Value

Does not return anything (null)

---

### void

Void a game round

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

