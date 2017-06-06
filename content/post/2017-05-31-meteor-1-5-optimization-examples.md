---

slug: "meteor-1-5-bundle-optimization"
title: "Meteor 1.5 ~ Bundle Optimization + Examples"
date: 2017-05-31
author: "alan blount"
categories:
- code learning
tags:
- meteor
- es7
- import
- optimization
- react
autoThumbnailImage: false
thumbnailImagePosition: "top"
description: "deep dive into Meteor 1.5 dynamic imports. When, how, and why."

---

I have been playing with [Meteor](https://meteor.com/) 1.5 and it's dynamic imports,
which can be used to reduce the amount of javascript code sent to the browser.

This article is a deep dive into how to profile and optimize your client side code
to make smarter use of the new dynamic imports...

Which will make **meteor load faster for your users**.

<!--more-->

> If you're not already familiar with Meteor you should checkout the [meteor website](https://meteor.com/) and the [meteor chef](https://themeteorchef.com/)
> Come back to this when you're comfortable and want to optimize an existing application.

<!-- toc -->

[Meteor 1.5 landed](https://blog.meteor.com/announcing-meteor-1-5-b82be66571bb),
and it's got a lot of improvements,
but I think the big new feature that people want to know about is
**the ability to dynamically import code**.

> What is that?<br/>
> Why is it useful?<br/>
> How do I do it?<br/>
> How do I know what to dynamically import?

Well, lets look into some of those questions, and how to use these new tools to improve your application's load time.

This post is **about identifying what to optimize and how to optimize**...
I will do another about how to develop/architect better dynamically imported components.

# Setup a Basic Meteor Project

> You can clone [this repo](https://github.com/zeroasterisk/meteor-example-optimize-client-bundle.git)
> and checkout the `01-initial` branch.
> Or you can use the following instructions to get up to this starting point.
> Or you can jump right into this topic on your own project.

To do anything, we need a simple test project.
I am starting with [Meteor Chef Base](https://github.com/themeteorchef/base)
and I am adding [react-dates](https://github.com/airbnb/react-dates)
and a bit of functionality to pick date ranges
_(I also happen to know it has a noticeable impact on bundle size, which we will see later)_.


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

> You can clone [this repo](https://github.com/zeroasterisk/meteor-example-optimize-client-bundle.git)
> and checkout the `02-before-optimization` branch.

Make things faster, always.
Initial render of the site/app is a *great* metric to improve on
and one of the pain points for meteor apps.

When in _production_ mode, Meteor builds all of your app into 1 big JS and 1 big CSS file - these are called the *"client bundle"*.

Meteor does it's best to minimize those files, and cache them... but it can still be a whole lot of stuff sent to the client (this is all of your code and your code's dependencies) on initial pageload...
Especially if you don't always need some of those components and dependencies.

To test and profile, **we have to run in production mode**.

`meteor --production`

Our super-tiny example app is _not too bad_, but could certainly get better.

Our biggest slowdown by far is the bundle of all javascript.

|            | size   | download_time |
|------------|--------|---------------|
| JS bundle  | 494 KB | 444 ms        |
| CSS bundle | 7 KB   | 29 ms         |

![time-to-render-prod-mode-before-optimize](https://puu.sh/w80ky/6ded16d16d.png)

Wow, but this is such a tiny application... what can we do?

# Plan: Profile to Determine Work to be Done

You should never optimize without profiling.

Sure, when you develop keep performance and optimization in mind - use best practices and all that.
But when you are trying to figure out how you can improve performance, you should use tools to profile and get information about what to optimize.
Without that profiling information, you will be treading water, spending lots of effort for little value.

## How to Profile Meteor JavaScript Bundle?

So we know we need to reduce the total JavaScript Bundle... but how?
What's Big?
What's easy to remove?
What's not used?

Luckily, there is a now a
[bundle-visualizer](https://atmospherejs.com/meteor/bundle-visualizer)
package which gives us an awesome
sundial graph for identifying size in our initial load.

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

## How can we optimize?

So we can look at our codebase in _sections_ and do something...

_What exactly will we even be doing?_

It really comes down to 3 options:

1. **remove** a section of code from our project entirely _(best solution, but not often possible)_
2. **reduce** the size of a section _(micro optimization, not usually effective)_
3. **delay** loading a section via **dynamic import** _(our new option)_

### Remove

In this example, I could **remove** `jquery` or `handlebars` because I'm not using it... but Meteor Chef Base is...

This is outside of the scope of this article.  If you can remove code, do so.

### Reduce

There's very little I could do to **reduce** the size of my code, my application code is *super tiny*.

This is usually the least valuable way to spend your time.  Better to remove or delay a bunch first.

### Delay

Ok... What can I **delay**, and what are the ramifications of doing so?

Basically you want to look for things which are only used once or twice,
but which have a heavy footprint.

## Pick high value targets - sections to delay

Armed with a profile of code sections _(size)_,
we try to identify a what we can remove from our initial bundle, and only load as needed.

We start by looking at the "biggest" sections of code _(clockwise)_,
and for each section we try to ask ourselves if we can **remove** or **delay** loading.

**Pro Delay:**

* Is the section of code used only once or in a very few components?
* Is the section of code an optional "secondary" feature?
* Is the section of code a big enough percentage of the bundle to make it worthwhile?

**Against Delay:**

* Is the component required to make the initial page render look ok to the user?
* Is the page going to significantly *jump* when the component gets loaded dynamically, after the rest of the page has rendered?
* Is the lack of that component going to mess things up for server side rendering?

In this case, `react-dates` is a great candidate, because I **only use it once** in my codebase, but **it's got a huge footprint** in the clientside bundle.

> Note: I'm using [ag (the silver searcher)](https://github.com/ggreer/the_silver_searcher) or [rg (ripgrep)](https://github.com/BurntSushi/ripgrep) to search through file contents, lightning fast.

```sh
$ ag 'react-dates' imports -c
imports/ui/components/PickDates.js:1
```


# Optimize: delay loading code with a dynamic import

Since we decided to target `react-dates`,
I am going to dynamically load the only component which uses it, `PickDates`.

I could do this several ways, but here's a very simple approach.

First I install a super-useful helper
[react-loadable](https://github.com/thejameskyle/react-loadable),
which handles the _"maybe I'm loading, maybe I'm loaded"_ react component.

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

It can get a lot more complex, but this is a fine place to start.

1. `react-loader` is a [HOC](https://facebook.github.io/react/docs/higher-order-components.html)
which will render our component when it's available.
2. The `loader: <func>` function recieves a promise which resolves as soon as the dynamic import is done.
3. The `import(<path>)` is the where the actual dynamic import _(magic)_ happens.
It returns a promise which resolves when the component is on the client.
4. The `LoadingComponent` shows while the component is not yet loaded.
5. The `delay` hides the `LoadingComponent` for this long, so there is less flicker in case the component is already cached and can just load-in super fast.

Because the `import` is no longer part of the initial clientside bundle,
it is compiled with all of it's dependencies, separately.
They will be loaded later, the first time they are used, and then cached on the client. _(more on this later)_

# Review: After our dynamic import

> You can clone [this repo](https://github.com/zeroasterisk/meteor-example-optimize-client-bundle.git)
> and checkout the `04-after-optimization` branch.

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

But if I get rid of the `bundle-visualizer` then I can see our date picker working...

```sh
meteor remove bundle-visualizer
```

![after-dynamic-import-still-working](https://media.giphy.com/media/3o7buf4zcCkBnlL0jK/giphy.gif)

## Gotcha: You have to dynamic import ALL references

You could have `MyComponent` and `OtherComponent`, both of which included `react-dates`.

The `react-dates` section of code would only be removed from the initial clientside bundle
if both `MyComponent` and `OtherComponent` were dynamically imported.

**Any static import, ensures the component and all descendants are in the clientside bundle.**

# Understand: So how does it work?

Where is did the component go?  How does it work?

If you use the _Developer Tools > Network tab > WS (websocket)_ and look at the frames,
you can find a request for the dynamic import,
and the response with the dynamic import payload.

![find-dynamic-imports-in-frames](https://media.giphy.com/media/3ohzdZbi1jPbNlx9Ty/giphy.gif)

![request](https://puu.sh/w86fn/6d86a850a7.png)
![response](https://puu.sh/w86gK/3819c7813f.png)

That's actually kind of a **big** frame!

But it _is cached_, websocket transfer is pretty fast, and it only is loaded when it's needed.

Furthermore, each component is **only imported 1 time**, even if imported via different paths.

## Keep this in mind: Dynamic Imports are not Free

The dynamic import does make the section of code disappear from your clientside bundle.

It does "improve" the profiled metrics... but it has a few costs:

* starting meteor is a wee bit slower _(sorry devs)_
* added complexity
* bundle over websocket can not be offloaded to a CDN
* dynamic imports are not _(currently)_ profileable... so to profile you'd have to make static imports again.

## Built In Optimization: Each Component IS Only Imported 1 Time

> You can clone [this repo](https://github.com/zeroasterisk/meteor-example-optimize-client-bundle.git)
> and checkout the `06-experiments` branch.

I conducted a few experiments (more below) and I now better understand how the imports work.

Here is an obvious structure of components:

```
- MyPage
  - Trunk <-- dynamicly imported from a page
    - Limb
      - Branch <-- dynamicly imported from a page
        - Twig
          - Leaf
```

At first, my experiment was to log into the console, when JS "loaded" code.

![ss](https://puu.sh/wbcVJ/ccfac76efc.png)

That caused me to believe they were in fact imported twice... but **they were not**.

In fact, if you look at the websocket frames,
you can see that `Leaf` component is only imported 1 time.
![ss](https://puu.sh/wbdrg/1e9d059d96.png)
![ss](https://puu.sh/wbdrP/9106658bc2.png)

And it is more obvious, when you load the Trunk first and the Branch second:
![ss](https://puu.sh/wbe2H/3caac65457.png)

Then there is not second import for Branch, because it was already imported above...

**This is REALLY impressive, and speaks highly of the module system in Meteor**

# Experiments: What can you do with dynamic imports?

> You can clone [this repo](https://github.com/zeroasterisk/meteor-example-optimize-client-bundle.git)
> and checkout the `06-experiments` branch.

I felt like trying out a few more variations on how things can be loaded.

**This is a Work In Progress, more coming soon**

- [x] import as in-file function, and in-file loader _(works)_
- [x] import as shared function, but in-file loader _(works)_
- [ ] import as shared function, path as arg _(fails, can not find the path)_
- [ ] import a npm package directly _(fails)_
- [x] import a component that imports a component _(works, and a good idea, within reason)_

# Experiments: Profiling imports?

> You can clone [this repo](https://github.com/zeroasterisk/meteor-example-optimize-client-bundle.git)
> and checkout the `07-profiling` branch.

- [x] import (static) `lodash.cloneDeep` = `8.9kb` [screenshot](https://puu.sh/wc2Gs/f694dacc7b.png)
- [x] import (static) `lodash/cloneDeep` = `19.5kb` [screenshot](https://puu.sh/wc2Lq/88f9db3c45.png)
 + [screenshot2](https://puu.sh/wc2KD/c333e1834e.png)


# Summary

You can now segment your meteor clientside codebase.

* You can profile and delay expensive components + dependencies
* You can make an admin section _(or whatever section)_ of your site only load when needed.
* You can make every single route, only load when needed _(sub-optimal, but effective, temporarily)_.
* Or better, you can dynamically load individual components... _(especially expensive ones)_

# Thanks & Credits

Huge thanks to Meteor, MDG, and especially [ben](https://github.com/benjamn/)... I've been learning a lot just by reading your commits, and I am grateful for your contributions.

Also a huge thanks to [jesse](https://github.com/abernix) for the bundle-visualizer and all your other meteor work.

Definitely read [the Meteor Blog Post](https://blog.meteor.com/announcing-meteor-1-5-b82be66571bb) and maybe the [huge pull request](https://github.com/meteor/meteor/pull/8327) for information about dynamic imports.

I've also played with code from meteor's [react-loadable example](https://github.com/meteor/react-loadable-example) which helped me to understand dynamic imports.

Other good reading, [webpack's dynamic import](https://webpack.js.org/guides/code-splitting-async/) docs are useful too.


