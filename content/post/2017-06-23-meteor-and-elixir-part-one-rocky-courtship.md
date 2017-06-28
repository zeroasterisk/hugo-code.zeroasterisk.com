---

slug: "meteor-and-elixir-rocky-courtship"
title: "Meteor and Elixir (part one) A Rocky Courtship"
date: 2017-06-23
author: "alan blount"
categories:
- code learning
tags:
- meteor
- mongo
- elixir
- ecto
- exploration
- packages
autoThumbnailImage: false
thumbnailImagePosition: "top"
description: "A story of getting Elixir to play nice with Mongo, for a MeteorJS application"

---

I have been watching Elixir for almost 2 years now, and debating if it's mature enough for me to invest in.

Obviously Erlang is super-mature, and Elixir is quite stable, but it's a question of "is the community mature enough yet?"

Since I'm now writing this, I believe it is "ready", and I'm beginning a journey to build out some services in Elixir
to stand up next to existing node services (meteorjs) and slow move funcitonality over to to Elixir.

Chapter One: **Proof of Concept, Mongo is supposed to be Simple.**

<!--more-->

> If you're not already familiar with any of these tools:
> - [Elixir you should checkout the [meteor website](https://meteor.com/)
> If you're not already familiar with Meteor you should checkout the [meteor website](https://meteor.com/)
> Come back to this when you're comfortable and want to optimize an existing application.

<!-- toc -->

## Goals

Build an elixir+phoenix application which will connect to an existing MongoDB
for an existing MeteorJS/Node application, and expose functionality via a GraphQL API.

- database and schema un-changeable, must match what is already there, in Mongo
- authentication must piggyback/use [Meteor Accounts](https://guide.meteor.com/accounts.html)
- permissions must re-implement functionality of [Meteor Roles](https://github.com/alanning/meteor-roles)
- framework = elixir+phoenix _(more on why, below)_
- graphql API to be handled via [absinthe](http://absinthe-graphql.org/)

### Why?

GraphQL is badass.  It will replace REST, like REST replaced SOAP.

I *could* make a GraphQL server for an existing Meteor application inside Meteor,
or via a node API, or any other server side framework...

But I've been watching Elixir for a while and I'm impressed by it's performace, HA, and syntax.
Functional &amp; Parallel.
*([Dave Thomas Approves](https://www.youtube.com/watch?v=a-off4Vznjs&feature=youtu.be))*

## Getting Started

### install elixir

Nothing hard or tricky here...

`brew install elixir` _(for OSX, others for you)_ and `mix `

### install phoenix

I chose to install phoenix `1.3.0-rc2` because I read up on it and
I think it's got some good changes (in app structure) and likely to land sometime soon-ish.

`mix archive.install https://github.com/phoenixframework/archives/raw/master/phoenix_new.ez`

### build a new phoenix app

Here's my command:

`mix phx.new --no-html --no-brunch --app e2mapi elixir-meteor-graphql`

- `--no-html` I don't need serverside rendering
- `--no-brunch` I don't need elixir to build JS packages
- `--app e2mapi` I have to give it a "friendly" app name
- `elixir-meteor-graphql` My project folder

I can then cd into the dir, (make it a git repo & commit,) then edit the `mix.exs` file to change deps.

### add mongodb_ecto (to connect to Mongo, but with Ecto awesomeness)

This should be as simple as adding in a mix package...
But I ran into several problems with versions and mis-matching.

I tried reverting to Phoenix 1.2, I tried various branches and tags of various packages.

After coming up with a stable seeming combination
[my PR was not accepted](https://github.com/michalmuskala/mongodb_ecto/pull/128)
and we are told to instead force the core `ecto` to `2.0`.

There is no readme telling us this, and I think it's a temporary requirement,
but it is currently a requirement.

So - here is my `mix.exs` for mongo + ecto:

```ex
     {:mongodb_ecto, github: "michalmuskala/mongodb_ecto", branch: "ecto-2"}, # <-- using mongo
     {:ecto, "~> 2.0", override: true}, # <-- have to FORCE ecto version :(
```

_(likely to change in the future, hopefully towards basic versions)_

### add absinthe (for GraphQL)

Again this should be simple, but I ran into issues getting the adaptors to play nice,
specifically with poison &amp; absinthe_ecto.

Here's the combination which worked for me, now.

```ex
     # Poision 3.+ flags older phoenix ecto -- no breaking changes override:true
     {:poison, "~> 3.1", override: true},
     {:absinthe, "~> 1.3"},
     {:absinthe_ecto, git: "https://github.com/absinthe-graphql/absinthe_ecto.git"},
     {:absinthe_plug, github: "absinthe-graphql/absinthe_plug", branch: "master"}
```

_(likely to change in the future, hopefully towards basic versions)_

### edit config to get to the database

Ecto should now be able to *talk mongo* but it does not know who to talk to yet.

Edit `config/dev.exs` and change the config at the bottom to be something like:

```ex
# Configure your database
config :e2mapi, E2mapi.Repo,
  adapter: Mongo.Ecto,
  database: "e2mapi_dev",
  hostname: "localhost"
  port: 27017
```

### get deps & sanity check

At this point, you should be able to get all deps without errors

`$ mix deps.get`

And you should be able to fire up `iex` and load your new (empty) phoenix server

`$ iex -S mix phx.server`

If it worked right you should see something like:

```
[info] Running E2mapi.Web.Endpoint with Cowboy using http://0.0.0.0:4000
Interactive Elixir (1.4.4) - press Ctrl+C to exit (type h() ENTER for help)
```

_(Can't connect to mongo?  Is mongo running locally?  Is the configuration correct?)_

### extra work for testing

Embrace testing.  If you have not already done so, **now is your chance**.

Elixir makes it super easy to write tests.
They are less fragile &amp; more powerful _(thanks to elixir syntax)_.
And doctests support is crazy good too.

Sadly, here too, *there are extra hoops to jump through* in order to get Ecto + Mongo playing nicely.

First, the easy one, we have a test env to config, just like dev.

Edit `config/test.exs` and change the config at the bottom to be something like:

```ex
# Configure your database
config :e2mapi, E2mapi.Repo,
  adapter: Mongo.Ecto,
  database: "e2mapi_test",
  hostname: "localhost"
  port: 27017
```

After doing so, I started getting errors like the following:

```
$ mix test
Compiling 12 files (.ex)
Generated e2mapi app
** (RuntimeError) cannot configure sandbox with pool nil.
To use the SQL Sandbox, configure your repository pool as:

      pool: Ecto.Adapters.SQL.Sandbox

    (ecto) lib/ecto/adapters/sql/sandbox.ex:429: Ecto.Adapters.SQL.Sandbox.mode/2
    (elixir) lib/code.ex:370: Code.require_file/2
    (elixir) lib/enum.ex:645: Enum."-each/2-lists^foreach/1-0-"/2
    (elixir) lib/enum.ex:645: Enum.each/2
    (mix) lib/mix/tasks/test.ex:229: Mix.Tasks.Test.run/1
```

After doing some searching, I found "SQL Sandbox" code in several test files.

```
$ ag Sandbox
test/support/channel_case.ex
30:    :ok = Ecto.Adapters.SQL.Sandbox.checkout(E2mapi.Repo)
32:      Ecto.Adapters.SQL.Sandbox.mode(E2mapi.Repo, {:shared, self()})

test/support/conn_case.ex
31:    :ok = Ecto.Adapters.SQL.Sandbox.checkout(E2mapi.Repo)
33:      Ecto.Adapters.SQL.Sandbox.mode(E2mapi.Repo, {:shared, self()})

test/support/data_case.ex
29:    :ok = Ecto.Adapters.SQL.Sandbox.checkout(E2mapi.Repo)
32:      Ecto.Adapters.SQL.Sandbox.mode(E2mapi.Repo, {:shared, self()})

test/test_helper.exs
3:Ecto.Adapters.SQL.Sandbox.mode(E2mapi.Repo, :manual)
```

I commented out the relevant sections and now `mix test` shows me green lights and my world is happy.

```
$ mix test
Compiling 3 files (.ex)
...

Finished in 0.03 seconds
3 tests, 0 failures

Randomized with seed 649726
```

## Build Our Application

TODO... this is a work in progress...  I will come back with a part 2, soon.
