---
title: Meteor 0.9.2-rc1 is fun to play with, Cordova support functional
author: alan
type: post
date: 2014-09-09T21:41:00+00:00
url: /2014/09/09/meteor-092rc1-with-cordova/
categories:
  - cordova / phonegap
  - geek
  - meteorjs

---
Last night I started playing with \`meteor &#8211;release 0.9.2-rc1\` which you can read about and play with too; check it out here:

<https://meteor.hackpad.com/Getting-Started-With-Cordova-Z5n6zkVB1xq>

**It&#8217;s wonderful!**

The local development and hot-code-push is a great improvement over any other solution I&#8217;ve been using. Builds were painless (mostly).

Most of my stumbling points were due to 9x, not the new Cordova stuff.

A few interesting details&#8230;

**Cordova gets built here:**

<pre>$ ll -a .meteor/local/cordova-build
drwxr-xr-x+  8 admin  wheel   272B Sep  9 16:37 .
drwxr-xr-x+  6 admin  wheel   204B Sep  9 17:14 ..
-rw-r--r--+  1 admin  wheel   488B Sep  9 10:14 config.xml
drwxr-xr-x+  3 admin  wheel   102B Sep  9 10:14 hooks
drwxr-xr-x+  2 admin  wheel    68B Sep  9 10:14 local-plugins
drwxr-xr-x+  4 admin  wheel   136B Sep  9 16:37 platforms
drwxr-xr-x+ 11 admin  wheel   374B Sep  9 16:37 plugins
drwxr-xr-x+ 13 admin  wheel   442B Sep  9 16:37 www

$ ll -a .meteor/local/cordova-build/platforms
drwxr-xr-x+  4 admin  wheel   136B Sep  9 16:37 .
drwxr-xr-x+  8 admin  wheel   272B Sep  9 16:37 ..
drwxr-xr-x+ 18 admin  wheel   612B Sep  9 16:38 android
drwxr-xr-x+ 10 admin  wheel   340B Sep  9 16:38 ios
</pre>

So far, nothing easy in regards to configuring Cordova, but I&#8217;d guess we can edit the config.xml files&#8230; I&#8217;d love to see an app icon builder package sometime&#8230;

  * point it at a source and it renders all sizes
  * or it allows you to specify versions for every size
  * it would recreate the XML config needed for all icons and rename them as needed

**Plugins FTW**

Installing Cordova plugins probably \*should\* be done via Meteor Packages, because it&#8217;s an awesome approach.

Here is my first attempt:

<https://github.com/zeroasterisk/meteor-cordova-geolocation-background>

Currently I&#8217;m working on integration and configuration code (porting from a non-package code over the next few days), but the Cordova build system seems to have correctly found and installed the right plugins for me:

<pre>$ ll -a .meteor/local/cordova-build/plugins
total 32
drwxr-xr-x+ 11 admin  wheel   374B Sep  9 16:37 .
drwxr-xr-x+  8 admin  wheel   272B Sep  9 16:37 ..
-rw-r--r--+  1 admin  wheel   4.7K Sep  9 16:37 android.json
drwxr-xr-x+  8 admin  wheel   272B Sep  9 16:37 com.romainstrock.cordova.background-geolocation
-rw-r--r--+  1 admin  wheel   4.6K Sep  9 16:38 ios.json
drwxr-xr-x+ 13 admin  wheel   442B Sep  9 16:37 org.apache.cordova.console
drwxr-xr-x+ 14 admin  wheel   476B Sep  9 16:37 org.apache.cordova.device
drwxr-xr-x+ 14 admin  wheel   476B Sep  9 16:37 org.apache.cordova.file
drwxr-xr-x+ 13 admin  wheel   442B Sep  9 16:37 org.apache.cordova.file-transfer
drwxr-xr-x+ 13 admin  wheel   442B Sep  9 16:37 org.apache.cordova.geolocation
drwxr-xr-x+ 13 admin  wheel   442B Sep  9 16:37 org.apache.cordova.statusbar
</pre>

I expect to see a lot of new Meteor Cordova plugins soon.

It&#8217;s a brave new world.

