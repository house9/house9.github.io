---
layout: post
title: "rbenv and Your user account isn't allowed to install to the system Rubygems"
date: 2015-02-03 09:54
comments: true
categories: ruby
---

This happened to me after re-installing my OS and using a fresh install of [rbenv](https://github.com/sstephenson/rbenv). I cloned my application repository and went to execute a `bundle install` and received the following error:

```
Fetching source index from http://rubygems.org/


Your user account isn't allowed to install to the system Rubygems.
You can cancel this installation and run:

    bundle install --path vendor/bundle

to install the gems into ./vendor/bundle/, or you can enter your password
and install the bundled gems to Rubygems using sudo.

Password:
```

What? rbenv should not be prompting me to use sudo, it was trying to install gems on the system version of ruby. Running `rbenv local` returned `2.1.4` as expected, what the heck was going on?

`gem list | grep bundler` returned nothing - doh! I didn't have bundler installed via rbenv so it was falling back to the system ruby. I was not expecting it to use a different version of bundler.

```
rbenv local
gem install bundler
rbenv rehash
bundle
```

And all was right with the world again!