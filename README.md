# Getting Started with Phoenix
Pluralsight | Nate Taylor

**NOTE:** This course came out in 2016. I am doing it using the most current versions of Elixir and Phoenix as of March 2019. Therefore, some of the code in my application differs from the tutorial on pluralsight

## 1: Course Overview
- We're building an event tracking application


----


## 2: What Is Phoenix

### 2.1: What Is Phoenix
- Based on Elixir which is written in Erlang
- Phoenix is NOT your application


### 2.2: Components of Phoenix
- Standard MVC Application Builder
- Other Components
  - Plugs
    - Allows you to interact with the conn
  - Channels
    - Allows soft real-time communication
- Prerequisites
  - Elixir
  - Node JS
  - PostgreSQL

### 2.3: Umbrella Application
- Below are all the commands you need to get started (once you have the prerequisite files)
```bash
# Create a new umbrella application
mix new application_name_umbrella --umbrella

# move into the apps folder of the application and run this to create a supervisor for your applications
mix new app_name --sup

# Create your Phoenix App with this command
mix phx.new app_name_web --no-ecto
# --no-ecto will omit Ecto from your Phoenix App. Which we want because we want the Supervisor interacting with the database, not the Phoenix app directly

# Start Your App at localhost:4000
mix phx.server

# Use IEX in your App
iex -S mix phx.server

```


----


## 3: Ecto Models & Migrations

### 3.1: What Is a Model

### 3.2: Installing Ecto

### 3.3: Create a Model

### 3.4: Creating a Database Table
- Create a Migration
```bash
mix ecto.gen.migration add_events_table
```
**NOTE:** You can define `up` or `down` functions *OR* a `change` function for a migration

### 3.5: Retrieving Data
- Here's a quick way to reload functions for testing them:

```iex
r(App.FunctionName)` to reload function
```

### 3.6: Adding a Record
- Here's an example of an event to add to your database
```iex
Rsvp.Events.changeset(%Rsvp.Events{}, %{date: "2016-11-22 00:00:00", title: "test"})
```

### 3.7: Seeding The Database


----


## 4: Controllers and Views

### 4.1: Fetching & Displaying Data
### 4.2: Creating an Event Route
### 4.3: Creating a Controller
### 4.4: Creating The Show Function
### 4.5: Creating a View
**NOTE:** Elixir/Phoenix know where to look for views and templates based on the name of the item.
- `event_controller` looks for `event_view` which looks for a folder named `event` to find files

### 4.6: Supplying Data To A View
### 4.7: Testing The View


----


## 5: Templates: Laying Out The View

### 5.1: Templates: Layout Out The View
- Templates are rendered using data from the view

### 5.2: Listing Events
- Create a page that lists all of our events

- In the `router.ex` file, define a new action that routes to the event controller and will handle showing the list
```elixir
get "/events", EventController, :list
```

- In the `event_controller.ex` file, add the `:list` function that will provide all the events
```elixir
  def list(conn, _params) do
    events = Rsvp.EventQueries.get_all
    render(conn, "list.html", events: events)
  end
```
- Create a new file called `list.html.eex` in the `events` folder and add this code to loop through the list of events provided:
*NOTE:* Since `list.html.eex` and `details.html.eex` both live in the `events` folder. They share the same `view` file and will both have access to the `format_date` function
```elixir
<h1>All Events</h1>
<ul class="list">
  <%= for e <- @events do %>
    <a href="<%= e.id %>">
      <div class="list-item">
        <h4><%= e.title %></h4>
        <p><%= e.location %></p>
        <p><%= format_date(e.date) %></p>
      </div>
    </a>
  <% end %>
</ul>
```


### 5.3: Sub-Templates
- I would refer to these as `partials` but Sub-Template is fine
*NOTE:* I added the underscore to the beginning of the file names to indicate that it's a partial. This is not necessary but encouraged

#### Partials Within a directory

- In the `list.html.eex` file, replace the markup concerning each specific event and replace it with this line:
```elixir
  <%= render("_summary.html", event: e) %>
```
- Then create a `_summary.html.eex` file in the `events` folder and add this content
```elixir
<a href="<%= @event.id %>">
  <div class="list-item">
    <h4><%= @event.title %></h4>
    <p><%= @event.location %></p>
    <p><%= format_date(@event.date) %></p>
  </div>
</a>
```

#### Shared Partials

- Create a new file called `shared_view.ex` and add this to it:
```elixir
defmodule RsvpWebWeb.SharedView do
  use RsvpWebWeb, :view
