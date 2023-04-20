That a user can access a particular server page using
* a conventional HTTP request e.g. get or post
* an XMLHttpRequest e.g. Javascript fetch
* a websocket connection.

Each of these mechanisms poses a different problem for an authentication infrastructure.

Websockets do not transmit cookies from client to server for authentication
OAuth2 flows can only be initiated using an anchor tag i.e. no JS so bearer authentication headers are not possible.

<iframe width="560" height="315" src="https://www.youtube.com/embed/M6N7gEZ-IUQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>