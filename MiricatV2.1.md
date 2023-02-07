# Miricat Chat Protocol (MCP)
### Version 2.1.0

## About
MCP is a protocol for chat applications. This protocol uses AES-256-CTR encryption to secure the connection between the client and the server. The protocol is designed to be simple and easy to implement. The protocol is designed to be used with TCP sockets.

## Memo
- The protocol is designed to be used with TCP sockets.
- MCP uses password for login to TCP chat server.
- MCP uses AES-256-CTR and Diffie-Hellman key exchange for encryption. Read Encryption for dital information.
- MCP uses JSON for transfer data between client and server. Read Packages for dital information.
- ALL data is encrypted, except public key exchange (Read connection). Read Encryption for dital information.
- MCP uses rooms for chat. Read Rooms for dital information.
- All MCP packages in TCP is strictly standardized. Read Packages for dital information.
- If user not joined to room, server doesn't send room messages to socket
- MCP Uses Heartbeat for keep connection alive. Read Heartbeat for dital information.
- MCP Uses packet ids for reply to client. Read Packages for dital information.

## Content
1. Packages
    1. Types 
    2. Ids
    3. Objects
2. Protocol (TCP)
    1. Handshake
    2. Authentication
    3. Messages
    4. Rooms
    5. Heartbeat
3. Encryption
    1. Diffie-Hellman
    2. AES-256-CTR
    3. MCP Key
4. Versions

## Packages
Package is a JSON object with type and data fields. This is strictly standardized.
```json5
{
   // Example: Handshake.Handshake.ID or Authentication.Login.ID, read Types for dital information
   "type": "MajorType.MinorType.ID",
   "data": {
      // Package object data, read Objects, Client Requests or Server Responses for dital information
   }
}
```
For Message types you need to specify the room. This is strictly standardized.
```json5
{
    // Example: Message.CreateTextMessage.ID
   "type": "MajorType.MinorType.ID",
   "data": {
      // Package object data, read Objects, Client Requests or Server Responses for dital information
   },
   "room": 1, // Room id
}
```
### Types
|   MajorType    |       MinorType        | Description                       |
|:--------------:|:----------------------:|-----------------------------------|
|   Handshake    |       Handshake        | Handshake                         |
|   Heartbeat    |          Ping          | Ping                              |
| Authentication |         Login          | Login                             |
| Authentication |        Register        | Registration                      |
| Authentication |        Accepted        | Server accepted you connection    |
|    Message     |   CreateTextMessage    | Create text message               |
|    Message     |      TextMessage       | Text message                      |
|    Message     |   CreateImageMessage   | Create image message              |
|    Message     |      ImageMessage      | Image message                     |
|      File      |   RequestImageUpload   | Request the image uploading       |
|      File      |  ImageUploadAccepted   | Server accepted upload            |
|      File      |  ImageUploadRejected   | Server rejected upload            |
|      File      |    ImageUploadPart     | Image part                        |
|      File      |  ImageUploadCompleted  | Image upload completed            |
|      File      |  RequestImageDownload  | Request the image downloading     |
|      File      |   ImageDownloadMeta    | Meta of image                     |
|      File      |   ImageDownloadPart    | Image part                        |
|      File      | ImageDownloadCompleted | Image download completed          |
|      User      |         Create         | Create user ONLY FOR ROOT         |
|      User      |        Created         | User created                      |
|      User      |         Delete         | Delete user ONLY FOR ROOT         |
|      User      |        Deleted         | User deleted                      |
|      Room      |         Create         | Create room ONLY FOR ROOT         |
|      Room      |        Created         | Room created                      |
|      Room      |          Join          | Join room                         |
|      Room      |         Joined         | User joined to room               |
|      Room      |         Leave          | Leave room                        |
|      Room      |          Left          | User left room                    |
|      Room      |         Delete         | Delete room ONLY FOR ROOT         |
|      Room      |        Deleted         | Room deleted                      |
|      Room      |         Update         | Update room ONLY FOR ROOT         |
|      Room      |        Updated         | Room updated                      |
|      Room      |      RequireList       | Require the room list from server |
|      Room      |          List          | List rooms                        |
|     Error      |  UserPasswordRequired  | You need to specify user password |
|     Error      |  UserPasswordInvalid   | User password is invalid          |
|     Error      |      UserNotFound      | User not found                    |
|     Error      |   UnsupportedVersion   | Version not supported             |
|     Error      |   UserAlreadyExists    | User already exists               |
|     Error      |  PacketDataIncorrect   | Packet data is invalid            |
|     Error      |   AuthDataIncorrect    | Auth data is invalid              |
|     Error      |  MessageDataIncorrect  | Message data is invalid           |
|     Error      |   UserDontConnected    | If user dont connected to room    |
|     Error      |      RoomNotFound      | Room not found                    |
|     Error      |   RoomAlreadyExists    | Room already exists               |
|     Error      |   RoomDataIncorrect    | Room data is invalid              |
|     Error      |     RoomDontExists     | Room dont exists                  |
|     Error      |      AccessDenied      | Access Denied                     |
|     Error      |    HeartbeatTimeout    | HeartbeatTimeout                  |

