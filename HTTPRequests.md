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
- **500 *(Internal Server Error)*** | An error occured while saving the resource to the server

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
- **500 *(Internal Server Error)*** | An error occured while reading the resource

---

### Launch Game

***GET*** | **/player/[endpoint]**

Launch a game client for the given endpoint

#### Query Parameters

The following query parameters are parsed:

- **replay** | bool *[optional - default false]* | True/false to launch the game client in replay mode
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

- **credit-scale** | uint *[optional - default 1]* | Virtual denomination multiplier | must satisfy all: [any option: =1 ,>1] ,[any option: =100 ,<100] | values must be in intervals of 1.000000
- **demo** | bool *[optional - default false]* | True to start game as demo (no provider)
- **game** | string *[required]* | The name of the target game configuration on the given host
- **host** | uint *[required]* | The ID of the target host |  | values must be in intervals of 1.000000
- **provider** | uint *[optional - default 0]* | The ID of the target provider |  | values must be in intervals of 1.000000
- **sid** | string *[optional - default ""]* | The session to use on the provider

#### Responses

The following HTTP response codes can occur:

- **204 *(No Content)*** | Connection already taken by someone else
- **302 *(Found)*** | Success - Game accessible at URL saved into the Location header
- **400 *(Bad Request)*** | Provider or host ID not given or too many URL fragments
- **401 *(Unauthorized)*** | No permission to launch games or account/IP is banned
- **404 *(Not Found)*** | Host:game combo not found or provider not found or session timed out
- **500 *(Internal Server Error)*** | Unknown error occured

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
- **500 *(Internal Server Error)*** | Error reading image file

---

### Get Info

***GET*** | **/info**

Return server status and core information

#### Responses

The following HTTP response codes can occur:

- **200 *(OK)*** | The body of the HTTP response is a JSON value with the following schema
```javascript
 // <object> 
```


---

### Croupier Login

***GET*** | **/croupier/auth**

Try to authenticate a croupier badge

#### Query Parameters

The following query parameters are parsed:

- **id** | string *[required]* | The badge ID to authenticate with

#### Responses

The following HTTP response codes can occur:

- **200 *(OK)*** | The body of the HTTP response is a JSON value with the following schema
```javascript
 // <object> Information about the croupier
```

- **400 *(Bad Request)*** | Missing 'id'
- **401 *(Unauthorized)*** | No croupier matches the given badge

---

### Display error page

***GET*** | **/error**

Load an embedded version of the game on the given endpoint

#### Query Parameters

The following query parameters are parsed:

- **code** | int *[optional - default 0]* | The error code |  | values must be in intervals of 1.000000
- **locale** | string *[optional - default ""]* | The locale to use when displaying the error
- **msg** | string *[optional - default ""]* | The error message

#### Responses

The following HTTP response codes can occur:

- **200 *(OK)*** | The body of the HTTP response is a JSON value with the following schema
```javascript
 // <object> 
```


---

## API Key

### Query specific host Instances

***GET*** | **/query/[module]/*/***

Query special methods on specific module instances (host or provider)

#### Responses

The following HTTP response codes can occur:

- **200 *(OK)*** | Response for the specified resource
- **400 *(Bad Request)*** | Module type invalid or too many URL fragments
- **404 *(Not Found)*** | Host or provider not found

---

### List Module Instances

***GET*** | **/list/[module]**

List all module instances (host or provider)

#### Query Parameters

The following query parameters are parsed:

- **bet-unit** | real *[optional - default 0.0]* | The minimal bet unit in units of currency of the selected provider. Used to filter-out games with a higher min-bet.
- **download-config** | bool *[optional - default false]* | If given, also return module configurations
- **host** | uint *[optional - default 0]* | The ID of the host to use for filtering |  | values must be in intervals of 1.000000
- **host-domain** | string *[optional - default ""]* | The host domain to use for filtering (incompatible with 'host')
- **locale** | string *[optional - default "en_US"]* | The desired locale to translate texts where applicable
- **max-bet-rules** | string *[optional - default ""]* | List of max bet rules to apply to the listing (do not list hosts which are unplayable due to the rule). Requires bet-unit. Format is [HostType1]=[max bet in currency];[HostType2]=[max bet];...
- **provider** | uint *[optional - default 0]* | The ID of the provider to use for filtering |  | values must be in intervals of 1.000000
- **provider-domain** | string *[optional - default ""]* | The provider domain to use for filtering (incompatible with 'provider')
- **skip-host-games** | bool *[optional - default false]* | If true, will not list games of returned hosts

#### Responses

The following HTTP response codes can occur:

- **200 *(OK)*** | The body of the HTTP response is a JSON value with the following schema
```javascript
 // <array> List of modules
```

- **400 *(Bad Request)*** | Module type invalid or too many URL fragments
- **404 *(Not Found)*** | Filtered host or provider not found

---

### Replay

***GET*** | **/replay**

Replay a specific game round

#### Query Parameters

The following query parameters are parsed:

- **player** | string *[required]* | Player username
- **provider** | string *[required]* | Casino provider ID or UID
- **round** | string *[required]* | Gameround ID

#### Responses

The following HTTP response codes can occur:

