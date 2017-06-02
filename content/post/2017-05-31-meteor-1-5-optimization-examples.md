+++
author = "alan blount"
categories = ["meteor", "optimization", "react"]
date = "2017-05-31T22:56:48-04:00"
description = "deep dive into Meteor 1.5 dynamic imports. When, how, and why."
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
title = "Meteor 1.5 ~ Bundle Optimization + Examples"

+++

I have been playing with Meteor 1.5 release candidates and I thought this would be a great topic for a quick article on how to optimize your clientside bundle... make meteor faster for your users.
<!--more-->

<!-- toc -->

> If you're not already familiar with Meteor you should checkout [the meteor website][meteor] and also [the meteor chef][meteorchef]

[Meteor 1.5 landed][meteor-1.5], and it's got a lot of fantastic features,
but I think the big one that people want to know about is
**the ability to dynamicaly import code**.

What is that?  Why is it useful?  How do I do it?  How do I know what to dynamicly import?

Well, this is a bit of a deep dive into some of those questions, and how to use these new tools to improve your application's load time.

This post is **about identifying what to optimize and how to optimize**...
I will do another about how to develop better dynamicly imported components.

# Setup a Basic Meteor Project

> You can clone [this repo][example-repo] and checkout the `01-initial` branch.
> Or you can use the following instructions to get up to this starting point.
> Or you can jump right into this topic on your own project.

To do anything, we need a simple test project.
I am starting with [Meteor Chef Base][meteorchefbase]
and I am adding [react-dates][react-dates] and a bit of functionality
to pick date ranges
_(I also happen to know it has a noticable impact on bundle size, which we will see later)_.


```sh
git clone https://github.com/themeteorchef/base.git meteor-example-optimize-client-bundle
cd meteor-example-optimize-client-bundle
rm -rf .git && git init && git add . && git add .meteor/*
meteor update --release 1.5
cat .meteor/release
  METEOR@1.5
meteor npm install --save react-dates moment react-addons-shallow-compare
npm start
```

I then edited the code and added an example to create a simple example project.

![screencast basic functionality](https://media.giphy.com/media/xUA7aRqXjSZBc4oHDy/giphy.gif)

Not terrible for a quick example _(and not a todo list)_.

# Goal: Reduce Time to Initial Render

> You can clone [this repo][example-repo] and checkout the `02-before-optimization` branch.

Make things faster, always.
Initial render of the site/app is a *great* metric to improve on
and one of the pain points for meteor apps.

Why?  When in production mode, Meteor builds all of your app into 1 big JS and 1 big CSS file - these are called the *"client bundle"*.

Meteor does it's best to minimize those files, and cache them... but it can still be a whole lot of stuff sent to the client (this is all of your code and your code's dependancies) on initial pageload...
Especially if you don't always need some of those components and dependancies.

To test and profile, *we have to run in production mode*,
as if we had deployed to a server.

`meteor --production`

Our super-tiny example app is _not too bad_, but could certainly get better.

Our biggest slowdown by far is the bundle of all javascript.

|            | size   | download_time |
|------------|--------|---------------|
| JS bundle  | 494 KB | 444 ms        |
| CSS bundle | 7 KB   | 2 ms          |

