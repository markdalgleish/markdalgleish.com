---
layout: post
title: "$.html5data - Creating Highly Configurable jQuery Plugins - Part 2"
date: 2011-09-10 09:08
comments: true
categories: []
---
<p>In my previous post on allowing users of your plugin to customise it in many different ways, I briefly touched on the use of HTML5 data attributes. If you want to provide a hash of settings from your markup, one way to do this is by writing an object literal as the value of a data attribute:</p>

``` html
<div data-myplugin='{foo:"true", bar: "10"}'></div>
```

<p>Even though this works, I react the same way to an object literal in markup as most JavaScript developers react to an onclick attribute.</p>

<p>If we are to follow the conventions of HTML when writing HTML (as we should) then each setting should have its own attribute.</p>

<p>The 'data' method in jQuery, which previously had nothing to do with HTML5 data attributes, now (as of v1.4.3) reads data values from the DOM. For example:</p>

``` html
<div id="foo" data-foo="true" data-bar="10"></div>
```

<p>These options can be turned into a single hash by calling $('#foo').data(). The problem is that we have now introduced another form of the global namespace problem. All plugins that use HTML5 data attributes have the potential to conflict with the attributes from another plugin, especially if some of the settings are particularly generic. It's not hard to imagine more than one plugin using 'data-width' or 'data-height'.</p>

<p>To counter this, we are encouraged to namespace our data attributes, so 'data-width' becomes 'data-myplugin-width'. The problem is that by following best practices, we essentially break the useful functionality that jQuery's 'data' method provides us:</p>

``` html
<div id="foo" data-myplugin-foo="true" data-myplugin-bar="10"></div>
```

<p>Becomes:</p>

``` js
{
  mypluginFoo: true,
  mypluginBar: 10
}
```

<p>This is no longer in a format that we can merge with our settings through '$.extend' since the property names won't match.</p>

<p>At this stage we could write some custom code to iterate over the object, filter out anything that doesn't begin with 'myplugin', then strip off our namespace and make 'Foo' and 'Bar' lower case. But this is way too much work to do regularly and ideally should be available to us in the form of a jQuery plugin.</p>

<p>This is where the <a href="http://markdalgleish.com/projects/jquery-html5data" target="_blank">$.html5data plugin</a> comes in. Unlike jQuery's 'data' method, it is designed from the ground up to work with HTML5 data attributes. Because it isn't overloaded to read and write data in memory stored against DOM elements, like the 'data' method is, it is able to have an API better suited for data attributes.</p>

<p>Using the previous example, if you wanted to get an object containing all the settings that use the 'myplugin' prefix, $.html5data lets you specify the namespace:</p>

``` js
$('#foo').html5data('myplugin');
```

<p>This would return the following object, no matter which other data attributes were set:</p>

``` js
{
  foo: true,
  bar: 10
}
```

<p>Visit the <a href="http://markdalgleish.com/projects/jquery-html5data" target="_blank">$.html5data project page</a> for more information or <a href="https://github.com/markdalgleish/jquery-html5data" target="_blank">contribute on GitHub</a>.</p>
