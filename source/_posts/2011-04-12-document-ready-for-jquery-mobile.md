---
layout: post
title: "$(document).ready() for jQuery Mobile"
date: 2011-04-12 11:56
comments: true
categories: []
---
[jQuery Mobile](http://jquerymobile.com/) is a great framework for getting a polished mobile site up and running relatively quickly and easily.

Due to the way that it injects pages into the DOM using [Hijax](http://domscripting.com/presentations/xtech2006/), our usual techniques for running code against the current page using selectors that traverse the body no longer make sense.

Basically, there isn't a $(document).ready().

It's for this very reason that we can't rely on IDs either, since multiple copies of the same page are likely to exist in the DOM at the same time.

So how do we run page-level code against the current page when it's finished loading without the risk of changes to the other pages?

If you keep an eye on the DOM in your developer tools while pages are loaded, you'll notice that any new pages are appended in a DIV with an [HTML5 data-role attribute](http://ejohn.org/blog/html-5-data-attributes/) value of 'page'. Any code in the page, either in a script block or in an external file, will be executed immediately.

So selecting the most recent page added to the DOM is pretty straightforward:

``` js
$("div:jqmData(role='page'):last")
```

Note that you should always use the ':jqmData' pseudo-selector to ensure your code will still work if jQuery Mobile's data attributes are namespaced.

Having to repeat that constantly in all page-level code is a bit fragile, and more than a little ugly, so let's break this logic out into a standard function which we'll place into our site-wide script file.

``` js
function pageScript(func) {
  var $context = $("div:jqmData(role='page'):last");
  func($context);
}
```

Now that we have a page-level script handler, we wrap all our scripts in an anonymous function with a '$context' parameter which is a reference to the jQuery object for the current page:

``` js
pageScript(function($context){
  console.log($context);
});
```

The equivalent of $(document).ready() is achieved via the 'pagecreate' event:

``` js
pageScript(function($context){
  $context.bind("pagecreate", function(event, ui) {
    console.log("The page is ready!");
  });
});
```


The other five jQuery Mobile events available for this DIV can now be bound within this anonymous function:

``` js
pageScript(function($context){
  $context.bind("pagebeforecreate", function(event, ui) {
    console.log("The DOM is untouched by jQM");
  });

  $context.bind("pagebeforeshow", function(event, ui) {
    console.log("Before show");
  });

  $context.bind("pageshow", function(event, ui) {
    console.log("Show");
  });

  $context.bind("pagebeforehide", function(event, ui) {
    console.log("Before hide");
  });

  $context.bind("pagehide", function(event, ui) {
    console.log("Hide");
  });
});
```

This anonymous function will only run once per page load, so by binding to the event handlers you'll ensure any event-driven logic will continue to run as users navigate your mobile site.

Working with jQuery Mobile can force you to think about your event handling in a completely different way. Once you get your head around it, it allows some very cool functionality with graceful degradation baked right in, so make sure any code you write doesn't force your app to require JavaScript or you'll be fighting against the framework's emphasis on progressive enhancement.
