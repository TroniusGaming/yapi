# Y Server Documentation

***Version 2.0.12***

## Communication concept

When a game client is started, it will be forwarded a few query parameters that it can rely on:

- `server` - The address of the YServer the client should connect to
- `locale` - The selected locale (en, it, es, de, ...)
- `secure` - A boolean indicating if the websocket connection should be secure (`wss://`) or insecure (`ws://`)

Connecting to the YServer is the first thing the client should do when starting up (idealy before loading all game assets).

After initiating the connection, the game client can expect the following outcomes:

- No response is received or host is unreachable (this would be due to bad setup or the internet connection on the client or server not working)
- An `error` message is sent with a `contextID` of 0 (this means the connection failed, there are details about the failure in this error message)
- An `init` message is sent with a `contextID` of 0 (this means the connection succeeded, with loads of data required for the game client to load properly - like denomination, currency, player name, etc.)

All communication between the server and client is done on the websocket connection and follows a shared set of rules called the Y protocol. This defines how messages are structured and how to handle cases where a request-response communication model is desired (classic HTTP).

All WebSocket frames must be the TEXT type.

Once a connection request is received by the server, it will attempt to find the account that owns the given session. If everything is ok, the server will send a message with `"type": "init"` that contains all the information required to set up a game instance. If something goes wrong during the connection process, the server will send a descriptive error of the problem (`"type": "error"`) and close the connection.
After the initialization message, the client may communicate with the server and perform requests. Messages between the server and client contain a `contextID` member, which is used to pair responses with requests correctly. (eg. a request sent by the client with `"contextID": 1` will receive a response message with the same `contextID`).

## Y Protocol

All messages sent over the WebSocket connection must be valid stringified JSON objects and sent with a TEXT frame type.

NOTE: Balances reported by the Y Server are always in credits and are **arrays** of integers instead of simple integers.
This is because there are different types of credits that some games may want to display differently (eg. promotional credits, or specialized types for different markets like Spain). For now, we have 6 credit types, though in pretty much all cases, you can sum these up together and do all your game logic with the single summed number.
If you need to know, these are the credit types in the array by index:

- 0 | Non-restricted credits (promotional credits, but they can be paid out)
- 1 | Restricted credits (promotional credits, but they cannot be paid out and must be played)
- 2 | Cashable credits (regular credits, which can be paid out normally)
- 3 | Fichas (special Spain jurisdiction credits)
- 4 | Reserva (special Spain jurisdiction credits)
- 5 | Banco (special Spain jurisdiction credits)

### Base message schema

```javascript
{
    "type": string, // determines the type of the "payload" member
    "sentTS": uint64, // millisecond EPOCH timestamp of when the message was sent
    "contextID": uint32, // a number used as a reference to know if this message is in response to a previous one
    "payload": any // different for every message "type"
}
```

Here is what each of those members mean:

#### `type`

The type of the message, which is a discrete value, that also determines how the `payload` member is structured.
Messages sent from the server to the client have the following types - explained in detail in the ***Server Messages*** section:

- `error` - Message payload describes an error which occured
- `init` - Message type sent when the connection with the game client is successfully initialized, payload contains game client setup information
- `response` - Sent in response to a `request` message by the game client, payload contains the actual response.
- `event` - An event has occured that the game client might need to react to, details about the event are in the payload
- `ping` - A ping message sent to see if the client is still alive
- `cookie` - A request to set or remove a cookie from the browser (optional)
- `question` - A request sent when the server wants to ask the user something and requires user input (only needed in some games)

When the client sends messages, they can only be one of the following types - explained in detail in the ***Client messages*** section:

- `request` - Perform an action on the server (eg. bet, get-balance, ...), the payload describes which request to execute and with what parameters
- `pong` - Client should respond with a message of this type after receiving a `ping` message from the server
- `response` - When a `question` is received from the server and the user has given their input, this is sent to confirm the user's selection

#### `sentTS`

