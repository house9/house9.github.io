---
layout: post
title: "ember-cli broccoli fingerprinting"
date: 2015-02-15 09:07
comments: true
categories: emberjs
---

ember-cli uses broccoli for asset compilation and [fingerprinting](http://www.ember-cli.com/asset-compilation/#fingerprinting-and-cdn-urls) of asset file names.

> When the environment is production, the addon will automatically fingerprint your js, css, png, jpg, and gif assets ...

No fingerprinting occurs in the development environment. These are great defaults, but what if you have a staging environment and want to fingerprint those assets? You can override the fingerprint setting when the ember application is created.

```javascript
var app = new EmberApp({
  fingerprint: {
    enabled: true
  }
});
```

Now all environments will fingerprint assets, but we really don't want that in development. Since the ember application has not booted up yet we do not have access to the configuration object directly - lucky for us it is available on the `process` object before the ember application is created.

```javascript
// Brocfile.js
var emberEnvironment = process.env.EMBER_ENV;
var fingerprint = (emberEnvironment === 'production' || emberEnvironment === 'staging');
var app = new EmberApp({
  fingerprint: {
    enabled: fingerprint
  }
});

```