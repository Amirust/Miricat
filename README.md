# Miricat Chat Protocol (MCP)

## About
MCP is a protocol for chat applications. This protocol uses AES-256-CTR encryption to secure the connection between the client and the server. The protocol is designed to be simple and easy to implement. The protocol is designed to be used with TCP sockets and HTTP for avatars, images and registration.

## Memo
- The protocol is designed to be used with TCP sockets and HTTP. HTTP is optional, but recommended.
- MCP uses password for login to TCP chat server. For access to HTTP server, MCP uses token what's updated every 10 minutes. Token sends to client after login to TCP server, read Events for dital information.
- MCP uses AES-256-CTR and Diffie-Hellman key exchange for encryption. Read Encryption for dital information.
- MCP uses JSON for transfer data between client and server. Read Messages for dital information.
- ALL data is encrypted, except public key exchange (Read connection). Read Encryption for dital information.

## Content
1. Protocol (TCP)
   1. Connection
   2. Authentication
   3. Messages
   4. Events
2. Protocol (HTTP)
   1. Registration
   2. Avatars
   3. Images
3. Authentication
   1. Token (HTTP)
4. Encryption
   1. Diffie-Hellman
   2. AES-256-CTR
   3. MCP Key
5. Client Requests
   1. TCP
6. Server Responses
   1. TCP
7. Features
   1. Disable Registration
## Protocol (TCP)
### Connection
Send JSON to server after TCP socket connection created. <br><br>
**CLIENT REQUEST**
```json
{
  "type": 0,
  "data": {
    "publicKey": "Diffie-Hellman public key in HEX encoding"
  }
}
```
**SERVER RESPONSE**
```json
{
  "type": 0,
  "data": {
    "publicKey": "Diffie-Hellman public key in HEX encoding"
  }
}
```
After this compute shared key and create MCP key for encryption (Read Encryption) <br>
### Authentication
If shared key is computed, send JSON to server with login and password. <br>
```json
{
  "type": 1,
  "data": {
    "login": "user login",
    "password": "user password, only if registration is enabled"
  }
}
```
If all is ok, server return this
```json
{
   "type": 1,
   "data": {
      "http": "HTTP Address:port or domain, if HTTP is enabled"
   }
}
```
If password is incorrect, server return this
```json
{
   "type": "2-3, type 2 if first data, type 3 if second data",
   "data": "User password required / User password incorrect"
}
```
If login is incorrect, server return this ONLY IF REGISTRATION IS ENABLED
```json
{
   "type": 4,
   "data": "User not registered"
}
```
If auth data is incorrect, server return this
```json
{
   "type": 2,
   "data": "Exception: AUTH: No username / Exception: AUTH: No public key"
}
```
If from you IP address connected more than 1 user, server return this
```json
{
   "type": 4,
   "data": "User with this nickname or ip address already connected"
}
``` 
### Messages
After authentication, you can send messages to server. <br>
```json
{
   "type": 2,
   "data": {
      "message": "Message text"
   }
}
```
If message is empty, server return this
```json
{
   "type": 10,
   "data": "Exception: USER_MESSAGE: No message"
}
```

### Events
After connection of new user, server broadcast this event to all users
```json
{
   "type": 7,
   "data": {
      "user": "username",
      "allUsers": ["user1", "user2", "user3"]
   }
}
```
After disconnect, server broadcast this event to all users
```json
{
   "type": 8,
   "data": {
      "user": "username"
   }
}
```
If user sends message, server broadcast this event to all users
```json
{
   "type": 6,
   "data": {
      "user": "username",
      "message": "message text"
   }
}
```
Every 10 minutes server send the message for you client with new access token
```json
{
   "type": 11,
   "data": {
      "accessToken": "token"
   }
}
```

