# HTTP Requests

HTTP requests on the YServer have 3 access levels:

- **Public:**
These requests are available for everyone and don't require any authorization

- **API Key**
These requests are only available when a valid API key is given in the ***Authorization*** HTTP request header.

- **Admin**
These requests are like API key request, only that the API key must be flagged as an admin key on the server side.

## Public

### Resource Upload

***PUT*** | **/res/[resourceID]**

This endpoint is used by games to upload resources to the server, which can be used by other APIs on the yserver (mainly Viewer in the case of slot machine top screens)

#### Responses

The following HTTP response codes can occur:

- **200 *(OK)*** | Success! The resource was saved.
- **204 *(No Content)*** | The resource being uploaded seems to be empty (zero Content-Length or empty HTTP body)
- **400 *(Bad Request)*** | resourceID was not given
- **404 *(Not Found)*** | A resource with the given ID was not found (a place for it has to be reserved by a Player API request)
- **500 *(Internal Error)*** | An error occured while saving the resource to the server

---

### Resource Download

***GET*** | **/res/[resourceID]**

This endpoint is used by APIs like Viewer, ehich download resources uploaded by games to display various information (eg. paytables, info screens, etc.)

#### Responses

The following HTTP response codes can occur:

- **200 *(OK)*** | Success! The resource is in the HTTp response body
- **204 *(No Content)*** | The resource has not yet loaded
- **400 *(Bad Request)*** | resourceID was not given
- **404 *(Not Found)*** | A resource with the given ID was not found
- **500 *(Internal Error)*** | An error occured while reading the resource

---

### Launch Game

***GET*** | **/player/[endpoint]/[signature]**

Launch a game client for the given endpoint

#### Query Parameters

The following query parameters are parsed:

- **secure** | bool *[optional - default true]* | True/false to indicate that the client should connect to the server via WSS instead of WS

#### Responses

The following HTTP response codes can occur:

- **302 *(Found)*** | Success - Game accessible at URL saved into the Location header
- **400 *(Bad Request)*** | Endpoint or signature was not given, or there was extra path fragments in the URL
- **401 *(Unauthorized)*** | Endpoint signature does not match
- **404 *(Not Found)*** | Endpoint not found or endpoint has an invalid game

---

### Launch New Game (Legacy)

***GET*** | **/game/[providerID]/[hostID]**

Launch a game given a provider and host

#### Query Parameters

The following query parameters are parsed:

- **credit-scale** | uint *[optional - default 1]* | Virtual denomination multiplier | >=1 AND <=100
- **demo** | bool *[optional - default false]* | True to start game as demo (no provider)
- **game** | string *[required]* | The name of the target game configuration on the given host
- **host** | uint *[required]* | The ID of the target host
- **provider** | uint *[optional - default 0]* | The ID of the target provider
- **sid** | string *[optional - default ""]* | The session to use on the provider

#### Responses

The following HTTP response codes can occur:

- **204 *(No Content)*** | Connection already taken by someone else
- **302 *(Found)*** | Success - Game accessible at URL saved into the Location header
- **400 *(Bad Request)*** | Provider or host ID not given or too many URL fragments
- **401 *(Unauthorized)*** | No permission to launch games or account/IP is banned
- **404 *(Not Found)*** | Host:game combo not found or provider not found or session timed out
- **500 *(Internal Error)*** | Unknown error occured

---

### Download Icons

***GET*** | **/icons/[hostID]/[iconIdx]**

Get the icon of a given host

#### Query Parameters

The following query parameters are parsed:

- **game** | string *[required]* | The name of the game to get the icon for

#### Responses

The following HTTP response codes can occur:

- **200 *(OK)*** | Success - Icon is sent as regular image file in the HTTP response
- **400 *(Bad Request)*** | Could not parse hostID, icon index, or wrong number of URL fragments
- **404 *(Not Found)*** | No such icon found (most likely iconIdx is too large for this host:game pair)
- **500 *(Internal Error)*** | Error reading image file

---

### Get Info

***GET*** | **/info**

Return server status and core information

#### Responses