end
```
- Move the `_count.html.eex` file to a new folder in `templates` called `shared`
- Rename `@events` to `@list` in the file
- Update the render function in `list.html.eex` to say this:
```elixir
<%= render RsvpWebWeb.SharedView, "_count.html", list: @events %>
```


### 5.4: Helpers
- Routing Helpers
  - Simplify using routes on pages in your application
- Template Helpers
  - Simplify creating HTML elements


**Routing Helpers**
- Make sure to use `Routes` when specifying a route helper
- They rely on the name of a controller to populate, i.e. `event_controller` => `event_path`

Example:
```elixir
<%= Routes.event_path(@conn, :show, @event.id) %>
```

**Template Helpers**
- These are very useful for form elements

Example 1: This will create a standard `href` tag
```elixir
<%= link("Text", to: "https://google.com") %>
```

Example 2: This will create a form with a post method
```elixir
<%= link("Text", to: "https://google.com", method: :post) %>
```

Example 3: Form Input
```elixir
<%= number_input, f, :number, min: 10 %>
```
Set the type to `number_input`, `f` is the form that the element is a part of, `:number` is the identifier for the field, `min: 10` is a list of options


----


## 6: Putting It Together: Creating an Event

### 6.2: Creating The Form
- Create a Route for the form in `router.ex`, as well as a route for adding a new event
  - Make sure to place this line above the `/events/:id` one
```elixir
get "/events/new", EventController, :create
post "/events/new", EventController. :add
```

- Add the `:create` and `:add` functions in `event_controller.ex`
```elixir
def create(conn, _params) do
  render(conn, "create.html")
end

def add(conn, _params) do

end
```

- Create a `create.html.eex` file in the `events` folder and add this form:
```elixir
<%= form_for @changeset, Routes.event_path(@conn, :add), fn f -> %>
  <div class="form-control">
    <%= label f, :title, "Event Title" %>
    <%= text_input f, :title, class: "form-control" %>
  </div>
  <div class="form-control">
    <%= label f, :location, "Event Location" %>
    <%= text_input f, :location, class: "form-control" %>
  </div>
  <div class="form-control">
    <%= label f, :date, "Event Date" %>
    <%= text_input f, :date, class: "form-control", type: "datetime-local" %>
  </div>

  <div class="form-control">
    <%= label f, :description, "Event Description" %>
    <%= textarea f, :description, class: "form-control", rows: 5 %>
  </div>

  <%= submit "Add Event", class: "button" %>
<% end %>
```

- Add `phoenix_ecto` to the `mix.exs` file's list of dependencies
```elixir
{:phoenix_ecto, "~> 4.0"},
```

### 6.3: Handling User Submitted Data
- Temporary Solution: Redirect the user
This is in the `event_controller.ex`
```elixir
def add(conn, _params) do
  redirect(conn, to: Routes.event_path(conn, :list))
end
```

- When ready, we add the new event to our list with this:
  - Convert our date
  - Destructure the new ID from the events changeset
  - Redirect to the new event page
```elixir
  def add(conn, %{"events" => events}) do
    naive_event_date = convert_date(events["date"])
    events = Map.replace!(events, "date", naive_event_date)

    %{id: id} = Rsvp.Events.changeset(%Rsvp.Events{}, events)
    |> Rsvp.EventQueries.create

    redirect(conn, to: Routes.event_path(conn, :show, id))
  end

  def convert_date(date) do
    {_, new_date} = date <> ":00" |> NaiveDateTime.from_iso8601()
    new_date
  end
```
**NOTE:** Because I'm doing this course in 2019 and the original course is expecting you to use an outdated date format, I had to make some modifications to get `naive_datetime` working. The `convert_date` function adds seconds to the `local-datetime` supplied by the form, converts it to a Naive DateTime and returns that back. Once we have that, we replace the date in the event map before passing it into the changeset.

### 6.4: Handling Form Errors
- First, we change `insert!` to `insert` in `event_queries.ex`
```elixir
def create(event) do
  Repo.insert(event)
end
```

- Next, we change the `add` function in the `event_controller.ex`
```elixir
changeset = Rsvp.Events.changeset(%Rsvp.Events{}, events)

case Rsvp.EventQueries.create(changeset) do
  {:ok, %{id: id}} -> redirect(conn, to: Routes.event_path(conn, :show, id))
  {:error, reasons} -> create(conn, %{errors: reasons}) # Here, we call the create function again, passing it the errors
