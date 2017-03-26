---
layout: post
title: "Save a ruby hash as a json file"
date: 2017-03-26 12:24
comments: true
categories: rails
---
Save a ruby hash as a json file

I have found the following technique useful when debugging json structures generated from ruby hashes.

```
some_json = {
  some_attribute: 'SOME_DATA'
}


IO.write "tmp/some_json-#{SecureRandom.hex(2)}.json", JSON.pretty_generate(some_json)
```
