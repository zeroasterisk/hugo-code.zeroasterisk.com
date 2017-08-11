---

slug: "elixir-ftw-it-really-is"
title: "Playing with Elixir is just FUN"
date: 2017-07-10
author: "alan blount"
categories:
- code learning
tags:
- elixir
- ecto
- exploration
- packages
autoThumbnailImage: false
thumbnailImagePosition: "top"
description: "Elixir really is as nice as you've been hearing"

---
I've been using Elixir for a few weeks now, sometimes
[forcing it to play nice with Mongodb/Meteor](/2017/06/meteor-and-elixir-rocky-courtship/)
, which it doesn't really want to do.
Sometimes with Posgres _(which it loves)_.
Sometimes just plain ole' Elixir with no persistance.

It *is* realllllly fun!


<!--more-->

You know when everyone tells you to watch a movie?
Sometimes it sucks (and sometimes it sucks *because* everyone told you to watch it).
Sometimes it's ok and you can kind of understand why people told you to watch it.
Sometimes it *is totally awesome*.

# Elixir really _is_ as nice as you've been hearing

Why?

I assume you already know...

If not, go [search "elixir" on medium](https://medium.com/search?q=elixir) --- ummm... yeah.

Fast, stable, scalable, effecient...

Ok it's not perfect - erlang can be quite chatty and slow for some purposes...

But erlang has got TONs of real-world engineering behind it, and it scales super-well.

And then you add the Elixir layer and the choice becomes easy...

<!-- toc -->

I know, this is what Ruby programmers were saying back in the day.
In fact a lot of the community behind Elixir are Rubyists.
But back in the day, when Ruby was saying that, Ruby was slow as a dog.
It's since gotten a lot faster, but it's not even close to comparing with the scaling effeciency of Erlang.

# Functional programming and the glorious pipe

Also, I've been doing Object Oriented Programming for a long time
_(in PHP, Node, Python - so, ummm, it's been a fight)_.
A lot of programming is all about managing *state* and objects make a lot of sense...
Keep the state organized, know it's boundaries.

But I've gotta tell you, and functional programming is a joy.
There state is even further removed, it's not a part of your code/function at all ---
you remove all the state to outside the scope of *your code* and persist to database or in memory stores,
and then all of your code is pure passthrough transformations and logic.

And transformations are exactly what the pipe opperator was built for.

Here's a super quick example.

Take a list of strings, map each one of them into a list of upper case strings, then make the list unique/distinct, then sort it.

_(for now, ignore that `&` and the `/1` thing --- it makes a lot of sense but might look wonky for now)_

```ex
def my_func(list_of_strings) do
  list_of_strings |> Enum.map(&String.capitalize/1) |> Enum.uniq |> Enum.sort
end
```

The pipe opporator `|>` basically says,
_"staring with some thing, do something to it, and pass it along"_.
The next function after the `|>` recieves the output of the thing on the left, as it's first argument...
you can pass in second, third, etc.  Whatever you output, goes on to the next function.

That is most of what we want to do in programming, but we usually have to do it all backwards...

Here is the same as above, in Javascript:

```js
const my_func = (list_of_strings) => {
  return Array.from(new Set(list_of_strings.map(s => s.toUpperCase()))).sort
}
```

Not terrible, we can chain the sort, and could possibly have made this more elegant in several ways...
We could use one of the great functional libraries for Javascript, like [ramdajs](http://ramdajs.com),
but it's still difficult to pass along content to the next function in the chain...
more often, we do something, and wrap that in the next step, so our input/argument is the most deeply nested part of the function.

Here is the same as above, in Python:

```py
def my_func(list_of_strings):
  return sorted(list(set([s.upper() for s in list_of_strings])))
```

It's actually pretty nice - the array comprehension is bad-ass.
The functions are all distinct and fairly obvious.

But the progress, from one step to the next, is always backwards.

It seems like such a little thing, but once you get used to it - not having it is annoying.

# Pattern matching makes me smarter

I don't think I'm going to put in the animated gif of a mind-blowing person here...
but know that I thought about it.

```ex
def my_func(%{"material": "wood"} = instructions) do
  Map.merge(instructions, %{"tool" => "nails"})
end
def my_func(%{"material": "metal"} = instructions) do
  Map.merge(instructions, %{"tool" => "welder"})
end
def my_func(%{"material": "sever"} = instructions) do
  Map.merge(instructions, %{"tool" => "elixir"})
end
```

Because of pattern matching, almost all of your logic can be avoided.

Seriously, how much of your code looks like: _"if I got this input, do this, otherwise, if I got this input do that, otherwise if I got this other input, do something else"_.

Instead of all of that, you can define the same function, and look for different inputs, and do different things...
*Every function is a router.*

It can also allow you to easily destructure from the function definition line, and extract variables and compare values.:w

# Tooling for days - testing and docs - reliable compiler - strict but flexible

I am very impressed with the tooling for Elixir.
Developer experience is obviously very important for this community and it shows.

Testing is fast and easy, convenient, and common.

Documentation is primarily done via `@doc` blocks in code, and generated into HTML via `ex_doc`.

Tying both of those together, the doctesting is very impressive...

```
  @doc """
  Here is a list to upper, uniquer, sorter (I know you've always wanted it!)

  Examples

      iex> MyModule.my_func(["a", "A", "a", "b"])
      ["A", "B"])

      iex> MyModule.my_func([])
      [])

  """
```

Basically - you describe your function in the normal doc block
_(since you know it will be rendered to HTML and read, you usually care more)_
and then you provide examples of how to use the function and what you expect to get out...

But here's the amazing thing... the Examples show up in your documentation, nicely formatted,
*and* they also are executed as unit tests...

_(thanks python for this one - still the implementation is so much simpler/faster/cleaner in Elixir)_

I also love that the compiler tells me about most of my errors.

Credo is a great linter, which I didn't have to spend a month configuring,
and didn't have to decide
[if semi-colons were in this year or not](https://stackoverflow.com/a/17903018/194105).

Bascially - even for web tools - for most of the work I'm doing, I setup `test.watch`
and I never actually have to look at the browser/API until I'm pretty confident things are working under-the-hood.
I always get a warm glow when that works out, and I can usually get there with other languages,
but there were no obsticales with Elixir.

# Quality & Maintainability

As you can tell, I've drunk the kool-aid a bit.
I'm happy with Elixir.

I'm not as fast with it as I expected to be.
There is a lot to (re)learn, but more-to-the fact,
Elixir kindof forces you to be a better developer.
The strong discouragement of `if` conditionals, and to take advantage of pattern matching,
helps me to take fewer shortcuts -- no procedural sprawl, all small functions.
The great testing experience encourages TDD and more test coverage.
The great docs encourages you to be nice to future developers.

I have a feeling that the stuff I build in Elixir will be stable and solid for a while
and a lot easier to maintain.