All messages must contain a millisecond EPOCH timestamp of when they were sent. If the timestamp deviates significantly from the current time, the server will respond to the sent message with a message of type `error`.

#### `contextID`

`"contextID": 0` is reserved for the response to the initial connection request and error messages where the incoming message is malformed (the `contextID` sent by the client can't be found in the message).

A **client-initiated** message is a message sent by the client which is not a reply to a server message.
Similarly, a **server-initiated** message is a message sent by the server which is not a reply to a client message.
To easily distinguish between server-initiated and client-initiated messages, an **odd/even** `contextID` rule is used.

- Client-initiated messages must use an **odd numbered** `contextID` (1, 3, 5, ...)
- Server-initiated messages use an **even numbered** `contextID` (0, 2, 4, ...)

Starting with `1`, the client must increment the context ID by `2` (`1, 3, 5, ...`) for every unique message it is sending to the server. If the client is replying to a message that was sent by the server, it must use the `contextID` that the server provided.
The server follows the same rule, except it is using an even context ID (starting with `0` for the initialization message, `2, 4, 6`).

For example, here's what communication can look like at the beginning:

- Client opens a new connection to the server
- Server sends an message of type `init` with `contextID=0`
- Client sets internal values based on that massage like denomination, currency, player name that it finds in the payload of that message
- Client executes a `request` on the server with `contextID=1`
- Server responds with a message of type `response` or `error` and with a `contextID=1`
- Server sends a message of type `event` with `contextID=2` for something which happened (eg. balance changed)
- Client doesn't respond to events, it just processes them
- Server sends a message of type `ping` with `contextID=4`
- Client responds with message of type `pong` with `contextID=4`

## Server Messages

### Response

`response` are message types sent by the server which are exclusively sent as responses to client messages of type `request`.
The `response` message type always carries a payload, which changes based on the request that it is responding to.
Details on available requests and their responses are in the ***YServer WebSocket Requests*** documentation.

### Errors

In adition to `response`, `error` is also a type that is sent only in response to a client message. If something goes wrong during message processing, or if the request being executed throws an error, the server will send a message of `error` type as a response which will have the same `contextID` as the client message which it is responding to. The `payload` of an error message is as follows:

```javascript
{
    "code": int, // an error code categorizing the problem
    "type": string, // the type of error (determines what is in the "context" member)
    "message": string, // the error message
    "context": any // determined by "type"
}
```

Error codes:

- `501` - unauthorized
- `502` - bad request
- `503` - missing parameters
- `504` - internal error
- `505` - not ok
- `506` - time out of sync
- `507` - bullshit data
- `508` - no such context
- `509` - connection unavailable
- `510` - bad reconnect
- `511` - banned
- `512` - session timed out

#### Examples

If a client sends a `request` that is correctly formatted but perhaps has incorrect parameters or something happens while executing the request, the `error` sent by the server will always have an error code of **505**(not ok).

When the client first opens a connection to the server, the server may send an error message to the client and close the connection if something goes wrong while creating a new player on the server. The client needs to account for the following possibilities:

- The server is shutting down or it is full, the RNG or accounting service which this player connects to is shutting down or full, the connection URL is incorrect, or the given session is already online - an error with code `509`(connection unavailable) will be sent
- The connection URL refers to a non-existent server endpoint or its authenticity could not be verified, or the player account is blocked - an error with code `501`(unauthorized) will be sent
- Something bad happens on the server that even the developers don't understand - an error with code `504`(internal error) will be sent
- The player's session is not valid - an error with code `512`(session timed out) will be sent

While parsing client messages, several errors can happen that are sent as replies to the message:

- `507` *bullshit data* - The websocket message is not a valid JSON object or it is not a valid message object, or it does not have a valid message type (see ***Base message schema***)
- `508` *no such context* - The `contextID` of the message is a server-initiated one (an even number), but the server never sent a message with this `contextID`
- `506` *time out of sync* - The message received has a `sentTS` that seems to be out of sync with the server. Valid windows are from `now - maxPing` to `now + maxPing/2`, where `maxPing` is something like 2 seconds.
- `504` *internal error* - If I screwed something up and the message wasn't properly handled (shouldn't happen but I'm not perfect)

Keep in mind that every error has a `type` and a `message` and can even have a `context` member. These are just to provide more information of what exactly went wrong and what likely caused it - the `message` being the most usefull.
For a given error `type`, the `context` member will always have the same form, but there are too many types to list all the variants here. I recommended you only look at the error context if you are encountering a specific error `type` often and want more information for debugging.

### Events

The server triggeres events based on changes in the state of the server in general, the game being played or the player account. These are sent as objects in the `payload` of a message with type `event` with the following schema:

```javascript
{
    "name": string, // the name of the event
    "triggered": uint64, // server EPOCH millisecond timestamp of when the event occured
    "data": any // changes based on "name"
}
```

Example event:

```json
{
    "type": "event",
    "sentTS": 184572315,
    "contextID": 46,
    "payload": 
    {
        "name": "player-joined", 
        "triggered": 184572302, 
        "data": 
        {
            "id": "4a261f1903b8129b8a48",
            "nickname": "Joe"
        } 
     }
}
```

Many events are game-type-specific, so they are added as a new type of game is being integrated and are only relevant for that game type.
The few events that are common to all game clients are:

- `player-joined`: a player has joined the game you are in
- `player-left`: a player has left the game you are in
- `void`: the game that you are playing has been void (cancelled) and any bets will be restored to balance
- `balance-update`: account balance changed.
    The `data` looks like this:
    ```javascript
    {
        "source": string,         // can be "game", "game-bonus", "jackpot" or "external"
        "old": Array<uint64>,     // the old balance
        "new": Array<uint64>      // the current balance
    }
    ```
- `lock`: the player account has been locked - the client must disable play and show the sent message until the lock is lifted.
    The `data` looks like this:
    ```javascript
    {
        "reason": string,
        "message": string
    }
    ```
- `unlock`: the player account has been unlocked - the client can enable normal play.
- `payout`: A payout attempt has been made.
    The `data` looks like this:
    ```javascript
    {
        "success": boolean,     // true if the payout succeeded
        "amount": int64         // the amount that was paid out or attempted to be paid out
    }
    ```
- `authentication-required`: the server requests that the client authenticate by providing login credentials. After this event, the client should prompt the player for login credentials and perform an `authenticate` request with the player's input.
    The `data` looks like this:
    ```javascript
    {
        "registration": boolean    // if this is a player registering (true) or just logging in (false)
    }
    ```
- `bet`: a bet was handled by the accounting service.
    The `data` looks like this:
    ```javascript
    {
        "success": boolean,    // true/false if the bet succeeded
        "bet": Array<uint64>,  // array of credit type amounts that was bet
        "message": string      // a message about what happened
    }
    ```
- `game-round-start`: a new game was started.
    The `data` looks like this:
    ```javascript
    {
        "gameroundID": string,    // the ID of the game that is now in progress
        "bet": Array<uint64>      // array of credit type amounts that was bet
    }
    ```
- `game-round-end`: the game you have bet in has ended.
    The `data` looks like this:
    ```javascript
    {
        "message": string,          // a message about what happened
        "won": Array<uint64>,       // array of credit type amounts that the player won
        "balance": Array<uint64>,   // the current balance (after the win is accounted)
        "result": string            // can be "won", "no win"(lost) or "bet restored"(game was void)
    }
    ```
- `chat`: a player has sent a message.
    The `data` looks like this:
    ```javascript
    {
        "msg": string,      // the message to display
        "from": string,     // player ID of the sender
        "private": boolean  // true if sent as a direct message to the client that received this event, false if it's for everyone
    }
    ```
- `action`: a remote action was triggered (landbase integration) - the game should perform the appropriate procedures that this action implies.
    The `data` looks like this:
    ```javascript
    {
        "type": string,   // enum as described under the 'support-actions' request in section __*Requests*__
        "action": string, // optional, if not present OR empty, the general action of type 'type' should be executed
        "context": any    // optional, can be null - any extra information that the action requires
    }
    ```

### Init

The initialization message is sent by the server only uppon successfull connection and only once. It looks like this:

```javascript
{
    "type": "init",
    "sentTS": 184570542,
    "contextID": 0,
    "payload": {
        "provider": {                        // information about the accounting service (in most cases game clients can ignore this as it's not relevant information)
            "id": int,
            "type": int,
            "name": string,
            "domain": string,
            "max-players": uint,             // max number of players allowed on this accounting service
            "players": uint,                 // number of players on this accounting service
            "is-full": bool,
            "ready": bool,
            "authentication-required": bool,
            "config": {...}                 // accounting-service-type-specific object
        },
        "host": {                           // information about the game host
            "id": int,
            "type": int,
            "name": string,
            "domain": string,
            "max-players": uint,            // max number of players allowed on this game host
            "players": uint,                // number of players on this game host
            "is-full": bool,
            "ready": bool,
            "config": {...},                // game-type-specific object
            "multi-session": boolean,
            "access-mode": string,
            "access-list": Array<string>,
            "info": {}                      // game-host-specific object
        },
        "balance": Array<uint64>,           // array of 6 numbers indicating credit amounts for each of 6 credit types
        "game": {
            "state": {},                      // game-specific object
            "unfinished": GameSnapshotObject, // game snapshot of an unfinished game, or null
            "players": {},                    // list of all players (same as response to player-list request, see that for more info)
            "config": {}                      // game-type-specific object
        },
        "account": {
            "UID": string,                  // usefull to know which player is you in player listing, chatting, win reporting of multiple players, etc...
            "name": string,                 // name of the player account (usually something like username)
            "nickname": string,             // nickname of the player account (name to display, can be the same as "name")
            "session": string,              // session ID of this player
            "settings": {},                 // list or persistent settings for this account
            "demo": bool,                   // true/false if this is a demo account (the game can decide to display extra things if this is enabled - like outcome forcing)
            "payout-enabled": bool,         // true/false if payout requests from the game client are enabled for this accounting service (game can show a payout button if this is enabled)
            "logout-supported": bool,       // true/false if logging out of the player account is enabled for this accounting service (game can show a logout button if this is enabled)
            "show-username": bool           // If true, the "name" of the player account should be displayed instead of the "nickname"
        },
        "denomination": double,             // The value of a single credit in units of "currency"
        "currency": string,                 // The currency used by this player
        "currency-value": double,           // The approximate value of this currency in EUR (how many EUR is 1.00 of "currency")
        "history": Array<GameSnapshotObject>,  // list of previouslly played games for this player account
        "version": {                           // server version
            "major": 2,
            "minor": 0,
            "revision": 12
        }
    }
}
```

### Cookie

The server-initiated message type `cookie` is sent when the server requests a cookie to be set on the client's browser.
It's okay to not have this implemented, if you don't then games will always behave like they are run in an incognito tab, which isn't a big issue.
These are mostly quality-of-life features that handle corner-cases better (eg. player disconnecting and reconnecting).
A cookie request from the server looks like this:

```javascript
{
    "type": "cookie",
    "sentTS": 184561622,
    "contextID": 82,
    "payload": [
        "name": string,              // name of the cookie to set
        "value": string,             // cookie value
        "options": Array<string>     // array of HTTP cookie attributes (eg. Path=example.com)
    ]
}
```

### Question

A server-initiated `question` message type is sent when the server needs to ask for player input.

Some game types don't use this, so you can skip it if that's the case for your game type.

A question can either be a kind of yes/no question (a **choice**) or an **input** question (eg. enter password).
The client needs to be able to display the question and its options to the player and reply to it with the message type `response`.
A question message has a `payload` that looks like this:

```javascript
{
    "topic": string,        // question topic that determines what kind of choinces there are
    "type": string,         // type of the question, can be either "input" or "choice"
    "question": string,     // a string to display to the user (eg. Please Enter Password)
    "inputStyle": string    // only given if type=input - can be "text", "int" or "float"
}
```

The client needs to respond to such a question with the appropriate type as the `payload` of a message with type `response`
Eg. if the question had `contextID : 60`, the question type was `input`, and the `inputStyle` was `float`:

```javascript
{
    "type": "response",
    "sentTS": 18455923565,
    "contextID": 60,
    "payload": 50.3
}
```

If the question type is `choice`, the payload instead continues with

```javascript
{
    "minNumChoices": uint,
    "maxNumChoices": uint,
    "choices": Array<ChoiceObject>         // array of choice objects
}
```

where a `ChoiceObject` is

```javascript
{
    "title": string,           // title of this choice
    "description": string,     // a description of this choice
    "data": any                // type is determined by question "topic"
}
```

The response to a choice question is an array of integers which represent the indexes of individual choices that were selected.
The structure of `data` in choices is determined by the question `topic`.
Currently, choice questions are only used in specific games, so you don't need to worry about `data` - it will be tailored to the needs of the client when a game is integrated and needs this functionallity.

### Ping

A server sends a `ping` message every once in a while. The client should respond to this message immediately with a message of type `pong`.
The payload of the `pong` response the client sends must contain a member `received` which is set to the value of `sentTS` in the `ping` message.
Example server ping:

```javascript
{
    "type": "ping",
    "sentTS": 1845729893584,
    "contextID": 26,
    "payload": null
}
```

The client should respond to this immediately with:

```javascript
{
    "type": "pong",
    "sentTS": 1845729893672,
    "contextID": 26,
    "payload": {
        "received": 1845729893584       // same as sentTS of the server message
    }
}
```

## Client messages

Client-initiated message types `response` and `pong` were already covered above. So in this section we will be covering

### Requests

A list of requests and their response types is listed in the ***YServer WebSocket Requests*** documentation.

A client may send requests to the server by sending a message with type `request`, which should look like this:

```javascript
{
    "type": "request",
    "sentTS": 184572956562,
    "contextID": 41,
    "payload": {
        "request": string,       // name of the request to execute
        "data": any              // changes based on "request"
    }
}
```

A response to this example would look like:

```javascript
{
    "type": "response",
    "sentTS": 184572956721,
    "contextID": 41,
    "payload": {},         // changes based on the request that triggered this response, see lower list for response types
    "executionTime": 0     // tells the client how long the request took to process on the server in ms. If a request is not measured, it will return 0.
}
```

## Game Client Requirements

For a game client to be compliant it must implement some behaviors that the Y Server will take advantage of. If the game client wishes to be compliant with slot machines, it must implement the behaviors required for that.

---

### Basic Integration

For basic integration, the game client need only implement the following behaviors

#### Locking & unlocking the play area

When locked (event `lock`), the client should prevent the player from betting and display the lock message provided by the server.
When an `unlock` event is received, the client may resume normal play. If a `lock` event is received during an active game round, the client can expect that game round to be voided (any bets made will be returned to the player).

#### Displaying a message

When receiving a `message` event, the game client should display the message to the player for the specified time. See the ***YServer Events*** documentation for more info.

---

### Landbase Integration

To support hardware buttons, RGB illumination of the slot machine during play, and other landbase-related features, the game client should implement the following:

The game should send a `support-actions` request at any time after receiving the `init` message.
The payload of this action should consist of an array of objects which describe actions supported by the game.
You can find how that request should be structured in the ***YServer WebSocket Requests*** documentation.

During the course of the game, some of the game actions which were registered with `support-actions` may become un-invokable (disabled) for various reasons (eg. the spin action is disabled because a spin is currently in progress). If this happens, the game should signal the server that an action has had its enabled state changed. This is done with the `action-enabled` request. You can find how that request should be structured in the ***YServer WebSocket Requests*** documentation.

When an action is to be executed on the game client, the server will send an event named `action`, which is described in the ***YServer Events*** documentation.

---
