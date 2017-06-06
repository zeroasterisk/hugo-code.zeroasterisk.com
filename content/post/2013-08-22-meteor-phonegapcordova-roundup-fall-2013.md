---
title: Meteor + PhoneGap/Cordova (Roundup – Fall 2013)
author: alan
type: post
date: 2013-08-23T02:49:23+00:00
url: /2013/08/22/meteor-phonegapcordova-roundup-fall-2013/
categories:
  - geek
  - js
  - meteorjs

---
UPDATE: see [meteor-core-cordova-support-alpha-release-is-out (2014-08-25)][1] for new information

* * *

&nbsp;

# About Meteor and PhoneGap / Cordova

**Meteor** is a rapid server/client development framework, written in JS &#8211; makes very interactive apps easy to build. <http://meteor.com>

**PhoneGap / Cordova** is a project to allow developers to use HTML, CSS, and JS to build mobile apps for all major devices. <http://phonegap.com>

### Basics of Integrating Meteor + PhoneGap / Cordova

The basic idea here is to use PhoneGap / Cordova to create a mobile App on several different platforms at once and it gives device hardware access; then “load” your Meteor application, which handles all of the rest of whatever you want your app to do.

### Why this approach?

**Meteor** already has a lot of what you’d want for any App, including client/server shared logic, client optimization and caching, and an amazing client/server data sync/transfer system.  It’s fast, easy to work in, and well documented.  If you ignore the mobile aspect, this is a wonderful platform &#8212; if we can get it to work with mobile devices, then it’s a no-brainer win.

**PhoneGap / Cordova** is a convenient “shell” for this, allowing you to get the device hardware hooks in/out of JS, and load up whatever HTML, CSS, JS you like.  Web developers love it, and even people who can work with native Android or iOS (etc) apps may like it because it’s cross platform, business logic only has to be written once.  (Also, it’s now a lot more organized and stable after the 3x release)

### Meteor + PhoneGap = Win!

The benefits of both are easy to combine. It’s your web app, and now it’s also your mobile app.

Just in case you didn’t hear that, **your full application is available on the web and on the device exactly the same, you only have to build it once**.  You simply have to slap on some responsive CSS and have logic in your app that determines if you’re in PhoneGap or not (more on that later).

More: <http://prezi.com/ig9gjm11mwsi/from-zero-to-mobile-web-app-in-sixty-minutes/>

## Integration Options

### CordovaPhonegap (Lazy Loading)

This approach starts the PhoneGap / Cordova out at the URL to the Meteor App.  This means NOTHING on the PhoneGap www folder gets loaded, and everything is loaded via Meteor.  After the page has loaded, it attempts to load the correct PhoneGap / Cordova JS file (again, from Meteor, out of the public folder) and then run device specific logic.

<https://github.com/awatson1978/cordova-phonegap>

#### Pros

  * Fast &#8211; fast to initially load, and internal changes are also fast
  * No middle layer in PhoneGap, no exchange
  * Simple way to get Meteor app into App Store and monetized
  * Apps built with this method **have** gotten approved in both iTunes and Google Play

#### Cons

  * Complicated because there are lots of different Cordova JS files you may have to support, for different browsers and different versions.
  * PhoneGap plugins JS more complicated (again, has to be managed inside Meteor, but versions kept in sync with PhoneGap)

### MeteorRider (DOM Hijacking)

This approach is somewhere between the other 2.  The PhoneGap / Cordova files live on the device in the PhoneGap www folder.  The App starts up with the local index.html file, but then JS “hijacks” the DOM, it does an AJAX request to your Meteor app and rewrites the document with your Meteor document, triggering all the events needed for Meteor to startup.

This gives Meteor access to all the PhoneGap JS (already loaded) but the PhoneGap JS is bundled for each device/version within PhoneGap.  It also puts all of the PhoneGap JS into the global scope alongside Meteor JS.

<https://github.com/zeroasterisk/MeteorRider>

#### Pros

  * Simple to manage, because all the device-specific code is within PhoneGap (each version is bundled)
  * Pretty fast, the base level AJAX request is pretty fast
  * Scope is easy, because PhoneGap is available to Meteor directly
  * App JS can be tested independent of Meteor (just don’t load it)

#### Cons

  * Scope collision is possible, be careful
  * Global scope collisions are very possible, you have to be careful with your plugins and variable names.

### Meteor-Cordova (Push / iFrames)

This approach is perhaps the most isolated.  It uses JS to make an iframe inside of the PhoneGap / Cordova “page” and to create a “queue” to exchange JS from Meteor to PhoneGap and back.  Meteor and PhoneGap can only talk to each other through this exchange, not directly.

<https://github.com/raix/Meteor-Cordova>

#### Pros

  * Simple to manage, because all the device-specific code is within PhoneGap (each version is bundled)
  * Sort-of fast, the base level iframe is pretty fast, but the in-iframe scrolling can lag
  * Scope is isolated, no collisions
  * App JS can be tested independent of Meteor (just don’t load it)
  * Plugins are basically guaranteed to work, because they are 100% isolated
  * Sophisticated use of Appcache

#### Cons

  * Scope is more complicated and slow, you HAVE to learn and use the exchange methods
  * it’s an iFrame&#8230; _(ick)_
  * because it’s an iFrame, scrolling can lag/flicker on some devices

### Packmeteor _(update)_

This builds all of the Meteor JS/CSS assets, along with the Cordova HTML/JS/CSS assets into a single bundle for deployment.

