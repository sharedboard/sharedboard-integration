### RoomRequest
``` Typescript

// Request object for creating a room.
interface RoomRequest {
  name: string;
  description?: string;
  isPublic?: boolean;

  // public rooms may be restricted by a passcode. If not used, leave this empty
  passcode?: string;

  // user id of owner of this room (most probably the user which created this room)
  owner?: string;

  // info about users that will participate in this room. At minimum it is advised to add owner information
  userData?: SharedboardUserSummary[];

  
  videoTopology?: VideoTopology;

  restrictions?: RoomRestrictions;
  permissions?: RoomPermissions;

  // specify which permissions terminals will have while joining
  terminalPermissionsOnJoin?: TerminalPermissions;

  // user ids of participants of this room
  participantIds?: string[];

  // Emails of participants of this room.
  participantEmails?: string[];

  // Groups which are allowed to join this room. Relevant only if this room is private.
  groups?: string[];
  
  // Initial content of the board. Don't set this unless you know what you are doing
  initialContent?: any;
}
```


``` Typescript
/**
  * mesh: every user can see each others camera(if camera is switched on, of course)
  * star: Users can only see room manager's camera, room manager can see every user
  */
type VideoTopology = 'mesh' | 'star';
```


``` Typescript
/**
 * Specifies restrictions that will be applied to rooms. 
 * Setting a number value to 0 means, there is no restriction for that value.
 * By default there is no restriction for a room other than the restrictions for a Tenant (host application)
 * 
 * Host applications may specify restrictions to their users while creating rooms.
 */
interface RoomRestrictions {
  maxSessionDurationInSeconds?: number;
  maxTotalDurationInSeconds?: number;
  maxNumberOfAttendees?: number;
  maxNumberOfVideoPresenters?: number;
  maxSingleFileSizeInKB?: number;
  maxTotalFileSizeInKB?: number;
  cameraAllowed?: boolean;
  microphoneAllowed?: boolean;
  screenShareAllowed?: boolean;
  chatAllowed?: boolean;
  fileUploadAllowed?: boolean;
  allowedVideoTopologies?: VideoTopology[];
  allowedRtcTypes?: RtcType[];
}
```

``` Typescript
interface RoomPermissions {
  // Specifies if users are allowed rename themselves
  overrideName?: boolean;
}
```


### RoomData
``` Typescript

/**
 * Interface representing a Room
 *
 */
interface RoomData {
  tenantId: string,
  name: string,
  description: string,
  isPublic: boolean,
  passcode?: string,
  createdAt: number,
  lastActiveDate?: number
  owner: string,
  active: boolean,
  boards: any,
  activeBoardId: string,
  terminals: any,
  participantEmails: string[],
  participantIds: string[],
  groups?: string[];
  permissions: RoomPermissions,
  terminalPermissionsOnJoin: TerminalPermissions,
  syncNavigation: boolean,
  master?: string,
  videoPresenters: string[],
  screenSharePresenter?: string,
  videoTopology: VideoTopology,
  restrictions: RoomRestrictions,
  userInfo: { [key: string]: SharedboardUserSummary },
  sessionId?: any,
  tags?: string[],
  rtc: {
    group?: string; 
    apiUrl?: string;
    wsUrl?: string;
    type: RtcType;
  }
  _v: number,
  _id: string,
}
```

``` Typescript
interface RoomSummary {
  _id: string,
  name: string,
  description?: string,
  ownerId: string,
  ownerName?: string,
  active: boolean,
  createdAt: number,
  lastActiveDate?: number,
  isPublic: boolean,
  groups?: string[],
  participantEmails: string[],
  participantIds: string[],
  tags?: string[]
}
```

``` Typescript
interface TerminalPermissions {
  edit: boolean;
  uploadFile: boolean;
  camera: boolean;
  chat: boolean;
  microphone: boolean;
  navigate: boolean;
  screenShare: boolean;
  modifyBoards: boolean;
  createPoll: boolean;
}
```

``` Typescript
type RoomListType = 'own' | 'participate' | 'group';
```

``` Typescript
interface ModifyRoomParticipantsRequest {
  participantIdsToAdd?: string[];
  participantIdsToRemove?: string[];
  participantEmailsToAdd?: string[];
  participantEmailsToRemove?: string[];
}
```

``` Typescript
interface ModifyRoomGroupsRequest {
  groupsToAdd?: string[];
  groupsToRemove?: string[];
}
```

``` Typescript
enum RoomDisconnectReason {
  INITIAL_DISCONNECT = 'INITIAL_DISCONNECT',
  CLIENT_REQUEST = 'CLIENT_REQUEST',
  MASTER_REQUEST = 'MASTER_REQUEST',
  NETWORK_ERROR = 'NETWORK_ERROR',
  AUTHENTICATION_ERROR = 'AUTHENTICATION_ERROR',
  PERMISSION_DENIED = 'PERMISSION_DENIED',
  SERVER_ERROR = 'SERVER_ERROR',
  MAX_TERMINAL_LIMIT_REACHED = 'MAX_TERMINAL_LIMIT_REACHED',
  SESSION_TERMINATED = 'SESSION_TERMINATED'
}

```



### SharedboardJoinOptions
``` Typescript

interface SharedboardJoinOptions {
  /**
   * Optional, should not be specified unless connecting to a different Sharedboard server
   */
  boardApiUrl?: string;

    
  /**
   * Optional, should not be specified unless connecting to a different Sharedboard server
   */
  boardWSUrl?: string;

  /**
   * Id of the room that is being connected
   */
  roomId: string;

  /**
   * JWT authentication token that is used to connect Sharedboard. Unlike the REST api calls
   * the token should not have 'Bearer ' prefix here. Just the raw token should be specified
   * without a header name etc. This token should be generated for current user with 'USER' role
   */
  boardApiToken: string;

  /**
   * If the room has a passcode, it can be specified here, so that, Sharedboard UI will not ask for the password.
   * Client applications may prefer to embed the passcode in the room url, so this would be
   * helpful to provide the password without asking to user, by reading from url
   */
  passcode?: string;

  /**
   * Optional. Default is 'en-US'. Currently only supports:
   * - en-US
   * - tr-TR
   */
  language?: string;


  /**
   * The name of host application. Some screens require to display application name. Write the name
   * of your applicatiion here.
   */
  applicationName?: string;


  /**
   * Specifies if the invite button should be rendered on Sharedboard UI. Default is false. If set to true,
   * Host application should also implement the invitation dialog
   */
  inviteSupported: boolean;
}

```
