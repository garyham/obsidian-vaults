Liveviews load in a 2 stages.  The 1st stage is the http load, the 2nd stage is the websocket connection.  Once established all communication is via the websocket.  

A liveview could be particularly useful for [[Multi Factor Authentication]]

It is not possible to set cookies from a Liveview.  There is a pattern using `phx-trigger-action` which allows a standard form submit.

The component library [**surface**](https://surface-ui.org/getting_started) is an excellent addition to phoenix.

