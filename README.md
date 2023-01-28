# Miricat Chat Protocol (MCP)
### Version 2.0.0

## About
MCP is a protocol for chat applications. This protocol uses AES-256-CTR encryption to secure the connection between the client and the server. The protocol is designed to be simple and easy to implement. The protocol is designed to be used with TCP sockets and HTTP for avatars, images and registration.

## Memo
- The protocol is designed to be used with TCP sockets and HTTP. HTTP is optional, but recommended.
- MCP uses password for login to TCP chat server. For access to HTTP server, MCP uses token what's updated every 10 minutes. Token sends to client after login to TCP server, read Events for dital information.
- MCP uses AES-256-CTR and Diffie-Hellman key exchange for encryption. Read Encryption for dital information.
- MCP uses JSON for transfer data between client and server. Read Packages for dital information.
- ALL data is encrypted, except public key exchange (Read connection). Read Encryption for dital information.
- MCP uses rooms for chat. Read Rooms for dital information.
- All MCP packages in TCP is strictly standardized. Read Packages for dital information.
- If user not joined to room, server doesn't send room messages to socket

## Content
1. Packages
   1. Types
   2. Objects
2. Protocol (TCP)
   1. Handshake
   2. Authentication
   3. Messages
   4. Rooms
3. Protocol (HTTP)
   1. Registration
   2. Avatars
   3. Images
4. Authentication
   1. Token (HTTP)
5. Encryption
   1. Diffie-Hellman
   2. AES-256-CTR
   3. MCP Key
6. Versions

## Packages
Package is a JSON object with type and data fields. This is strictly standardized.
```json5
{
   // Example: Handshake.Handshake or Authentication.Login, read Types for dital information
   "type": "MajorType.MinorType",
   "data": {
      // Package object data, read Objects, Client Requests or Server Responses for dital information
   }
}
```
For Message types you need to specify the room. This is strictly standardized.
```json5
{
    // Example: Message.CreateTextMessage
   "type": "MajorType.MinorType",
   "data": {
      // Package object data, read Objects, Client Requests or Server Responses for dital information
   },
   "room": "Room id"
}
```
### Types
|   MajorType    |      MinorType       | Description                       |
|:--------------:|:--------------------:|-----------------------------------|
|   Handshake    |      Handshake       | Handshake                         |
| Authentication |        Login         | Login                             |
| Authentication |       Accepted       | Server accepted you connection    |
| Authentication |  UpdateAccessToken   | Update MCP Access Token           |
|    Message     |  CreateTextMessage   | Create text message               |
|    Message     |     TextMessage      | Text message                      |
|    Message     |  CreateImageMessage  | Create image message              |
|    Message     |     ImageMessage     | Image message                     |
|      Room      |        Create        | Create room ONLY FOR ROOT         |
|      Room      |       Created        | Room created                      |
|      Room      |         Join         | Join room                         |
|      Room      |        Joined        | User joined to room               |
|      Room      |        Leave         | Leave room                        |
|      Room      |         Left         | User left room                    |
|      Room      |        Delete        | Delete room ONLY FOR ROOT         |
|      Room      |       Deleted        | Room deleted                      |
|      Room      |        Update        | Update room ONLY FOR ROOT         |
|      Room      |       Updated        | Room updated                      |
|      Room      |     RequireList      | Require the room list from server |
|      Room      |         List         | List rooms                        |
|     Error      | UserPasswordRequired | You need to specify user password |
|     Error      | UserPasswordInvalid  | User password is invalid          |
|     Error      |     UserNotFound     | User not found                    |
|     Error      |  UserAlreadyExists   | User already exists               |
|     Error      | PacketDataIncorrect  | Packet data is invalid            |
|     Error      |  AuthDataIncorrect   | Auth data is invalid              |
|     Error      | MessageDataIncorrect | Message data is invalid           |
|     Error      |  UserDontConnected   | If user dont connected to room    |
|     Error      |     RoomNotFound     | Room not found                    |
|     Error      |  RoomAlreadyExists   | Room already exists               |
|     Error      |  RoomDataIncorrect   | Room data is invalid              |
|     Error      |    RoomDontExists    | Room dont exists                  |
|     Error      |     AccessDenied     | Access Denied                     |

### Objects
Here described objects for Major.Minor types. (if minor type is not specified, then object is for all minor types)
#### Client
##### Handshake
```json5
{
   "type": "Handshake.Handshake",
   "data": {
      "version": "MCP version", // Read Versions
      "publicKey": "Public key for Diffie-Hellman key exchange"
   }
}
```

