---
layout: post
title: "CircleCI: Could not get cache stat, action npm install failed"
date: 2015-04-07 08:06
comments: true
categories: 
---

Our builds started to randomly fail on [CircleCI](https://circleci.com/) when executing npm install:

```
npm install
npm ERR! Could not get cache stat 
npm ERR! Could not get cache stat 
npm ERR! Could not get cache stat 
npm ERR! Linux 3.14.28-031428-generic
npm ERR! argv "node" "/home/ubuntu/nvm/v0.10.32/bin/npm" "install"
npm ERR! node v0.10.32
npm ERR! npm  v2.1.16
npm ERR! path /home/ubuntu/.npm/_git-remotes/git-github-com-SOMEPATH-ON-GITHUB-2799bac7/objects/pack/pack-ba6de98838bb215ca34b4ebaac78c6dbad04a80b.keep
npm ERR! code ENOENT
npm ERR! errno 34

npm ERR! enoent ENOENT, chown '/home/ubuntu/.npm/_git-remotes/git-github-com-SOMEPATH-ON-GITHUB-2799bac7/objects/pack/pack-ba6de98838bb215ca34b4ebaac78c6dbad04a80b.keep'
npm ERR! enoent This is most likely not a problem with npm itself
npm ERR! enoent and is related to npm not being able to find a file.
npm ERR! enoent 

npm ERR! Please include the following file with any support request:
npm ERR!     /home/ubuntu/PROJECT-NAME/npm-debug.log

npm install returned exit code 34
action npm install failed
```

Sometimes it would take re-running the build 3 or 4 times before going green. 

Each failure was always due to installing via [Git URL](https://docs.npmjs.com/files/package.json#git-urls-as-dependencies).

Adding `~/.npm/_git-remotes/` to the `cache_directories` of the circle configuration fixed the majority of these issues:

```
machine:
  node:
    version: 0.10.32
dependencies:
  cache_directories:
    - "node_modules"
    - "~/.npm/_git-remotes/"
  override:
    - npm prune && npm install
  post:
    - bower install
```