![time-to-render-prod-mode-before-optimize](https://puu.sh/w80ky/6ded16d16d.png)

Wow, but this is such a tiny application... what can we do?

# Profile Profile Profile....

You should never optimize without profiling.

Sure, when you develop keep performance and optimization in mind - use best practices and all that.
But when you are trying to figure out how you can improve performance, you should use tools to profile and get information about what to optimize.
Without that profiling information, you will be treading water, spending lots of effort for little value.

# How to Profile Meteor JavaScript Bundle?

So we know we need to reduce the total JavaScript Bundle... but how?
What's Big?
What's easy to remove?
What's not used?

Luckily, there is a now a
[bundle-visualizer][bundle-visualizer] package which gives us an awesome
sundail graph for identifying size in our initial load.

```sh
meteor add bundle-visualizer
npm run start-as-prod
# ^ if you don't have my package.json scripts, you would run
#   meteor --production
```

Then an overlay happens in your browser and you can hover over parts to see what they total size is for the part, and it's children...

![bundle-visualizer-before-optimize](https://media.giphy.com/media/l4FGFeTW6pqVvabGE/giphy.gif)

This is **AMAZING**
![aw_yeah](http://emojis.slackmojis.com/emojis/images/1450447981/152/aw_yeah.gif)

Real quick review of some numbers here:


| section           | percent |
|-------------------|---------|
| `react-dom`       | 9.4%    |
| `react-dates`     | 9.1%    |
| `react-bootstrap` | 8.4%    |
| `bootstrap`       | 6.4%    |
| `jquery`          | 4.6%    |
| `handlebars`      | 4.0%    |
| app code          | 1.7%    |

![app-1.7percent](https://puu.sh/w82Pi/11b30cc350.png)

My entire app code is `32k` which is only `1.7%` of the clientside bundle!

# Pick high value targets - sections to remove, reduce, or delay

Armed with this information, we try to identify a section of code we can reduce from our bundle.

We start by looking at the "biggest" sections of code (clockwise),
and for each section we try to ask ourselves if we can **remove** or **reduce** or **delay** loading.

1. **remove** from our project entirely (best solution, but not often possible)
2. **reduce** _significantly_ the size of a section (micro optimization, not usually effective)
3. **delay** loading a section via **dynamic import** (our new option)

In this example, I could **remove** `jquery` or `handlebars` becaue I'm not using it... but Meteor Chef Base is... This is outside of the scope of this article, but a good thing to plan in your application.

My application code is *super tiny* comparativly, so there's very little I could do to **reduce** the side of my code... This is usually the least valuable way to spend your time.  Better to remove or delay a bunch first.

Ok... What can I **delay**, and what are the ramifications of doing so?

Basically you want to look for things which are only used once or twice,
but which have a heavy footprint.

In this case, `react-dates` is a great candidate, because I **only use it once** in my codebase, but **it's got a huge footprint** in the clientside bundle.

Other things to consider:
- is the component required to make the initial page render look ok to the user?
- is the lack of that component going to mess things up for server side rendering?
- is the page going to significantly *jump* when the component gets loaded dynamically, after the rest of the page has rendered?

# Optimize with a dynamic import

Since we decided to target `react-dates` I am going to dynamicly load the only component which uses it.

I could do this several ways, but here's a fairly simple one.

First I install a super-useful helper [react-loadable][react-loadable], which handles the loading react component

```sh
meteor npm install --save react-loadable
```


**before: normal import** part of clientside bundle

```js
import PickDates from './PickDates';
```

**after: dynamic import** no longer a part of clientside bundle

```js
import Loader from 'react-loader';

// generic loading component to show while transfering section of code
const LoadingComponent = () => <span className="text-muted"><i className="fa fa-refresh" /></span>;
// new version of the component, now: loading --> rendered
const PickDates = Loader({
  // this does the dynamic import, and returns a promise
  loader: () => import('./PickDates'),
  // this is our generic loading display (optional)
  LoadingComponent,
  // this is a delay before we decide to show our LoadingComponent (optional)
  delay: 200,
});
```

This is a bit of a simple answer, it can get a lot more complex... but in short:
`react-loader` will render our component when it's available.
The `loader` function recieves a promise which resolves as soon as the dynamic import is done.
The `import(<path>)` is the where the magic happens.

Because the `import` is no longer part of the initial clientside bundle,
it is compiled with all of it's dependancies, seperately.
They will be loaded later, the first time they are used, and then cached on the client. _(more on this later)_

# After our dynamic import

> You can clone [this repo][example-repo] and checkout the `04-after-optimization` branch.

Did it do anything?  Lets profile again.

```sh
npm run start-as-prod
# ^ if you don't have my package.json scripts, you would run
#   meteor --production
```

| JS bundle             | size   |
|-----------------------|--------|
| before dynamic import | 494 KB |
| after dynamic import  | 459 KB |

![after-dynamic-import-network-tab](https://puu.sh/w85w8/40aaf947e5.png)

And how about our sundial?

![after-dynamic-import-bundle](https://media.giphy.com/media/3og0IwwJqrZwK8JlMQ/giphy.gif)

It looks like we no longer have the `react-dates` module at all...

But if I get rid of the `bundle-visualizer` then I can see it working...

```sh
meteor remove bundle-visualizer
```

![after-dynamic-import-still-working](https://media.giphy.com/media/3o7buf4zcCkBnlL0jK/giphy.gif)

# So how does it work?

Where is did the component go?  How does it work?

If you use the _Developer Tools > Network tab > WS (websocket)_ and look at the frames,
you can find a request for the dynamic import,
and the response with the dynamic import payload.

![find-dynamic-imports-in-frames](https://media.giphy.com/media/3ohzdZbi1jPbNlx9Ty/giphy.gif)

![request](https://puu.sh/w86fn/6d86a850a7.png)
![response](https://puu.sh/w86gK/3819c7813f.png)

That's actually kind of big!

But it is cached, and it only is loaded when it's needed.

# Experiments - what can you do?

WIP

- [x] import as in-file functions
- [ ] import as shared functions
- [x] import as shared function, but in-file loader
- [ ] import a npm package
- [x] import a component that imports a component

# Summary


[meteor]: https://meteor.com/ Meteor JS Framework
[meteor-1.5]: https://blog.meteor.com/announcing-meteor-1-5-b82be66571bb Meteor v 1.5
[meteorchef]: https://themeteorchef.com/ The Meteor Chef
[example-repo]: https://github.com/zeroasterisk/meteor-example-optimize-client-bundle.git
[react-dates]: https://github.com/airbnb/react-dates
[react-loadable]: https://github.com/thejameskyle/react-loadable