The following HTTP response codes can occur:

- **200 *(OK)*** | The body of the HTTP response is a JSON value with the following schema
```javascript
{
   "admin" : , // <bool> True/false if the key used has admin rights
   "domain" : , // <string> Externally accessible address (public name)
   // <object> The version of the server
   "version" : 
   {
      "major" : , // <uint> Major version number
      "minor" : , // <uint> Minor version number
      "revision" :  // <uint> Revision build number
   }
} // <object> 
```


---

## API Key

### List Module Instances

***GET*** | **/list/[module]**

List all module instances (host or provider)

#### Query Parameters

The following query parameters are parsed:

- **download-config** | bool *[optional - default false]* | If given, also return module configurations
- **host** | uint *[optional - default 0]* | The ID of the host to use for filtering
- **host-domain** | string *[optional - default ""]* | The host domain to use for filtering (incomparrible with 'host')
- **provider** | uint *[optional - default 0]* | The ID of the provider to use for filtering
- **provider-domain** | string *[optional - default ""]* | The provider domain to use for filtering (incomparrible with 'provider')

#### Responses

The following HTTP response codes can occur:

- **200 *(OK)*** | The body of the HTTP response is a JSON value with the following schema
```javascript
[
   // <object> 
   {
      // <object> [optional] [ADMIN ONLY] list of actions which can be performed on this module
      "actions" : 
      {
         // <object> 
         "exampleMember1" : 
         {
            // <object> 
            "schema" : 
            {
               // <object> [optional] member tree of default values
               "default" : {},
               // <object> [optional] member tree of schemas
               "schema" : 
               {
                  // <object> 
                  "exampleMember1" : 
                  {
                     // <object> [optional] Another schema object defining how children of this container should be structured
                     "child-schema" : ,
                     "constraints" : 
                     [
                        [
                           // <object> 
                           {
                              "type" : , // <string> The type of this limit [The type of this limit]
                              "value" :  // <any> The limit value
                           }
                        ] // <array> [optional] List of OR conditions
                     ], // <array> [optional] List of AND conditions
                     "desc" : "", // <string> [optional] The description of this value
                     // <object> [optional] Member values which need to have conditions met for this value to be editable
                     "edit-conditions" : 
                     {
                        "exampleMember1" : 
                        [
                           // <object> 
                           {
                              "type" : , // <string> The type of this limit [The type of this limit]
                              "value" :  // <any> The limit value
                           }
                        ] // <array> [optional] List of conditions for this member
                     },
                     "interval" : 0.0, // <real> [optional] Rounding interval of the value (valid for numerical values only)
                     "max-children" : 0, // <uint> [optional] The maximum number of children
                     "min-children" : 0, // <uint> [optional] The minimum number of children
                     "optional" : false, // <bool> [optional] True/false if this value is optional
                     "scope" : "Default", // <string> [optional] True/false if this value is optional [True/false if this value is optional]
                     "type" :  // <string> The type of this value [The type of this value]
                  }
               }
            },
            "type" :  // <string> The type of action [The type of action]
         }
      },
      "config-error" : "", // <string> [optional] The configuration error which occured while loading this module
      "configured" : , // <bool> True/false if the module is configured (had no config errors)
      "domain" : "", // <string> [optional] The domain of this module
      "id" : , // <uint> The ID of this module
      "isFull" : , // <bool> True/false if the module has no more capacity for new players
      "maxPlayers" : , // <uint> The maximum allowed number of players on this module
      "name" : , // <string> The name of this module
      "owner" : "", // <string> [optional] The UID of the owner of this module (if any)
      "players" : , // <uint> The current number of players on this module
      // <object> The configuration of this module (only if download-config was requested)
      "raw-config" : {},
      "ready" : , // <bool> True/false if the module is ready for games to be played on it
      "status" : , // <string> The status of this module [The status of this module]
      "type" : , // <uint> The type of this module
      "type-name" : , // <string> The type name of this module
      "uid" :  // <string> [ADMIN ONLY] The UID of this module
   }
] // <array> List of modules
```

