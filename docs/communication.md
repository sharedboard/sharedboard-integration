
## Communication with Sharedboard Room Server
Host applications communicates with **Sharedboard Room Server** through its REST API.
Sharedboard API documentation is available as OpenAPI specification here: <https://api.sharedboard.io/api-docs/>  

Communication with **Sharedboard Client** is via `window.postMessage` calls when required.

All API calls require authentication so before making any API call we should learn how to authenticate with Sharedboard.

### Authentication
Sharedboard uses **JWT** tokens to authenticate clients. To access Sharedboard functionality, host applications should create a JWT authentication token and then the token should be added as `Authorization` http header in the API calls with `Bearer ` prefix.
```http
Authorization: Bearer YOUR_JWT_TOKEN_HERE
```
This is a pretty common authentication mechanism for REST APIs nowadays and lots of documentation can be found on internet about details of creating **JWT** tokens. There are also many JWT libraries for most programming languages. Examples in this document are with `Javascript` or `Typescript` and are using `jsonwebtoken` library to create JWT tokens.

To create JWT authentication tokens for Sharedboard, host applications will need an **Application Id** and a  **Application Secret Key**. These credentials can be obtained from Sharedboard support (<support@sharedboard.io>) until application registration pages are published in <https://sharedboard.io>.

**Important Note!!!**
You should not share **Aplication Secret Key** with someone else and should NEVER EVER save or put this key on a client side application. This key should be kept as an absolute secret on server side, since a user having this key means, that user can perform any action on behalf of your application without any restriction.

#### Creating JWT Token
The JWT claims should be in format of [`SharedboardAuthClaim`](./auth-types.md). Please see [`SharedboardAuthClaim`](./auth-types.md) page for details of fields.

Following is an example in Typescript/Nodejs to create a JWT token for Sharedboard APIs:

``` Typescript
const YOUR_APP_ID = ...;
const YOUR_APP_SECRET_KEY = ...;

/**
 * Creates a JWT token for given user info
 **/
public createSharedboardAccessTokenForUser(userId: string, name: string, email: string, scopes: Scope[]) {
  const claims: SharedboardAuthClaim = {
      userId,
      name,
      roles: [Role.USER],
      email,
      scopes,
      app: YOUR_APP_ID,
      exp: new Date().getTime() + 1000 * 60 * 60 * 24; // 1 day from now,
  };

  // sign the claims with your secret key. The result will be your JWT token
  return jwt.sign(claims, YOUR_APP_SECRET_KEY);
}

.....
.....

const user = ...; //get user info
const token = createSharedboardAccessTokenForUser(user.id, user.name, user.email, [Scopes.READ_ROOM, Scopes.ROOM_HISTORY]);

// Add JWT token to HTTP 'Authorization' header with 'Bearer ' prefix while calling Sharedboard APIs
https.request(
  {
    headers: {
      'Authorization': 'Bearer ' + token
    },
    hostname: 'api.sharedboard.io',
    port: 443,
    path: '/rooms',
    method: 'GET',
  },
  function() {
    ...
  }
);

```

And that's it. We're now ready to call Sharedboard APIs.
