resource:

[The Pragmatic Studio](https://pragmaticstudio.com/tutorials/getting-started-with-phoenix-liveview)

# What’s LiveView?

LiveView is a library that allows WebSocket communication without any function or mundane code, focusing on interactive real-time apps. It starts with a common HTTP request and the WebSocket communication starts when the server responds.  This allows that when the user interacts with any point of the page/app, the backend update or send the response just for the desired point.

# Create a project

To create a liveView project first create a normal phoenix project

```elixir
mix phx.new my_code_liveView
```

Notice that I need to setup the DB (ecto ORM) to run the server

```elixir
config :live_view_studio, LiveViewStudio.Repo,
  username: "postgres",
  password: "my_powerfull_password", #type the correct password 
  hostname: "localhost",
  database: "live_view_studio_dev",
  stacktrace: true,
  show_sensitive_data_on_connection_error: true,
  pool_size: 10
```

After all, create my data base with ecto

```elixir
mix ecto.create
```

Now I can start the server

```elixir
mix phx.server
```

# Add The live route

In the router I need to create a live endpoint with the handle function

```elixir
live "/light", LightLive #Create a live view at the endpoint /light
```

# Create a LiveView Module

I need to create a folder /live inside my project_web folder and create a endpoint_live.ex file in it. In this example, the folder looks like **`live_view_studio_web/live/`** and the file looks like **`light_live.ex`.** Inside create a module such as a controller

```elixir
defmodule LiveViewStudioWeb.LightLive do
  use LiveViewStudioWeb, :live_view
end
```

After creating a module I need to define some callbacks

- mount: Assign the initial state of the process
- handle_event: Change the state of the process
- render: Render the new view

## Define callbacks inside the module

### Mount

Mount is a callback that needs 3 input arguments

- params: Is a map that contains the query params
- Session: Is a map that contains the session key
- socket: Is a struct that represents the WebSocket connection

```elixir
def mount(_params, _session, socket) do
  socket = assign(socket, :brightness, 10) #Assign to my socket the keyword list brightness
  {:ok, socket} #Return an ok and a map with the socket information 
end
```

### Render

Render as its name say, render the template with the information that I need to update, Here I type the template or call the function to render a complex template

```elixir
def render(assigns) do
    ~H"""
    <h1>Front Porch Light</h1>
    <div class="meter">
      <span style={"width: #{@brightness}%"}>
        <%= @brightness %>%
      </span>
    </div>

		<!-- Create buttons -->
    <button phx-click="off"> <!-- Call a Phoenix function event off -->
      Off
    </button>

    <button phx-click="down"> <!-- Call a Phoenix function event down -->
      Down
    </button>

    <button phx-click="up"> <!-- Call a Phoenix function event up -->
      Up
    </button>

    <button phx-click="on"> <!-- Call a Phoenix function event on -->
      On
    </button>
    """
  end
```

### Create the handle_event

The handle_event are phoenix functions that will be executed when occurs an event with a specific name, If I look the previous template has buttons with a phx-click property, the value of this element is the event name.  Here is an example of this function:

```elixir
def handle_event("on", _, socket) do #handle event for the "on" event
  #assign to my socket the keyword list brightness with the value 100 
	socket = assign(socket, :brightness, 100) 
  {:noreply, socket} #return this tuple with the socket structure 
end
```

If I need to read the value of the socket key word list for any reason here is an example:

```elixir
def handle_event("down", _, socket) do
	#Get the value of the brightness and substract 10
  brightness = socket.assigns.brightness - 10 
  socket = assign(socket, :brightness, brightness) #Assign to my socket the keyword list brighess
  {:noreply, socket} # Return the keyword list with the socket structure 
end
```

Notice something, I can insert in the keyword list tuple a anonymous function as the following example, this allow complex operations

```elixir
def handle_event("down", _, socket) do
#Store in socket a keyword list with the brightness and a function as a value 
  socket = update(socket, :brightness, fn b -> b - 10 end) 
  {:noreply, socket} #return the tuple
end
```