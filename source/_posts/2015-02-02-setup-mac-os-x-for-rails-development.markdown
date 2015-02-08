---
layout: post
title: "Setup Mac OS X for rails development"
date: 2015-02-02 09:33
comments: true
categories: rails
---

Whenever I get new machine or re-install my OS I like to use pivotals [sprout-wrap](https://github.com/pivotal-sprout/sprout-wrap) for the setup. I have been through the process many times and finally documented it back in October.

From the app store install XCode

* open XCode and accept the terms, then close app
* from the terminal run: `xcode-select --install`
* copy ssh keys from old machine via airdrop
* then install sprout-wrap dependencies using the built in version of Ruby that ships with Mac OS X:

```
sudo gem install soloist
sudo gem install bundler
mkdir installs
cd installs
git clone https://github.com/pivotal-sprout/sprout-wrap.git
cd sprout-wrap
sudo bundle
```

Modify the `soloistrc` file if desired; for me that means adding 'sourcetree', 'hipchat' and a few others to the casks section.

The full list of available casks are here: [https://github.com/pivotal-sprout/sprout-osx-apps/tree/master/recipes](https://github.com/pivotal-sprout/sprout-osx-apps/tree/master/recipes)

You probably want to change the 'Energy Saver' settings to never while the installs are running.

`bundle exec soloist`

Enter your password after prompted and then watch the magic happen...

A few hours later, after sprout-wrap is done; time for a few more 'manual' installs 

* [janus](https://github.com/carlhuda/janus) for vim configuration (i dont' use vim much but occasionally pair with people who do)
* Clone personal dotfiles
* install some firefox add ons
* For chrome once I login to Google all of my add ons will auto install
* Install [SublimeText3](http://www.sublimetext.com/3) and [Package Control](https://packagecontrol.io/installation)
  * If you prefer SublimeText2 it is available as a cask and can be installed via soloist
  
> then Encrypt your hard drive using FileVault2

Done! :)

----

*NOTE: if you install sprout-wrap with defaults it will configure many os-x settings which you may or may not like; I personally do like them, especially the 'Fast Key Repeat Rate'.*

This post originally appeared on the private discussion board [Ruby Rogues Parley](http://parley.rubyrogues.com/). 