##### Authentication.Login
```json5
{
   "type": "Authentication.Login",
   "data": {
      "username": "Username",
      "password": "Password"
   }
}
```
##### Message.CreateTextMessage
```json5
{
   "type": "Message.CreateTextMessage",
   "data": {
      "text": "Text message"
   },
    "room": "Room id"
}
```
##### Message.CreateImageMessage
```json5
{
   "type": "Message.CreateImageMessage",
   "data": {
      "image": "Image URL, read HTTP Images for dital information"
   },
   "room": "Room id"
}
```
##### Room.Create
```json5
{
   "type": "Room.Create",
   "data": {
      "name": "Room name"
   }
}
```
##### Room.Join
```json5
{
   "type": "Room.Join",
   "data": {
      "room": "Room id"
   }
}
```
##### Room.Leave
```json5
{
   "type": "Room.Leave",
   "data": {
      "room": "Room id"
   }
}
```
##### Room.Delete
```json5
{
   "type": "Room.Delete",
   "data": {
      "room": "Room id"
   }
}
```
##### Room.Update
```json5
{
   "type": "Room.Update",
   "data": {
      "room": "Room id",
      "name": "Room name"
   }
}
```
##### Room.RequireList
```json5
{
   "type": "Room.RequireList"
}
```
#### Server
##### Handshake
```json5
{
   "type": "Handshake.Handshake",
   "data": {
      "version": "MCP version", // Read Versions
      "publicKey": "Public key for Diffie-Hellman key exchange"
   }
}
```
#### Authentication.Accepted
```json5
{
   "type": "Authentication.Accepted",
   "data": {
      "rooms": [{
         "id": "Room id",
         "name": "Room name",
         "allUsers": ["Username", "Username", "Username"]
      }]
   }
}
```
#### Authentication.UpdateAccessToken
```json5
{
   "type": "Authentication.UpdateAccessToken",
   "data": {
      "accessToken": "MCP Access Token"
   }
}
```
##### Message.TextMessage
```json5
{
   "type": "Message.TextMessage",
   "data": {
      "text": "Text message",
      "user": "Username",
      "time": "Message timestamp"
   },
   "room": "Room id"
}
```
##### Message.ImageMessage
```json5
{
   "type": "Message.ImageMessage",
   "data": {
      "image": "url",
      "user": "Username",
      "time": "Message timestamp"
   }
}
```
##### Room.Created
```json5
{
   "type": "Room.Created",
   "data": {
      "room": "Room id",
      "name": "Room name"
   }
}
```
##### Room.Joined
```json5
{
   "type": "Room.Joined",
   "data": {
      "room": "Room id",
      "user": "Username",
      "allUsers": ["Username", "Username", "Username"]
   }
}
```
##### Room.Left
```json5
{
   "type": "Room.Left",
  "data": {
     "room": "Room id",
     "user": "Username"
  }
}
```
##### Room.Deleted
```json5
{
   "type": "Room.Deleted",
   "data": {
      "room": "Room id"
   }
}
```
##### Room.Updated
```json5
{
   "type": "Room.Updated",
   "data": {
      "room": "Room id",
      "name": "Room name"
   }
}
```
##### Room.List
```json5
{
   "type": "Room.List",
   "data": [
      {
         "room": "Room id",
         "name": "Room name"
      }
   ]
}
```
##### Error
```json5
{
   "type": "Error.MinorType",
   "data": {
      "message": "Error message"
   }
}
```

## Protocol (TCP)
### Handshake
1. Client sends Handshake.Handshake with Handshake object
2. Server sends Handshake.Handshake with Handshake object
3. After this compute the shared key, after this encrypt all messages (Read Encryption)

### Authentication
After handshake
1. Client sends Authentication.Login with Login object
2. Server sends Authentication.Accepted with Accepted object
3. After this client can send any message

If user data is invalid:
1. Server sends Error.AuthDataInvalid with Error object

### Messages
After authentication
1. Client sends Message.CreateTextMessage with CreateTextMessage object
2. Server sends Message.TextMessage with TextMessage object

If you want to send image:
1. Upload image to HTTP server and get URL (or upload image to any host and get direct URL)
2. Client sends Message.CreateImageMessage with CreateImageMessage object
3. Server sends Message.ImageMessage with ImageMessage object

If your client is CLI, you can not implement image sending/receiving, because how :trl:
If message is invalid:
1. Server sends Error.MessageInvalid with Error object

If room don't exist:
1. Server sends Error.RoomDontExists with Error object

### Rooms
After authentication
If you want to connect to room:
1. Client sends Room.Join with Join object
2. Server sends Room.Joined with Joined object
3. After this client can send any message

If you want to leave room:
1. Client sends Room.Leave with Leave object
2. Server sends Room.Left with Left object

If user don't connected to room, but want to send message or leave room:
1. Server sends Error.UserDontConnected with Error object

If you want to create room:
1. Client sends Room.Create with Create object
2. Server sends Room.Created with Created object

If you are not a root user, but want to create room:
1. Server sends Error.AccessDenied with Error object

If room name is claimed:
1. Server sends Error.RoomAlreadyExists with Error object

If you want to delete room:
1. Client sends Room.Delete with Delete object
2. Server sends Room.Deleted with Deleted object

If you are not a root user, but want to delete room:
1. Server sends Error.AccessDenied with Error object

If room don't exist:
1. Server sends Error.RoomDontExists with Error object

If you want to update room:
1. Client sends Room.Update with Update object
2. Server sends Room.Updated with Updated object

If you are not a root user, but want to update room:
1. Server sends Error.AccessDenied with Error object

If room don't exist:
1. Server sends Error.RoomDontExists with Error object

If you want change room name to name what's claimed:
1. Server sends Error.RoomAlreadyExists with Error object

If you want to get room list:
1. Client sends Room.RequireList
2. Server sends Room.List with Room.List object

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
Token is generated with this algorithm (Dart example): <br>
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
When you compute shared key,
1. Convert it to base64
2. Slice it to 32 characters and use for encryption. <br>
Node.js example:
```js
const crypto = require('crypto');
const diffieHellman = crypto.createDiffieHellmanGroup('modp5');
diffieHellman.generateKeys();

const publicKey = diffieHellman.getPublicKey('hex');
const serverPublicKey = 'server public key';
const sharedKey = diffieHellman.computeSecret(serverPublicKey, 'hex', 'base64');
const mcpKey = sharedKey.slice(0, 32);
```

## Versions
### 1.0.0
The first version of MCP. <br>
Very simple, without rooms, pls don't use it.

### 2.0.0
Current version of MCP. <br>
Added rooms, but without private rooms. <br>


## Plans
- [ ] Add private rooms
