---
layout: post
title: Self-Executing Anonymous Functions
date: 2011-03-21 10:44
comments: true
categories: []
---
When learning JavaScript, with all the attention given to variables, functions, 'if' statements, loops and event handlers, often little is done to educate you on how you might cleanly organise your code into a cohesive, structurally-sound whole.

Let's take the following code for example:

``` js
var foo = 'Hello';
var bar = 'World!';

function baz(){
	return foo  + ' ' + bar;
}

console.log(baz());
```

This style of code looks quite normal, works fine and doesn't cause any problems. At least for now.

This style of code, when implemented in a large application, can start to become an unwieldy mess. The global namespace becomes littered with functions and variables, all tenuously linked to each other through a combination of rudimentary comments and potentially unspoken developer knowledge.

The first step on the journey to beautiful, modular JavaScript is to learn the art of the self-executing anonymous function.

``` js
(function(){
	console.log('Hello World!');
})();
```

Let's look at this carefully. This code is made up of two key parts.

First is the anonymous function:

``` js
(function(){
	//Normal code goes here
})
```

The really interesting part is what happens when we add this right at the end:

``` js
();
```

Those two little brackets cause everything contained in the preceding parentheses to be executed immediately. What's useful here is that JavaScript has function level scoping. All variables and functions defined within the anonymous function aren't available to the code outside of it, effectively using closure to seal itself from the outside world.

Let's apply this design patten to our gloriously inane example code.

``` js
(function(){
	var foo = 'Hello';
	var bar = 'World!'
	
	function baz(){
		return foo + ' ' + bar;
	}
})();

 //These all throw exceptions:
console.log(foo);
console.log(bar);
console.log(baz());
```

The last three lines throw exceptions because currently nothing is accessible outside the anonymous function. To allow access to a variable or function, we need to expose it to the global 'window' object.

``` js
(function(){
	var foo = 'Hello';
	var bar = 'World!'
	
	function baz(){
		return foo + ' ' + bar;
	}

	window.baz = baz; //Assign 'baz' to the global variable 'baz'...
})();

console.log(baz()); //...and now this works.

//It's important to note that these still won't work: 
console.log(foo);
console.log(bar);
```

One of the major benefits of this pattern, as seen on the last two lines of the previous example, is that you can limit access to variables and functions within your closure, essentially making them private and only choosing to expose an API of your choice to the global scope.

One popular spin on this design pattern, which can be seen in the [jQuery source](http://code.jquery.com/jquery-1.5.1.js), is to pass in some commonly used objects. In our code we reference 'window', so let's pass that in as a parameter to the anonymous function.

``` js
(function(window){
	var foo = 'Hello';
	var bar = 'World!'
	
	function baz(){
		return foo + ' ' + bar;
	}

	//In this context, 'window' refers to the parameter
	window.baz = baz;
})(window); //Pass in a reference to the global window object
```

When minifying your code, this design pattern will yield great results. All references to 'window' in your code can be renamed to 'a', for example:

``` js
(function(a){
	console.log(a === window); //Returns 'true'
})(window);
```

Normally you'll want to pass in a few objects. A technique you can see used within jQuery itself is to reference an extra parameter that isn't defined when the anonymous function is executed, in effect creating an alias for 'undefined':

``` js
(function(window, document, $, undefined){
	var foo;
	console.log(foo === undefined); //Returns 'true'
})(window, document, jQuery);
```

You may have noticed that the previous example also aliased 'jQuery' to '$', allowing it to play nicely with other frameworks without having to use [jQuery in noConflict mode](http://api.jquery.com/jQuery.noConflict/).

It's worth pointing out that the parameter names are purely for convention. The following code would work equally as well and serves as a great illustration of what's really going on in this design pattern:

``` js
(function(mark, loves, drinking, coffee){
	mark.open('http://www.google.com'); //window
	loves.getElementById('menu'); //document
	drinking('#menu').hide(); //jQuery
	var foo;
	console.log(foo === coffee); //undefined
})(window, document, jQuery);
```

Although, for obvious reasons, I advise against this ;)

The benefits of this design pattern will become even more apparent in later posts. Harnessing the power of self-executing anonymous functions will allow you to create more complex but ultimately more intuitive code structures that will make your larger projects much easier to manage.