- **400 *(Bad Request)*** | Module type invalid or too many URL fragments
- **404 *(Not Found)*** | Filtered host or provider not found

---

## Admin

### Update YServer

***PUT*** | **/update**

Upload a new yserver binary and restart the server

#### Request Body

Must contain the binary file

#### Responses

The following HTTP response codes can occur:

- **200 *(OK)*** | Successfully uploaded new binary!
- **500 *(Internal Error)*** | An error occured while writing the file

---

### Get Module Schema

***GET*** | **/schema/[module]/[type]**

Get the schema of the specified module (host or provider) and type (eg. GameartSlotGame, Roulette, ...)

#### Responses

The following HTTP response codes can occur:

- **200 *(OK)*** | The body of the HTTP response is a JSON value with the following schema
```javascript
{
   // <object> [optional] member tree of default values
   "default" : {},
   // <object> [optional] member tree of schemas
   "schema" : 
   {
      // <object> 
      "exampleMember1" : 
      {
         // <object> [optional] Another schema object defining how children of this container should be structured
         "child-schema" : ,
         "constraints" : 
         [
            [
               // <object> 
               {
                  "type" : , // <string> The type of this limit [The type of this limit]
                  "value" :  // <any> The limit value
               }
            ] // <array> [optional] List of OR conditions
         ], // <array> [optional] List of AND conditions
         "desc" : "", // <string> [optional] The description of this value
         // <object> [optional] Member values which need to have conditions met for this value to be editable
         "edit-conditions" : 
         {
            "exampleMember1" : 
            [
               // <object> 
               {
                  "type" : , // <string> The type of this limit [The type of this limit]
                  "value" :  // <any> The limit value
               }
            ] // <array> [optional] List of conditions for this member
         },
         "interval" : 0.0, // <real> [optional] Rounding interval of the value (valid for numerical values only)
         "max-children" : 0, // <uint> [optional] The maximum number of children
         "min-children" : 0, // <uint> [optional] The minimum number of children
         "optional" : false, // <bool> [optional] True/false if this value is optional
         "scope" : "Default", // <string> [optional] True/false if this value is optional [True/false if this value is optional]
         "type" :  // <string> The type of this value [The type of this value]
      }
   }
} // <object> 
```

- **400 *(Bad Request)*** | module or type not given, or could not be parsed
- **404 *(Not Found)*** | Invalid module type

---

### Restart

***GET*** | **/restart**

Gracefully restart the server

#### Responses

The following HTTP response codes can occur:

- **200 *(OK)*** | The server will restart

---

### Game History

***GET*** | **/history**

Get a list of game rounds

#### Query Parameters

The following query parameters are parsed:

- **a** | string *[required]* | Accounting round ID (incompatible with 'a)
- **p** | string *[required]* | Player client UID
- **r** | string *[required]* | Game round ID (incompatible with 'a')
- **s** | string *[required]* | Signature of this request

#### Responses

The following HTTP response codes can occur:

- **200 *(OK)*** | The body of the HTTP response is a JSON value with the following schema
```javascript
{
   // <object> 
   "balance" : 
   {
      "after" : [], // <array> The balance after the round
      "before" : [] // <array> The balance before the round
   },
   "data" : , // <any> [optional] Extra data attributed to the round
   "endT" : , // <uint> The end timestamp
   "endpoint" : , // <string> Endpoint of this round
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
   ] // <array> Array of transactions in this round
} // <object> 
```

- **400 *(Bad Request)*** | Missing 'p', or one of 'r', 'a'
- **403 *(Forbidden)*** | Bad signature
- **404 *(Not Found)*** | No gameround found with given parameters

---

### Create Endpoint

***GET, POST*** | **/api/[type]**

Create an endpoint on the given API

#### Responses

The following HTTP response codes can occur:

- **400 *(Bad Request)*** | Bad structure of request URL or invalid body content
- **401 *(Unauthorized)*** | No permission to create endpoint or banned
- **404 *(Not Found)*** | Unknown API type
- **501 *(Not Implemented)*** | API is disabled
- **503 *(Service Unavailable)*** | The server is full

---