end
```

- After that, we change add another `create` function above our first one to handle errors
```elixir
def create(conn, %{errors: errors}) do
  render(conn, "create.html", changeset: errors)
end
```

- Then, we add error tag helpers to our form in `create.html.eex`
```eex
<div class="form-control">
  <%= label f, :title, "Event Title" %>
  <%= text_input f, :title, class: "form-control" %>
  <%= error_tag f, :title %>
</div>
```

**NOTE:** When you change `event_queries`, restart your application for the change to take effect

----


## 7: Plugs

### 7.1: What is a Plug
*A Plug is a function that adds information to the connection.* A Plug is a specification for composable modules in between web applications. It is also an abstraction layer for connection of different web servers

- 2 kinds of Plugs
  - Function Plug
    - Any function that takes two parameters and returns a connection struct
  - Module Plug
    - Implements two functions, `call/2` and `init/1`

### 7.2: Creating a Cookie
We need to create a way for users to log in to the site

- First, we add two new routes to the `router.ex`
```elixir
get "/login", LoginController, :index
post "/login", LoginController, :login
```

- Next, we create a new controller, `login_controller.ex`
```elixir
defmodule RsvpWebWeb.LoginController do
  use RsvpWebWeb, :controller

  def index(conn, _params) do
    render(conn, "login.html")
  end

  def login(conn, %{"login" => %{"username" => name}}) do
    expiration = 60*60*24*7
    conn
    |> Plug.Conn.put_resp_cookie("user_name", name, max_age: expiration)
    |> redirect(to: "/")
  end
end
```

- Then, we add a `login_view.ex` in the views folder
```elixir
defmodule RsvpWebWeb.LoginView do
  use RsvpWebWeb, :view
end
```

- After that, create a `login` folder in `templates` and create a `login.html.eex` file where you add this:
```elixir
<%= form_for @conn, Routes.login_path(@conn, :login), [as: :login], fn f -> %>
  <%= text_input f, :username %>
  <%= submit "Login" %>
<% end %>
```

- Add a navigation to your site
```eex
<ul>
  <li><%= link("Home", to: Routes.page_path(@conn, :index))%></li>
  <li><%= link("Events", to: Routes.event_path(@conn, :list))%></li>
  <li><%= link("Login", to: Routes.login_path(@conn, :index))%></li>
</ul>
```

- Show the cookie to someone who has a username
```eex
<%= if @conn.cookies["user_name"] do %>
  <span>
    Hello, <%= @conn.cookies["user_name"] %>!
  </span>
<% end %>
```


### 7.3: Creating a Plug
- We create a plug! It will be named `authorized_plug.ex` and live in the `lib` folder
  - This will be a module plug, that gets passed a username and then checks to see if there is a username defined. If so, then they will be able to continue, if not, they will be prompted to login
```elixir
defmodule RsvpWeb.AuthorizedPlug do
  import Plug.Conn
  import Phoenix.Controller

  def init(opts) do
    opts
  end

  def call(conn, _) do
    user_name = conn.cookies["user_name"]
    authorize_user(conn, user_name)
  end

  defp authorize_user(conn, nil) do
    conn
    |> redirect(to: "/login")
    |> halt
  end

  defp authorize_user(conn, _) do
    conn
  end
end
```

- With our Plug created, we can add it to the `router.ex` file
  - One option is to add it as its own pipeline and scope some routes to it
```elixir
pipeline :authorized do
  plug :browser
  plug RsvpWeb.AuthorizedPlug
end

...

scope "/events", RsvpWebWeb do
  pipe_through :authorized

  get "/", EventController, :list
  get "/new", EventController, :create
  post "/new", EventController, :add
  # Keep/:id the bottom so Elixir doesn't accidentally match on this when looking for /new
  get "/:id", EventController, :show
end
```

- The better solution, however, is to only call the plug when we need it using the `event_controller.ex` file
```elixir
plug RsvpWeb.AuthorizedPlug when action in [:create]
```


### 7.4: Passing a Parameter
- Plugs are used to place information into cookies, the session, or a header
- They can be accessed in pipeline, a controller or specific actions in a controller

- First, we add a guard to make sure our `user_name` matches a specific name in `authorized_plug.ex`
```elixir
def call(conn, name) do
  user_name = conn.cookies["user_name"]
  authorize_user(conn, user_name, name)
end

defp authorize_user(conn, nil, _) do
  conn
  |> redirect(to: "/login")
  |> halt
end

defp authorize_user(conn, user_name, name) when user_name === name do
  conn