## Protocol (HTTP)
### Registration
If registration is enabled, you can register new user with HTTP request. <br>
```http request
POST /register HTTP/1.1
Content-Type: application/json

{
   "login": "user login",
   "password": "user password"
}

### Expect code: 200 and body: "User created"
### If user with this login already exists, expect code: 400 and body: "User already exists"
```
### Avatars
```http request
GET /user/:username/avatar HTTP/1.1
Authorization: MCPToken

### Expect code: 200 and file response
### If user not found, expect code: 404 and body: "User not found"
### If user not have avatar, expect code: 404 and body: "User not have avatar"

POST /user/:username/avatar HTTP/1.1
Authorization: MCPToken
Content-Type: multipart/form-data

avatar: file

### Expect code: 200 and body: "Avatar updated"
### If no file, expect code: 400 and body: "Invalid file"
### If user access token is invalid, expect code: 403 and body: "Forbidden"
### If user not found, expect code: 404 and body: "User not found"
### If file type is invalid, expect code: 415 and body: "Invalid file type"
```
### Images
```http request
GET /image/:image HTTP/1.1
Authorization: MCPToken

### Expect code: 200 and file response
### If image not found, expect code: 404 and body: "Image not found"
### If authorization token is invalid, expect code: 401 and body: "Unauthorized"

POST /image HTTP/1.1
Authorization: MCPToken
Content-Type: multipart/form-data

image: file

### Expect code: 200 and body: { "file": "filename.ext" }
### If authorization token is invalid, expect code: 401 and body: "Unauthorized"
```

## Authentication
### Token (HTTP)
MCPToken is token for access to HTTP server. <br>
Token is generated every 10 minutes, and when server accepted you connection to TCP <br>
Token is generated with this algorithm: <br>
```dart
String generateAccessToken() {
 List<int> bytes = List.generate(8, (index) => Random().nextInt(256));
 accessToken = base64.encode(bytes) + base64.encode(sha1.convert(utf8.encode(username)).bytes);
 return accessToken;
}
```
Token is required in all HTTP requests, except registration.

## Encryption
### Diffie-Hellman
Diffie-Hellman is used for key exchange. <br>
MCP Uses modp5 group. <br>

### AES-256-CTR
AES-256-CTR is used for encryption. <br>
MCP uses 16 bytes IV filled by zero-bytes for all messages. <br>
MCP uses 32 bytes key for encryption. This name is MCP Key <br>

### MCP Key
MCP Key is shared key between client and server. But sliced to 32 characters. <br>
When you compute shared key, you need to slice it to 32 characters and use for encryption. <br>
Node.js example:
```js
const crypto = require('crypto');
const diffieHellman = crypto.createDiffieHellmanGroup('modp5');
diffieHellman.generateKeys();

const publicKey = diffieHellman.getPublicKey('hex');
const serverPublicKey = 'server public key';
const sharedKey = diffieHellman.computeSecret(serverPublicKey, 'hex');
const mcpKey = sharedKey.slice(0, 32);
```

## Client requests
TCP
```
Handshake:   0, {"publicKey": "public key"}
Auth:        1, {"login": "user login", "password": "user password"}
UserMessage: 2, {"message": "message text"}
```

## Server responses
TCP
```
Handshake:               0, {"publicKey": "public key"}
Accepted:                1, {"http": "HTTP Address:port or domain, if HTTP is enabled"}
UserPasswordRequired:    2, "User password required"
UserPasswordIncorrect:   3, "User password incorrect"
AlreadyConnected:        4, "User with this nickname or ip address already connected"
UserNotFound:            5, "User not found"
UserMessage:             6, {"user": "username", "message": "message text"}
UserAdd:                 7, {"user": "username", "allUsers": ["user1", "user2", "user3"]}
UserRemove:              8, {"user": "username"}
AuthDataInvalid:         9, "Exception: AUTH: No username / Exception: AUTH: No public key"
MessageDataInvalid:      10, "Exception: USER_MESSAGE: No message"
UpdateAccessToken:       11, {"accessToken": "access token"}
UserNotRegistered:       12, "User not registered"
```

## Future plans
- [ ] Add rooms