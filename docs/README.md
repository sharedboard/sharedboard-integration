# Integrating Sharedboard 

## Introduction
Sharedboard is a virtual classroom environment. It is a Saas platform which makes it possible to integrate with 3rd party applications seamlessly. Sharedboard is a web based platform. Any application that supports rendering web pages may integrate Sharedboard and use it to provide collaborative whiteboard and video conferencing features to its users.

3rd party applications integrating Sharedboard will be called **Host Application** hereafter.

Sharedboard system has different components to make this service available. Among those, host applications will need to interact with the following 2 components:
- **Sharedboard Room Server**: This is the server which creates and manages rooms and whiteboards and handles all other server side functionality. Throughout this document, this server may be called as *Sharedboard API* or just *Sharedboard*
- **Sharedboard Client**: This is the web UI that renders the rooms and all other stuff like whiteboards, video conferencing, chats, etc.

Host applications calls the APIs from **Sharedboard Room Server** to create and manage rooms as well as reading room details, statistics and history of a room. To display the virtual classroom UI, host applications will need to render the **Sharedboard Client**, most probably by using an `iframe` html element in a web based environment or some kind of *WebView* element (may differ depending on the platform) on mobile clients.

Below are the contents of this documentation. If you've not previously checked the documentation, we think it is best to read in given order.

## Contents

1. [Communication with Sharedboard and Authentication](./communication.md)
2. [Creating and Displaying Rooms](./room-operations.md)


## Appendix
1. [Room types](./room-types.md)
2. [Authentication types](./auth-types.md)