end

defp authorize_user(conn, _, _), do: authorize_user(conn, nil, nil)
```
- Then we add our desired name to the Plug call in `event_controller.ex`
```elixir
plug RsvpWeb.AuthorizedPlug, "create" when action in [:create]
```


----


## 8: Channels: Real-Time Communication

### 8.1: Channels: Real-Time Communication
Channels are based soley on sending and receiving messages
- Channels can have many receivers for messages
- There can be various consumers as well

Socket - Handles message dispatching

These are placed in the socket module
**Examples**
```elixir
channel "room:*", RsvpWeb.RoomChannel

channel "room:dogs", RsvpWeb.DogChannel
channel "room:cats", RsvpWeb.CatChannel
```

**Four Functions in Channels**
`join/3`
`terminate/2`
`handle_in/3`
`handle_out/3`

**Broadcast Message**
- Topic
- Event
- Payload


### 8.2: Accepting Reservations

Add a new field to the `events.ex` model schema
```elixir
field :quantity_available, :integer, default: 25
```

Run a Migration
```bash
mix ecto.gen.migration add_quantity_available
```

Add a change function to the newly created migration
```elixir
defmodule Rsvp.Repo.Migrations.AddQuantityAvailable do
  use Ecto.Migration

  def change do
    alter table(:events) do
      add :quantity_available, :integer, default: 25
    end
  end
end
```

Run the migration
```bash
mix ecto.migrate
```

Add a function to `event_queries.ex`
```elixir
def decrease_quantity(id, quantity) do
  event = Repo.get!(Events, id)
  changes = Ecto.Changeset.change event, quantity_available:
  event.quantity_available - String.to_integer(quantity)
  Repo.update changes
end
```

Add the quantity and this form to the Event Details:
```eex
<h4>
  <span class="quantity"><%= @event.quantity_available %></span>
</h4>


<%= form_for @conn, Routes.event_path(@conn, :reserve, @event.id), [as: reservation], fn f -> %>
  <div class="form-group">
    <%= label f, :quantity, "Quantity" %>
    <%= number_input f, :quantity, min: 1 %>
  </div>

  <%= submit "Reserve" %>
<% end %>
```

Add this line to `router.ex`
```elixir
post "/:id/reserve", EventController, :reserve
```

Add this function in `event_controller.ex`:
```elixir
def reserve(conn, %{"id" => id, "reservation" => %{"quantity" => quantity}}) do
  Rsvp.EventQueries.decrease_quantity(id, quantity)
  redirect(conn, to: Routes.event_path(conn, :show, id))
end
```


### 8.3: Preparing For Channels
In this module, he uses bower to add jQuery as a dependency to brunch, however, the newest version of Phoenix is utilizing Webpack so none of this is very relevant.


### 8.4: Creating a Channel
Add a new channel to your `user_socket.ex` file
```elixir
channel "event:*", RsvpWebWeb.EventChannel
```

Create that channel (here it's `event_channel.ex`) in the `channels` folder
```elixir
defmodule RsvpWebWeb.EventChannel do
  use Phoenix.Channel

  def join("event:" <> event_id, _message, socket) when event_id <= 0 do
    {:error, %{reason: "Invalid Event ID"}}
  end

  def join("event:" <> event_id, _message, socket) do
    {:ok, socket}
  end
end
```

Create a JavaScript file for interacting with the channel. I've created one in `js/modules/event.js`
```javascript
import socket from "../socket";

export default class Event {
  constructor () {
    let idElem = document.querySelector('#id');
    if (!idElem) { return; }
    this.id = idElem.getAttribute('data-id');
    this.build();
  }

  build () {
    this.createChannel();
    this.joinChannel();
  }

  createChannel () {
    this.channel = socket.channel(`event:${this.id}`, {});
  }

  joinChannel () {
    this.channel.join()
    .receive("ok", resp => { console.log(`Joined successfully event: ${this.id}`, resp) })
    .receive("error", resp => { console.log(`Unable to join`, resp) });
  }
}
```

Add an element that will provide an event id to the Javascript in `event/details.html.eex`
```eex
<span id="id" data-id="<%= @event.id %>"></span>
```

### 8.5: Consuming a Message

**NOTE:** DO NOT PUT A SPACE BETWEEN "event:" and "#{event.id}"

Create a new function in your `event_channel.ex` file for sending the update
```elixir
def send_update(event) do
  payload = %{"quantity" => event.quantity_available}
  RsvpWebWeb.Endpoint.broadcast!("event:#{event.id}", "update_quantity", payload)