### IDs
Client generate the id for packet, this is needed for async reply to client. 
The ID client generates self, it may be any number, string or etc., but it must be unique for each packet.
ID is specified in the type field.
```json5
{
   "type": "MajorType.MinorType.ID",
   "data": {
      // Package object data, read Objects, Client Requests or Server Responses for dital information
   }
}
```
Server will respond with the same id.
```json5
{
   "type": "MajorType.MinorType.ID",
   "data": {
      // Package object data, read Objects, Client Requests or Server Responses for dital information
   }
}
```

**IMPORTANT**
If server cannot parse the packet, ID **MUST** be DontHaveID.
Types which don't have ID:
- Error.HeartbeatTimeout

### Objects
Here described objects for Major.Minor types. (if minor type is not specified, then object is for all minor types)
#### Client
##### Handshake
```json5
{
   "type": "Handshake.Handshake.ID",
   "data": {
      "version": "MCP version", // Read Versions
      "publicKey": "Public key for Diffie-Hellman key exchange"
   }
}
```

##### Authentication.Login
```json5
{
   "type": "Authentication.Login.ID",
   "data": {
      "username": "Username",
      "password": "Password"
   }
}
```

##### Authentication.Register
```json5
{
   "type": "Authentication.Register.ID",
   "data": {
      "username": "Username",
      "password": "Password"
   }
}
```

##### Message.CreateTextMessage
```json5
{
   "type": "Message.CreateTextMessage.ID",
   "data": {
      "text": "Text message"
   },
    "room": 1, // Room id
}
```
##### Message.CreateImageMessage
```json5
// If you want to send image by url
{
   "type": "Message.CreateImageMessage.ID",
   "data": {
      "url": "Image url",
   },
   "room": 1, // Room id
}
```

```json5
// If you want to send image which saved on server
{
   "type": "Message.CreateImageMessage.ID",
   "data": {
      "image": "mcp_image://image_id",
   },
   "room": 1, // Room id
}
```

##### File.RequestImageUpload
```json5
{
   "type": "File.RequestImageUpload.ID",
   "data": {
      "name": "Image name",
      "size": 123456789, // Image size in bytes
      "type": "Image type", // Mime type
   }
}
```
##### File.ImageUploadPart
```json5
{
   "type": "File.ImageUploadPart.ID",
   "data": {
      "id": "Image id",
      "part": 1, // Part number
      "data": "Base64 encoded image part"
   }
}
```
##### File.ImageUploadCompleted
```json5
{
   "type": "File.ImageUploadCompleted.ID",
   "data": {
      "id": "Image id",
   }
}
```
##### File.RequestImageDownload
```json5
{
   "type": "File.RequestImageDownload.ID",
   "data": {
      "id": "Image id",
   }
}
```

#### User.Create
```json5
{
   "type": "User.Create.ID",
   "data": {
      "username": "Username",
      "password": "Password"
   }
}
```

#### User.Delete
```json5
{
   "type": "User.Delete.ID",
   "data": {
      "username": "Username"
   }
}
```

