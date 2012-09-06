---
layout: post
title: jQuery Selector for HTML5 Input Types
date: 2011-05-04 22:26
comments: true
categories: []
---
<p>With HTML5 comes a new set of input types, including search and date types. The problem is that currently most of these are currently treated as textboxes and the following selectors won't pick them up:</p>

``` js
$('input[type=text]');
$('input:text');
```

<p>Obviously, you could select just the type of input you like by substituting 'input[type=text]' with 'input[type=search]', but if you need to attach the same logic to all textbox-style HTML5 inputs, your selector will quickly get a little messy.</p>

<p>As a solution to this, I've created the following HTML5 input selector for jQuery called ':textall'.</p>

``` js
// ':textall' jQuery pseudo-selector for all text input types
(function($) {
  var types = 'text search number email datetime datetime-local date '
        + 'month week time tel url color range'.split(' '),
      len = types.length;
  $.expr[':']['textall'] = function(elem) {
    var type = elem.getAttribute('type');
    for (var i = 0; i < len; i++) {
      if (type === types[i]) {
        return true;
      }   
    }
    return false;
  };
})(jQuery);
```

<p>With this you can easily select all input types listed in the 'types' array with one pseudo-selector.</p>

``` js
$('input:textall');
```

<p>Feel free to add or remove input types as necessary. Obviously you will get better performance by having less attributes to test. This also serves s a great blueprint for creating similar selectors for your projects.</p>
