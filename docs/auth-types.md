## SharedboardAuthClaim

```Typescript
interface SharedboardAuthClaim {
    /**
     * The id of a user in the host application. Each user accessing Sharedboard will need to have an auth token,
     * thus will need to have an id. Some applications may allow public access to rooms. Applications should still
     * need to create auth token for these public users probably by generating temporary unique `userId`s for these
     * users.
     */
    userId: string;


    /**
     * Name of the user. If empty, system asks the user to enter a name on joining the room.
     */
    name: string;


    /* Roles of the entity making the API call. If the API call is made by server of the host application,
     * then this should be ['SYSTEM'] which bypasses all auhtorizations checks. If this API call is made
     * by host application clients, then it should be ['USER'] which requires authorizing the user for the calls.
     * For example a token with 'SYSTEM' role can access and get all the rooms' data, while a token with 'USER'
     * role will be able to get data of rooms with room ownerId equal to his/her userId only. (Note although the
     * data structure is an array, currently it only makes sense to put a single value in it. But should still be
     * a JSON array)
     */
    roles: Role[];


    /**
     * The **Application Id** obtained from Sharedboard
     */
    app: string;


    /**
     * Email of the user. If the user is added to a room as a participant with his/her email, then this should not be empty.
     */
    email?: string;


    /**
     * Expiration time of this token in milliseconds since EPOCH
     */
    exp: number;


    /**
     * Indicates what a user can do with this token. See each API description for required scopes to that specific
     * API call. If the user has `'SYSTEM'` role, then this can be left as an empty array.
     */
    scopes?: Scope[];


    /**
     * Groups of the user. This is used while joining rooms. A group may be specified as participant of a private room.
     * So, users belonging to that group may join the room, even if they are not specified as participant by their
     * "userId"s or emails.
     */
    groups?: string[];
}
```

```Typescript
enum Role {
    SYSTEM = 'SYSTEM',
    USER = 'USER',
}
```

```Typescript
enum Scope {
    READ_ROOM = 'READ_ROOM',
    CREATE_ROOM = 'CREATE_ROOM',
    EDIT_ROOM = 'EDIT_ROOM',
    DELETE_ROOM = 'DELETE_ROOM',
    ROOM_HISTORY = 'ROOM_HISTORY',
    ROOM_STATS = 'ROOM_STATS',
    USER_HISTORY = 'USER_HISTORY',
    USER_STATS = 'USER_STATS',
}
```