end
```

Call that function from your `event_controller.ex` file
```elixir
def reserve(conn, %{"id" => id, "reservation" => %{"quantity" => quantity}}) do
  {:ok, event} = Rsvp.EventQueries.decrease_quantity(id, quantity)
  RsvpWebWeb.EventChannel.send_update(event)
  redirect(conn, to: Routes.event_path(conn, :show, id))
end
```


Observe the channel in your JavaScript and change the quantity amount as it updates
```javascript
observeChannel () {
  this.channel.on("update_quantity", event => {
    console.log(event);
    this.quantity.innerText = event.quantity;
  });
}
```

----


## 9: Deployment

### 9.1: Deployment

- Create a Heroku account
- Add the Heroku CLI
- Login to Heroku
- Make sure to track your file with Git

### 9.2: Creating The Heroku Application
[Additional help found here](https://hexdocs.pm/phoenix/heroku.html)

[Download the Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli)

Run these command / Add These Buildpacks to your site
```bash
heroku create --buildpack "https://github.com/HashNuke/heroku-buildpack-elixir.git"
heroku buildpacks:add https://github.com/gjaldon/heroku-buildpack-phoenix-static.git
```


### 9.3: Configuring The Application

#### Telling Heroku Where to Look & What To Do
Create a file named `phoenix_static_buildpack.config` and add these lines to set the path for your app and set up a compile file
```
phoenix_relative_path=apps/rsvp_web
compile="compile"
```

Create a file named `compile` and add these lines to run scripts that will start the app
```
npm run deploy
cd $phoenix_dir
mix "${phoenix_ex}.digest"
```

Add these lines to the `config/prod.exs` file:
- Load the secret key from Heroku
- Connect to the right database
- Bind to Heroku's Dyno Networking by setting the `http` port
- Set the `url` to your Heroku App's URL (you can also change the host to be a custom domain)
```elixir
config :hello, HelloWeb.Endpoint,
  http: [port: System.get_env("PORT")],
  url: [scheme: "https", host: "mysterious-meadow-6277.herokuapp.com", port: 443], force_ssl: [rewrite_on: [:x_forwarded_proto]],
  cache_static_manifest: "priv/static/cache_manifest.json",
  secret_key_base: Map.fetch!(System.get_env(), "SECRET_KEY_BASE")

config :hello, Hello.Repo,
  url: System.get_env("DATABASE_URL"),
  pool_size: String.to_integer(System.get_env("POOL_SIZE") || "10"),
  ssl: true
```
**NOTE:** Make sure to comment out/remove the `import_config "prod.secret.exs"` line at the end of this file

Decrease WebSocket Transport Timeout in `endpoint.ex`
```elixir
websocket: [timeout: 45_000],
```

Create a `Procfile` and add this line
```
web: MIX_ENV=prod elixir --sname server -S mix phx.server
```

Connect to the correct database by creating `prod.exs` and `dev.exs` files in `rsvp/config`

`prod.exs` should look like this:
```elixir
use Mix.Config

config :rsvp, Rsvp.Repo,
  url: System.get_env("DATABASE_URL"),
  pool_size: String.to_integer(System.get_env("POOL_SIZE") || "10"),
  ssl: true
```

`dev.exs` should look like this
```elixir
use Mix.Config
```

`config.exs` should have this line at the bottom
```elixir
import_config "#{Mix.env}.exs"
```

#### Creating Your Production Database
Run this command to create the database
```
heroku addons:create heroku-postgresql:hobby-dev
```

Set it's Pool Size so you don't overdo the resources
```
heroku config:set POOL_SIZE=18
```

**NOTE:** When running tasks, you'll want to limit pool sizes so you don't use all your resources
```
heroku run "POOL_SIZE=2 mix hello.task"
```

#### Fine, Keep Your Secrets
Generate a secret key for your application to use in Heroku
```
mix phx.gen.secret
```

Add this key to Heroku
```
heroku config:set SECRET_KEY_BASE="YOUR_SECRET_KEY"
```

#### Commit and Deploy!
I have this repo tracked in Github so whenever I push to master, it deploys

#### Migrate That Repo
After the app is built, run this command
```bash
heroku run "POOL_SIZE=2 mix ecto.migrate -r Rsvp.Repo" --app your-app-name
```


----


## 10: Recap & Where to Go

### 10.1: Recap & Where to Go
### 10.2: Learning Resources
### 10.3: Extend The Application