<https://github.com/SpaceCapsule/packmeteor>

#### Pros

  * Fastest load time
  * All assets always correct versions, in sync

#### Cons

  * You have to have your users download a new version of your app to get updates
  * Versions can get out of sync

### “Loader” _(update)_

This is a newer approach which builds all Cordova JS files into an asset hosted-on/loaded-from Meteor&#8217;s public folder.

<https://github.com/andrewreedy/cordova-loader>

#### Pros

  * Fast load time
  * Sophisticated use of Appcache
  * You no not have download a new version of your app to get updates
  * Versions can no longer get out of sync

#### Cons

  * ???

### “Core Support” _(update)_

The Meteor Core team is working on Cordova Support.

See their [Cordova Support Brainstorming][2] HackPad

and the [Dev Branch on GitHub][3].

**Update**:

Watch the [DevShop Presentation of Cordova ALPHA release][4]

and [here are instructions][5] to get it going and play with it.

#### Pros

  * Fast load time
  * Sophisticated use of Appcache/localStorage
  * You no not have download a new version of your app to get updates
  * Versions can no longer get out of sync

#### Cons

  * ???

* * *

# Smoothing the Rough Edges

For any of these options, you need to know:

  * from Meteor
      * Am I loaded into a device via PhoneGap?
      * Is the device loaded?  (deviceread, before Meteor is loaded, or after)
      * What hardware hooks are available?
  * from PhoneGap
      * Is Meteor connected (status)?
      * Do we need a “fallback” URL, if Meteor is not online

Some of that is built into all of the options, but there is a lot of room left for “making is easy”.

There is also some need for simplifying and standardizing some of the JS needed on the PhoneGap side (for the “hijack” and “iframe” methods) to load the Meteor app and failover if it’s not available.

Feel free to contribute if you accomplish any of these, or if you want to help out with the documentation (github fork & pull request).

# Conclusions

These are all new approaches and libraries.  There will be more support from the Meteor core for Mobile sometime, but that may be a while.  Until then, these are great options for making an excellent mobile app with Meteor and PhoneGap.

**Alan’s favorite = “iframes” because the isolation is good for PhoneGap plugins**

I personally HATE iframes with a passion, but after having used (and contributed to) all three options, I’m personally using the “iframe” method now.  This is because I have had trouble with PhoneGap plugins with the other methods.

**no plugins?  = “hijack” because it’s fast and simple**

If you don’t plan on using plugins, but may use PhoneGap core API then I recommend the “hijack” method.  It’s fast, simple, and you don’t have the awkward iframe exchange in the middle.

Finally, if you don’t use anything from PhoneGap at all, and it’s just a shell/browser for your Meteor App, you may prefer “lazy loading” &#8211; it is dead simple.  Be aware though, the long list of Cordova versions your app will eventually have to support makes this option my least favorite long term.

# Resources

**on Meteor**

  * <http://meteor.com>

  * http://docs.meteor.com/

  * Resource Roundup &#8211; http://goo.gl/cVlbB

  * Videos &#8211; http://www.eventedmind.com/

  * Book &#8211; http://www.discovermeteor.com/

  * Excellent overview/case study &#8211; <http://www.ibm.com/developerworks/web/library/wa-meteor-webapps/index.html>

**General PhoneGap Guides**

  * [PhoneGap Cordova Android Platform Guide][6]

  * [PhoneGap Cordova iOS Platform Guide][7]

  * <http://msdn.microsoft.com/en-us/magazine/hh975345.aspx>

**on Meteor + PhoneGap**

  * <http://prezi.com/ig9gjm11mwsi/from-zero-to-mobile-web-app-in-sixty-minutes/>

  * [http://stackoverflow.com/questions/10322723/can-meteor-be-used-with-phon][8][egap][9]

  * [http://blog.snowflax.com/meteor-on-mobile-device-using-phone][9][gap/][10]

  * Example of Lazy Loading: <https://github.com/zeroasterisk/Presenteract> & https://github.com/zeroasterisk/Presenteract-PhoneGap-ios

**Meteor Packages:**

  * <https://github.com/awatson1978/cordova-phonegap> (lazy loading)
  * <https://github.com/raix/Meteor-Cordova> (iframes)
  * <https://github.com/zeroasterisk/MeteorRider> (hijacking dom)


 [1]: https://zeroasterisk.com/2014/08/25/meteor-core-cordova-support-alpha-release-is-out/
 [2]: https://meteor.hackpad.com/Cordova-Support-Brainstorming-hL7s1ggxw4Q
 [3]: https://github.com/meteor/meteor/tree/cordova-hcp
 [4]: https://www.youtube.com/watch?v=zzNoXbv1DX4&feature=youtu.be&t=24m
 [5]: https://meteor.hackpad.com/Getting-Started-With-Cordova-Z5n6zkVB1xq
 [6]: http://docs.phonegap.com/en/edge/guide_platforms_android_index.md.html#Android%20Platform%20Guide
 [7]: http://docs.phonegap.com/en/edge/guide_platforms_ios_index.md.html#iOS%20Platform%20Guide
 [8]: http://stackoverflow.com/questions/10322723/can-meteor-be-used-with-phonegap
 [9]: http://blog.snowflax.com/meteor-on-mobile-device-using-phonegap/
 [10]: http://msdn.microsoft.com/en-us/magazine/hh975345.aspx
