
## Creating and Displaying Rooms
Rooms in Sharedboard are the virtual equivalent of a classroom. Users with specific [Scopes](./auth-types.md) may create and delete rooms, read room data and room history.

Rooms can be public or private. Only owner of the room, specified participants and specified groups can join a private room. Any user can join a public room. Either public or private, all rooms require an authentication token while joining.

Whether a user can create a room or not is specified with [Scopes](./auth-types.md). Scopes are specified in the `scopes` array of the authentication token. To create a room, a user should have 'CREATE_USER' scope in the authentication token. (*Adding this scope to users and enabling them to directly create rooms may not be the best option, keep reading... ;)* )

### Creating A Room
To create a room, a host application should issue the following API call to Sharedboard Room Server:
``` HTTP
POST /rooms
```
The POST request body should contain a [`RoomRequest`](./room-types.md#roomrequest) object. Details of `RoomRequest` can be seen [here](./room-types.md#roomrequest). A `RoomRequest` at minimum should contain a room name which is denoted by the field `name`. And remember that this POST request should contain the authentication token in `Authorization` header. If you have not already checked, see authentication details [here](./communication.md)

If the role in the authentication token is `'USER'` then the owner of the room is set as the `userId` in the authentication token regardless of the `owner` value in the `RoomRequest`. If the role in authentication token is `'SYSTEM'` then room owner should be specified in the `owner` field of `RoomRequest` and it should be the `userId` of the owner user.

A successful call to this API will return the [`RoomData`](./room-types.md#roomdata) object that represents the created room. Host application will need the room id (the `_id` field) from this data, to join the room.

#### Should the APIs be called from Client or Server?
As explained in previous section, the room is created differently depending on the role. So this is a good time to talk about how these APIs should be called by host applications. Host applications may call this API either from their clients, or server side. If this API is called from Host Application client, then that client will need the Sharedboard authentication token which has the role 'USER'. How that client will obtain that authentication token depends on Host Application architecture and design. Clients should not generate this token by themselves. The reason is, token generation requires **Application Secret Key** and you should NEVER EVER save or put this key on clients. This key should be kept as an absolute secret on server side, since a user having this key means, that user can perform any action on behalf of your application without any restriction.

To repeat it, Host Applications can call Sharedboard APIs from server or client side. If the APIs are called from client side, Host Application's server should generate an authentication token for the client with `role: ['USER']` and with a reasonable expiration time. This token should be sent to clients from server. And client will use it to access SharedBoard APIs. If Sharedboard APIs will be called from Host Application's server, then it should generate the token for itself with `role: ['SYSTEM']`.

Note that `USER` role is restricted by the scopes specified in authentication token and is also resticted to operate on own rooms (and sometimes participating rooms, depending on the operation). 'SYSTEM' role can unrestrictedly call any API and access any room data.

Also note that, depending on the use case, calling the room creation API from client side may or may not be a good decision. If you want to control room creation and put any restrictions on room properties, then you should not allow your clients to call this API directly, which means you should not add 'CREATE_ROOM' scope to clients authentication tokens. In this case you should call the room creation APIs from your server. Your clients send a room creation request to your server and then you call Sharedboard APIs from your server with controlled parameters. The APIs for getting/listing room data are usually safer to call from client side.

### Rendering Room UI
**Sharedboard Client** is the web application that renders the room UI. Host applications should render **Sharedboard Client** web page by loading it from its URL. How this page will be rendered depends on the type of client application. If the client application is a web application, then an HTML `iframe` element is the best option. Other clients (such as react native, flutter, electron, native code, etc) should probably use an in-app browser or a WebView (or usually similarly named) component to render this web page. Note that, currently official support is only for web clients. That said, any technology that can render web pages with modern rendering engines should be OK. And that's available almost in all platforms. 

**Sharedboard Client** is served from the URL: **<https://client.sharedboard.io>**

By default, loading this page displays nothing, since rendering the room UI requires some parameters such as room id and authentication token. Rest of this document will explain how to integrate the room UI to a web client, however this can be adapted to other clients in a similar fashion.

To render the room UI, host application client should include an `iframe` element in the page where it wants to display the room UI. The `iframe` should be styled accordingly to have the desired width and height.

``` html
<iframe src="https://client.sharedboard.io" allow="microphone; camera; display-capture; screen-wake-lock"></iframe>
```
The iframe should be given following permissions to operate correctly:
- `microphone`
- `camera`
- `display-capture`
- `screen-wake-lock`

After placing the `iframe` to page, make sure it is visible on the page with desired width and height.

When the iframe is loaded, we should send the `join` message to iframe window. Following snippet shows how to send this message.

``` html
<script>
  const roomId = ...; // room id to join
  const authenticationToken = ...; // Sharedboard authentication token generated for this client

  function joinRoom(roomFrame) {
    roomFrame.contentWindow.postMessage({
        action: 'join',
        options: {
          roomId: roomId,
          boardApiToken: authenticationToken,
          applicationName: 'My Awesome Application',
        }
      }, '*');
  }
</script>

.....

<iframe onload="joinRoom(this)" src="https://client.sharedboard.io" allow="microphone; camera; display-capture; screen-wake-lock"></iframe>

```

If everything is OK, at this point you should see **Sharedboard Client** joining the room. As seen in the example we specify the room Id and authentication token in the `options` field. All of the join options are [listed here](./room-types.md#sharedboardjoinoptions)

### Listening Room Message Events
**Sharedboard Client** Room UI sends some messages to host application in certain conditions. Host application clients should listen for these messages and handle them properly. **Sharedboard Client** sends the messages by calling `window.postMessage` so, host application should listen for **Sharedboard Client** messages in following way:

``` javascript
  window.addEventListener('message', function(evt: MessageEvent) {
    const { data } = evt;

    if (evt.origin !== 'https://client.sharedboard.io') {
      return;
    }

    if (data.type === SHOW_INVITE_DIALOG) {
      ..... // display user invitation dialog
    } else if (data.type === ROOM_DISCONNECTED) {
      ..... //display room disconnected message
    }
  });
```

The messages send from **Sharedboard Client** are as follows:

**SET_TITLE**: Host application client should set its title. This message is sent when the user wants to record session. When record button is pressed, the borwser asks for which application window or tab to record and lists the window titles to select from. Sharedboard sets the title here to help users to select correct window. The data property of the MessageEvent is as follows:
``` javascript
{
  "type": "SET_TITLE",
  "title": ... // The string value, which will be set as document title
}
```

**RESTORE_TITLE**: Host application client should set its title to the original value which was before 'SET_TITLE' message
``` javascript
{
  "type": "RESTORE_TITLE"
}
```

**SHOW_INVITE_DIALOG**: The user clicked on invite button on **Sharedboard Client**. Host application should display user invitation dialog, and after the user selects participants to invite, the host application should call participant modification APIs on the server for selected participants if necessary
``` javascript
{
  "type": "SHOW_INVITE_DIALOG",
  "roomData": RoomData, // The room object for the current room
  "isMaster": boolean // whether invite functionality is called by the room manager(master user) or not
}
```

**ROOM_DISCONNECTED**: Room is disconnected. Show proper messages/UI to user taking the disconnect reason into consideration.
``` javascript
{
  "type": "ROOM_DISCONNECTED",
  "reason": RoomDisconnectReason, // disconnect reason
}
```
