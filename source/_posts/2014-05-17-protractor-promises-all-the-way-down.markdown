---
layout: post
title: "Protractor - promises all the way down"
date: 2014-05-17 19:54
comments: true
categories: angular
---

> Expected undefined to equal 7.

We recently started using [Protractor](https://github.com/angular/protractor) for end to end testing of an angular application. Protractor returns promises when locating elements on the page, in some cases you may need to do your assertions inside of callback functions. 

For example:

```javascript
// this works just fine
expect(element(by.id('email')).getText()).toEqual('x@x.com');

// this will fail
expect(element(by.id('email')).getText().length).toEqual(7);

// getText retuns a promise and not a string, instead...
element(by.id('email')).getText().then(function (data) {
  expect(data.length).toEqual(7);
});

```

You can use the elementexplorer to debug your locators, but be aware that even non-matched elements will return an 'Element Finder' object.

```javascript
node node_modules/protractor/bin/elementexplorer.js

Getting page at: about:blank
> browser.get('http://www.angularjs.org');
null
> element(by.model('xxxx'))
{ click: [Function],
  sendKeys: [Function],
  getTagName: [Function],
  getCssValue: [Function],
  getAttribute: [Function],
  getText: [Function],
  getSize: [Function],
  getLocation: [Function],
  isEnabled: [Function],
  isSelected: [Function],
  submit: [Function],
  clear: [Function],
  isDisplayed: [Function],
  getOuterHtml: [Function],
  getInnerHtml: [Function],
  toWireValue: [Function],
  findElements: [Function],
  isElementPresent: [Function],
  evaluate: [Function],
  findElement: [Function],
  find: [Function],
  isPresent: [Function],
  element: { [Function] all: [Function] },
  '$': [Function],
  '$$': [Function] }
```

However, invoking one of the methods on an unmatched element will raise a `NoSuchElementError`.


Resources:

* https://github.com/angular/protractor/blob/master/docs/api.md 
* http://www.youtube.com/watch?v=aQipuiTcn3U