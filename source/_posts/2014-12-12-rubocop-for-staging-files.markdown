---
layout: post
title: "rubocop script for your modified files"
date: 2014-12-12 16:08
comments: true
categories: ruby
---

Our CI server is setup to run [rubocop](https://github.com/bbatsov/rubocop) (ruby linter) after every checkin.
On my local development machine I prefer to lint files that have recently changed and not the entire code base.

Add this code to a shell script and it will lint all files that have been staged.
Note it uses the autofix rubocop option, optionally you could run it on non-staged files without the autofix flag.

```
git diff --name-only --cached | xargs rubocop -a
```

Don't forget to `git add .` after rubocop finds/fixes any code style violations in your staged files.

