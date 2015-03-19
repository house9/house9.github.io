---
layout: post
title: "upgrading from rspec 2 to rspec 3"
date: 2015-03-19 16:42
comments: true
categories: [rspec, rails]
---

I've spent the last few days upgrading a rails application from 4.1 to 4.2 and as part of the upgrade going from rspec 2.99 to 3.2.

The rspec upgrade was much easier than I had anticipated thanks to the [transpec](https://github.com/yujinakayama/transpec) [gem](https://rubygems.org/gems/transpec).

* added the [rspec-its](https://github.com/rspec/rspec-its) [gem](https://rubygems.org/gems/rspec-its) to my Gemfile
* then ran the transpec migration

```
gem install transpec
transpec --keep its --negative-form to_not
rspec spec
git commit -aeF .git/COMMIT_EDITMSG
```

Only a few minor tweaks were needed to my specs after the migration - the transpec is brilliant!