##### Room.Create
```json5
{
   "type": "Room.Create.ID",
   "data": {
      "name": "Room name"
   }
}
```
##### Room.Join
```json5
{
   "type": "Room.Join.ID",
   "data": {
      "room": 1, // Room id
   }
}
```
##### Room.Leave
```json5
{
   "type": "Room.Leave.ID",
   "data": {
      "room": 1, // Room id
   }
}
```
##### Room.Delete
```json5
{
   "type": "Room.Delete.ID",
   "data": {
      "room": 1, // Room id
   }
}
```
##### Room.Update
```json5
{
   "type": "Room.Update.ID",
   "data": {
      "room": 1, // Room id
      "name": "Room name"
   }
}
```
##### Room.RequireList
```json5
{
   "type": "Room.RequireList.ID"
}
```
#### Server
##### Handshake
```json5
{
   "type": "Handshake.Handshake.ID",
   "data": {
      "version": "MCP version", // Read Versions
      "publicKey": "Public key for Diffie-Hellman key exchange"
   }
}
```
#### Authentication.Accepted
```json5
{
   "type": "Authentication.Accepted.ID",
   "data": {
      "rooms": [{
         "id": 1, // Room id
         "name": "Room name",
         "allUsers": ["Username", "Username", "Username"]
      }],
      "heartbeatRate": 30000 // Heartbeat rate in milliseconds
   }
}
```
##### Message.TextMessage
```json5
{
   "type": "Message.TextMessage.ID",
   "data": {
      "text": "Text message",
      "user": "Username",
      "time": "Message timestamp"
   },
   "room": 1 // Room id
}
```
##### Message.ImageMessage
```json5
{
   "type": "Message.ImageMessage.ID",
   "data": {
      "image": "url",
      "user": "Username",
      "time": "Message timestamp"
   },
   "room": 1 // Room id
}
```
##### File.ImageDownloadMeta
```json5
{
   "type": "File.ImageDownloadMeta.ID",
   "data": {
      "id": "Image id",
      "name": "Image name",
      "size": 123456789, // Image size in bytes
      "type": "Image type", // Mime type
   }
}
```
##### File.ImageDownloadPart
```json5
{
   "type": "File.ImageUploadPart.ID",
   "data": {
      "id": "Image id",
      "part": 1, // Part number
      "data": "Base64 encoded image part"
   }
}
```
##### File.ImageDownloadCompleted
```json5
{
   "type": "File.ImageUploadCompleted.ID",
   "data": {
      "id": "Image id",
   }
}
```
##### File.ImageUploadRejected
```json5
{
   "type": "File.ImageUploadRejected.ID",
   "data": null
}
```
##### File.ImageDownloadAccepted
```json5
{
   "type": "File.ImageDownloadAccepted.ID",
   "data": {
      "id": "Image id",
      "name": "Image name",
      "size": 123456789, // Image size in bytes
      "type": "Image type", // Mime type
   }
}
```
##### User.Created
```json5
{
   "type": "User.Created.ID",
   "data": {
      "username": "Username"
   }
}
````
##### User.Deleted
```json5
{
   "type": "User.Deleted.ID",
   "data": {
      "username": "Username"
   }
}
```
##### Room.Created
```json5
{
   "type": "Room.Created.ID",
   "data": {
      "room": 1, // Room id
      "name": "Room name"
   }
}
```
##### Room.Joined
```json5
{
   "type": "Room.Joined.ID",
   "data": {
      "room": 1, // Room id
      "user": "Username",
      "allUsers": ["Username", "Username", "Username"]
   }
}
```
##### Room.Left
```json5
{
   "type": "Room.Left.ID",
  "data": {
     "room": 1, // Room id
     "user": "Username"
  }
}
```
##### Room.Deleted
```json5
{
   "type": "Room.Deleted.ID",
   "data": {
      "room": 1, // Room id
   }
}
```
##### Room.Updated
```json5
{
   "type": "Room.Updated.ID",
   "data": {
      "room": 1, // Room id
      "name": "Room name"
   }
}
```
##### Room.List
```json5
{
   "type": "Room.List.ID",
   "data": [
      {
         "room": 1, // Room id
         "name": "Room name"
      }
   ]
}
```
##### Error
```json5
{
   "type": "Error.MinorType.ID",
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
1. Upload image to server and get URL (or upload image to any host and get direct URL)
2. Client sends Message.CreateImageMessage with CreateImageMessage object
3. Server sends Message.ImageMessage with ImageMessage object

If your client is CLI, you can not implement image sending/receiving, because how :trl:
If message is invalid:
1. Server sends Error.MessageInvalid with Error object

If room don't exist:
1. Server sends Error.RoomDontExists with Error object

### Users
After authentication
If you want to create user:
1. Client sends User.Create with UserCreate object
2. Server sends User.Created with UserCreated object

If you want to delete user:
1. Client sends User.Delete with UserDelete object
2. Server sends User.Deleted with UserDeleted object

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

### Heartbeat
After authentication, you must send Heartbeat.
If you don't send heartbeat, server will disconnect you.
In Authentication.Accepted packet you can get heartbeat interval.
To send heartbeat, you must send Heartbeat packet.
```json5
{
   "type": "Heartbeat.Ping.ID"
}
```
Server will don't response to this packet.

If server want to disconnect you, he will send Error.HeartbeatTimeout packet.


## Encryption
### Diffie-Hellman
Diffie-Hellman is used for key exchange. <br>
MCP Uses modp5 group. <br>

### AES-256-CTR
AES-256-CTR is used for encryption. <br>
MCP uses 16 bytes IV filled by zero-bytes for all messages. <br>
MCP uses 32 bytes key for encryption. This name is MCP Key <br>

## Versions
### 1.0.0
The first version of MCP. <br>
Very simple, without rooms, pls don't use it.

### 2.0.0
Current version of MCP. <br>
Added rooms, but without private rooms. <br>

### 2.1.0
Removed HTTP, only TCP. <br>
Added Ids to packets. <br>


## Plans
- [ ] Add private rooms