- **302 *(Found)*** | The launch url is given in the Location header
- **400 *(Bad Request)*** | Missing 'provider', 'player' or 'round'
- **404 *(Not Found)*** | No gameround found with given parameters

---

### Create Endpoint

***GET, POST*** | **/api/Player**

Create a Player endpoint

#### Query Parameters

The following query parameters are parsed:

- **autorebet-override** | bool *[optional]* | The virtual credit multiplier
- **credit-scale** | uint *[optional - default 1]* | The virtual credit multiplier |  | values must be in intervals of 1.000000
- **demo** | bool *[optional - default false]* | True to launch a demo version of the game
- **dev** | bool *[optional - default false]* | If true, use dev launch url
- **drop-on-disconnect** | bool *[optional - default false]* | If true, drop player when he disconnects
- **game** | string *[required]* | Game configuration to launch
- **host** | null *[required]* | Host ID or UID
- **max-bet** | real *[optional - default 0.0]* | The max bet in currency units
- **max-win** | real *[optional - default 0.0]* | The max possible win in currency units
- **provider** | uint *[optional - default 0]* | Provider ID or UID |  | values must be in intervals of 1.000000
- **sid** | string *[optional - default ""]* | The session

#### Responses

The following HTTP response codes can occur:

- **400 *(Bad Request)*** | Bad structure of request URL or invalid body content
- **401 *(Unauthorized)*** | No permission to create endpoint or banned
- **404 *(Not Found)*** | Unknown API type
- **501 *(Not Implemented)*** | API is disabled
- **503 *(Service Unavailable)*** | The server is full

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
- **500 *(Internal Server Error)*** | An error occured while writing the file

---

### Get Module Schema

***GET*** | **/schema/[module]/[type]**

Get the schema of the specified module (host or provider) and type (eg. GameartSlotGame, Roulette, ...)

#### Responses

The following HTTP response codes can occur:

- **200 *(OK)*** | The body of the HTTP response is a JSON value with the following schema
```javascript
 // <object> 
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
 // <object> 
```

- **400 *(Bad Request)*** | Missing 'p', or one of 'r', 'a'
- **403 *(Forbidden)*** | Bad signature
- **404 *(Not Found)*** | No gameround found with given parameters

---

### Create Endpoint

***GET, POST*** | **/api/Logger**

Create a Logger endpoint

#### Query Parameters

The following query parameters are parsed:


#### Responses

The following HTTP response codes can occur:

- **400 *(Bad Request)*** | Bad structure of request URL or invalid body content
- **401 *(Unauthorized)*** | No permission to create endpoint or banned
- **404 *(Not Found)*** | Unknown API type
- **501 *(Not Implemented)*** | API is disabled
- **503 *(Service Unavailable)*** | The server is full

---

### Create Endpoint

***GET, POST*** | **/api/Commander**

Create a Commander endpoint

#### Query Parameters

The following query parameters are parsed:


#### Responses

The following HTTP response codes can occur:

- **400 *(Bad Request)*** | Bad structure of request URL or invalid body content
- **401 *(Unauthorized)*** | No permission to create endpoint or banned
- **404 *(Not Found)*** | Unknown API type
- **501 *(Not Implemented)*** | API is disabled
- **503 *(Service Unavailable)*** | The server is full

---

### Create Endpoint

***GET, POST*** | **/api/Viewer**

Create a Viewer endpoint

#### Query Parameters

The following query parameters are parsed:


#### Responses

The following HTTP response codes can occur:

- **400 *(Bad Request)*** | Bad structure of request URL or invalid body content
- **401 *(Unauthorized)*** | No permission to create endpoint or banned
- **404 *(Not Found)*** | Unknown API type
- **501 *(Not Implemented)*** | API is disabled
- **503 *(Service Unavailable)*** | The server is full

---

### Create Endpoint

***GET, POST*** | **/api/StaticViewer**

Create a StaticViewer endpoint

#### Query Parameters

The following query parameters are parsed:

- **ar** | string *[optional - default ""]* | Accounting round ID
- **dev** | bool *[optional - default false]* | If true, use dev launch url
- **endp** | string *[optional - default ""]* | Endpoint
- **player** | string *[required]* | Player username
- **provider** | null *[required]* | Provider ID or UID
- **round** | string *[optional - default ""]* | Gameround ID

#### Responses

The following HTTP response codes can occur:

- **400 *(Bad Request)*** | Bad structure of request URL or invalid body content
- **401 *(Unauthorized)*** | No permission to create endpoint or banned
- **404 *(Not Found)*** | Unknown API type
- **501 *(Not Implemented)*** | API is disabled
- **503 *(Service Unavailable)*** | The server is full

---

### Add Croupier

***GET*** | **/croupier/add**

Register a new croupier and return the badge ID

#### Query Parameters

The following query parameters are parsed:

- **birthdate** | string *[required]* | The date of birth of the croupier (in the format day/month/year)
- **location** | string *[required]* | The location the croupier is from
- **name** | string *[required]* | The full name of the croupier to add

#### Responses

The following HTTP response codes can occur:

- **200 *(OK)*** | The body of the HTTP response is a JSON value with the following schema
```javascript
 // <string> The badge ID which was created
```

- **400 *(Bad Request)*** | Missing 'name', 'location' or 'birthdate'
- **409 *(Conflict)*** | Croupier already exists

---
