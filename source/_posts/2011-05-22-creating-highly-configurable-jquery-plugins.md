---
layout: post
title: Creating Highly Configurable jQuery Plugins
date: 2011-05-22 01:46
comments: true
categories: []
---
When writing jQuery plugins, you have many options for how you might structure your code.

Depending on the complexity of your plugin, you'll probably want to allow users of your plugin to exert some level of control over how it operates. There are a few ways to do this, but I like to ensure that there are many ways even within a single plugin to allow the greatest level of flexibility.

Let's start off with a basic and decidedly pointless plugin:

``` js
(function(window, $){
  var Plugin = function(elem){
      this.elem = elem;
      this.$elem = $(elem);
    };
  
  Plugin.prototype = {
    init: function() {      
      this.displayMessage();
      return this;
    },
    displayMessage: function() {
      alert('Hello World!');
    }
  };
  
  $.fn.plugin = function() {
    return this.each(function() {
      new Plugin(this).init();
    });
  };
  
  window.Plugin = Plugin;
})(window, jQuery);
```

If the first and last lines look unfamiliar to you, the plugin is wrapped in a [self-executing anonymous function](http://markdalgleish.com/2011/03/self-executing-anonymous-functions/).

One thing to note is that even in this basic example, we've already allowed for a couple of use cases. The plugin logic is not nested inside a jQuery plugin. The jQuery plugin merely allows for easy instantiation of the Plugin object. If the user of your plugin prefers, she might prefer to instantiate it manually by calling the following:

``` js
var elem = document.getElementById('elem'),
    p = new Plugin(elem).init();
```

Calling the plugin this way allows continued access to the innards of the plugin. This might allow the creation of a custom interface to your plugin, for example. The user of your plugin might like to trigger another alert, like so:

``` js
p.displayMessage();
```

Currently, the plugin is extra useless. Let's make it a little less so by allowing the message to be customised.

``` js
(function(window, $){
  var Plugin = function(elem, options){
      this.elem = elem;
      this.$elem = $(elem);
      this.options = options;
    };
  
  Plugin.prototype = {
    defaults: {
      message: 'Hello world!'
    },
    init: function() {
      this.config = $.extend({}, this.defaults, this.options);
      
      this.displayMessage();

      return this;
    },
    displayMessage: function() {
      alert(this.config.message);
    }
  };
  
  Plugin.defaults = Plugin.prototype.defaults;
  
  $.fn.plugin = function(options) {
    return this.each(function() {
      new Plugin(this, options).init();
    });
  };
  
  window.Plugin = Plugin;
})(window, jQuery);
```

As you can see, we've introduced a set of defaults within the plugin and allowed it to be extended globally or using an object literal either of the following ways:

``` js
//Set the message per instance:
$('#elem').plugin({
  message: 'Goodbye World!'
});

var p = new Plugin(document.getElementById('elem'), {
  message: 'Goodbye World!'
}).init();

//Or, set the global default message:
Plugin.defaults.message = 'Goodbye World!';
```

This was due to the wonderful [jQuery utility method called 'extend'](http://api.jquery.com/jQuery.extend/). Extending a set of object literals merges the contents of each into the object specified as the first argument. As is the case with our plugin, the first item should be an empty object literal if you don't want to override anything.

But what if we want to customise the message displayed on a per-element basis? I admit, with this example it makes little sense, but bear with me. At present, the message can be set globally, or it can be set per collection of elements. But if the user of your plugin wanted to override it easily for a small subset of elements, they're out of luck. Let's change that:

``` js
(function(window, $){
  var Plugin = function(elem, options){
      this.elem = elem;
      this.$elem = $(elem);
      this.options = options;
      this.metadata = this.$elem.data('plugin-options');
    };
  
  Plugin.prototype = {
    defaults: {
      message: 'Hello world!'
    },
    init: function() {
      this.config = $.extend({}, this.defaults, this.options, this.metadata);
      
      this.displayMessage();

      return this;
    },
    displayMessage: function() {
      alert(this.config.message);
    }
  }
  
  Plugin.defaults = Plugin.prototype.defaults;
  
  $.fn.plugin = function(options) {
    return this.each(function() {
      new Plugin(this, options).init();
    });
  };
  
  window.Plugin = Plugin;
})(window, jQuery);
```

We've set the 'metadata' property by getting the [HTML5 data attribute](http://ejohn.org/blog/html-5-data-attributes/) 'data-plugin-options'. Now, you can customise the plugin on an element basis like so:

``` js
<div id="elem" data-plugin-options='{"message":"Goodbye World!"}'></div>
```

Notice the subtle change in the usage of single and double quotation marks.

Now our useless plugin can be setup and instantiated in a myriad of ways to suit the end users needs.

When designing a plugin, it's important to allow users to interact with it in a way that negates the need to change the original source code. Opening up the number of was in which the default configuration can be changed is the first step in that process.
