# Elixir

Phoenix LiveView is a library for the Elixir programming language that allows developers to create real-time, reactive web applications without the hard client/server split. This eliminates the need for complex APIs (REST or Graphql) and is inherently ‘live’ in that server-side changes can be propagated to the client-side in real time.

It allows the developer to write front-end code using the familiar Elixir language and uses WebSockets to automatically update the user interface in real-time without the need for writing any additional code for managing WebSockets or implementing complex client-server communication logic

> It has the disadvantage of being more difficult to ‘think in’ because it is not comparable to an SPA application. This is probably why I have tended to avoid it.

[Elixir/Phoenix ecosystem](https://www.notion.so/3919a0199ce84ae48496e3927ed0133e)

# Mix

Mix is the **Swiss Army knife** tool for Elixir applications. It can be compared to the npm client tool (not the repo) in that it uses a `Mix.Project` module in the same way npm uses package.json.

With this file defined, we can install dependencies, build, test and run an application using built-in tasks.

You can also add additional tasks by creating a module that ‘uses’ `Mix.Task`

To install dependencies, mix can use Hex

# Hex

Hex is the package manager for Erlang and Elixir applications. It can be installed locally, and made available to mix, by using the command `mix local.hex`. This will install a precompiled version.

If the precompiled version does not work,

# Pages

When the user navigates to a URL, a single liveview page is rendered.

# Rendering

Rendering is a two step process. Firstly a normal HTML page is loaded (step 1), which then connects to the server (step 1).

During both step 1 and step 2, the params() and mount() functions are called.

# Sockets

A socket is a persistent transport layer connection between the server and a single client. The socket is managed by a process on the server - there is one process per client. Messages can be initiated by both the client and server (as distinct from HTTP where messages on initiate on the client). Heartbeats are used to ensure that the connection is still ‘alive’.

On the server, the socket is exposed as a URL which client-side Javascript can access. The connection is established in 2 steps. First a normal HTTP request is made to the URL, then an ‘upgrade’ request is made to establish the persistent websocket connection.

The makes authentication a 2 step process. Firstly, we authenticate in the :browser pipeline which will establish the user. We then extract some unique data about that user to be signed and inject it - as a token - in the returned HTML. Below, we have selected user.id. Secondly, we reflect this token

​

⚡ It is critical that this signed data not identify the user (a primary key is probably ok as it is random letters) or contain sensitive information (credit cards etc).

​

</aside>

## Server Side

socket "/socket", HelloWeb.UserSocket,  
  websocket: true,  
  longpoll: false

When we load a page, the plugs in the browser pipeline are executed, and the authentication code injects the user into the conn. The user is then used to inject a user_token into the conn.

pipeline :browser do  
  ...  
  plug UserFromSession  
  plug :put_user_token  
end  
​  
defp put_user_token(conn, _) do  
  if user = conn.assigns[:user] do  
    token = Phoenix.Token.sign(conn, "user socket", user.id)  
    assign(conn, :user_token, token)  
  else  
    conn  
  end  
end

The user_token is then injected into the layout file HTML as a one line script which sets a window variable called `userToken`. The client side will pluck this out of the window variable.

<script>window.userToken = "<%= assigns[:user_token] %>";</script>

The server-side UserSocket then authenticates the user using this token in the connect() callback. An id() callback can also be used to tie this socket to a specific user (there is one socket per connected user).

defmodule HelloWeb.UserSocket do  
  use Phoenix.Socket  
​  
    # route room messages to the RoomChannel  
  channel "room:*", HelloWeb.RoomChannel  
​  
    # use params to authenticate the user and inject into socket  
  @impl true  
  def connect(%{"token" => token}, socket, _connect_info) do  
        case Phoenix.Token.verify(socket, "user socket", token, max_age: 1209600) do  
        {:ok, user_id ->  
          {:ok, assign(socket, :user_id, user_id)}  
        {:error, reason} ->  
          :error  
        end  
  end  
​  
  # Socket id's are topics that allow you to identify all sockets for a given user:  
  # Returning `nil` makes this socket anonymous.  
  @impl true  
  #     def id(socket), do: "user_socket:#{socket.assigns.user_id}"  
  def id(_socket), do: nil  
end

## Client Side

On the client, the socket is an object created using the following code. The `userToken` injected server-side is sent in the params to authenticate the user.

let socket = new Socket("/socket", {params: {token: window.userToken}});  
socket.connect();  
​  
socket.onError( () => console.log("there was an error with the connection!") )  
socket.onClose( () => console.log("the connection dropped") )  
socket.onOpen(() => {  
    socket.onOpen(() => {  
    console.log('the connection is open');  
    // Now that you are connected, you can join channels with a topic.  
    // Let's assume you have a channel with a topic named `room` and the  
    // subtopic is its id - in this case lobby:  
    let channel = socket.channel('room:lobby', {});  
    let messagesContainer = document.querySelector('#messages');  
​  
    channel.on('new_msg', (payload) => {  
        let messageItem = document.createElement('p');  
        messageItem.innerText = `[${Date()}] ${payload.body}`;  
        messagesContainer.appendChild(messageItem);  
    });  
​  
    channel  
        .join()  
        .receive('ok', (resp) => {  
            console.log('Joined successfully', resp);  
            let chatInput = document.querySelector('#chat-input');  
​  
            chatInput.addEventListener('keypress', (event) => {  
                if (event.key === 'Enter') {  
                    channel.push('new_msg', { body: chatInput.value });  
                    chatInput.value = '';  
                }  
            });  
        })  
        .receive('error', (resp) => {  
            console.log('Unable to join', resp);  
        });  
        });  
})

## Gotchas

​

⚡ A socket cannot set cookies.

​

</aside>

# Channels

Channels are the one of the mechanisms - liveview being the other - that send messages to, and receive messages from, a socket. The messages are routed from a socket to a channel based on their topic. Many users may join

Below is an example which routes messages starting with “room:” to the RoomChannel.

  channel "room:*", HelloWeb.RoomChannel

## Channel authorization (joining)

The Channel them implements authorization (as opposed to authentication which is performed in the socket). In the example below we use the user_id from the socket to find the %User{} structure and then ensure that only “Winston Smith” can enter room:101

```elixir
defmodule HelloWeb.RoomChannel do  
  use Phoenix.Channel   
  def join("room:lobby", _message, socket) do  
        # Let anyone access the lobby.  
    {:ok, socket}  
  end  

  def join("room:101", _params, _socket) do  
    # Only "Winston Smith" can enter room 101  
        case User.findById(socket.assigns[:user_id]) do  
            {:ok, %User{name: "Winston Smith"}} ->   
                {:ok, socket}  
            _ ->   
                {:error, %{reason: "unauthorized"}}  
  end  

  def handle_in("new_msg", %{"body" => body}, socket) do  
    broadcast!(socket, "new_msg", %{body: body})  
    {:noreply, socket}  
  end  
end
```

`defmodule HelloWeb.RoomChannel do  
  use Phoenix.Channel  
​  
  def join("room:lobby", _message, socket) do  
        # Let anyone access the lobby.  
    {:ok, socket}  
  end  
​  
  def join("room:101", _params, _socket) do  
    # Only "Winston Smith" can enter room 101  
        case User.findById(socket.assigns[:user_id]) do  
            {:ok, %User{name: "Winston Smith"}} ->   
                {:ok, socket}  
            _ ->   
                {:error, %{reason: "unauthorized"}}  
  end  
​  
  def handle_in("new_msg", %{"body" => body}, socket) do  
    broadcast!(socket, "new_msg", %{body: body})  
    {:noreply, socket}  
  end  
end`

## Message Flows

### Incoming Events using handle_in()

  def handle_in("new_msg", %{"body" => body}, socket) do  
    # do something with the message  
    {:noreply, socket}  
  end

### Outgoing Messages using push()

def handle_in("current_rank", _, socket) do  
  push(socket, "current_rank", %{val: Game.get_rank(socket.assigns[:user])})  
  push(socket, "photo", {:binary, File.read!(socket.assigns.photo_path)})  
  {:noreply, socket}  
end

### Direct Reply

reply with a JSON response

def handle_in("create:post", attrs, socket) do  
  changeset = Post.changeset(%Post{}, attrs)  
​  
  if changeset.valid? do  
    post = Repo.insert!(changeset)  
    response = MyAppWeb.PostView.render("show.json", %{post: post})  
    {:reply, {:ok, response}, socket}  
  else  
    response = MyAppWeb.ChangesetView.render("errors.json", %{changeset: changeset})  
    {:reply, {:error, response}, socket}  
  end  
end

or just reply with :ok

def handle_in("create:post", attrs, socket) do  
  changeset = Post.changeset(%Post{}, attrs)  
​  
  if changeset.valid? do  
    Repo.insert!(changeset)  
    {:reply, :ok, socket}  
  else  
    {:reply, :error, socket}  
  end  
end

### Broadcast within channel

Broadcast within the channel using the socket. This only broadcasts to the topic of the socket.

  def handle_in("new_msg", %{"body" => body}, socket) do  
    broadcast!(socket, "new_msg", %{body: body})  
    {:noreply, socket}  
  end