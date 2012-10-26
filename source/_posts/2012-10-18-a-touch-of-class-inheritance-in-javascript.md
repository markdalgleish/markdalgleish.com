---
layout: post
title: "A Touch of Class: Inheritance in JavaScript"
date: 2012-10-18 13:05
comments: true
categories: 
---

The object-oriented features of JavaScript, such as constructors and prototype chains, are possibly the most misunderstood aspects of the language. Plenty of people who have been working full time with JavaScript still struggle with these concepts, and I argue that this is purely a result of its confusing, Java-style syntax.

But it doesn't have to be this way. Hiding beneath the "new" keyword is a rich and elegant object model, and we'll spend some time coming to grips with it.

By seeing how simple inheritance in JavaScript can be if we use more modern syntax, we'll be able to better understand the unfortunate syntax we've been stuck with from the beginning.

## JavaScript's lack of class

The biggest act of misdirection in JavaScript is that it looks like it has [classes](http://en.wikipedia.org/wiki/Class_computer_programming). When someone is new to JavaScript, particularly if they come from another language that *does* have classes, they might be surprised to find code like this:

``` js
var charlie = new Actor();
```

Which leads to quite possibly the biggest surprise people face when trying to come to grips with JavaScript: it *doesn't* have classes. You might assume that elsewhere in the code is the class definition for "Actor", but instead you find something like this:

``` js
function Actor(name) {
    this.name = name;
}

Actor.prototype.act = function(line) {
    alert(this.name + ': ' + line);
};
```

From here it only gets worse when multiple inheritance is involved:

``` js
function Actor(name) {
    this.name = name;
}
Actor.prototype.canSpeak = true;

function SilentActor() {
    Actor.apply(this, arguments);
}
SilentActor.prototype = new Actor();
SilentActor.prototype.canSpeak = false;
```

Sometimes despite years of experience, even those who realise that JavaScript doesn't have classes may still find themselves never quite grasping the concepts behind such cryptic syntax.

## Taking a step back

To put these decisions in the proper historical context, we need to take a trip back to the year 1995. Java was the hot new web language, and [Java applets were going to take over the world](http://sunsite.uakom.sk/sunworldonline/swol-07-1995/swol-07-java.html).

We all know how this story ends, but this was the environment in which [Brendan Eich was toiling away at Netscape on his "little" web scripting langauge](https://brendaneich.com/2010/07/a-brief-history-of-javascript/).

This cultural pressure resulted in some rebranding. What was originally "Mocha", then "LiveScript", eventually became known as "JavaScript". While it may have eventually extended beyond its original scope, Brendan Eich wasn't shy about pitching it as Java's little brother.

## It's just a prototype

The problem, of course, is that despite JavaScript's syntactic similarities to Java, its conceptual roots lay elsewhere. A dynamically typed language, it borrowed functional aspects from [*Scheme*](http://en.wikipedia.org/wiki/Scheme_programming_language), and prototypal inheritance from [*Self*](http://en.wikipedia.org/wiki/Self_programming_language).

In classical languages, like Java, instances are created from classes. JavaScript differs in that it has [prototypal inheritance](http://en.wikipedia.org/wiki/Prototype-based_programming), in which there are no classes.

Instead, *objects inherit from other objects*.

## Modern inheritance

To best understand the concepts behind prototypal inheritance, you must unlearn what you have learned. You'll find it is much easier to appreciate object-to-object inheritance if you pretend you've never seen JavaScript's misleading, classically-styled syntax.

In ECMAScript 5, the latest version of JavaScript available in all modern browsers (Chrome, Firefox, Safari, Opera, IE9+), we have new syntax for creating object-to-object inheritance, based on [Douglas Crockford's original utility method for simple inheritance](http://javascript.crockford.com/prototypal.html):

``` js
var childObject = Object.create(parentObject);
```

If we use [*Object.create*](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Object/create) to recreate our *'SilentActor'* example from earlier, it becomes much easier to see what's really going on:

``` js
// Our 'actor' object has some properties...
var actor = {
  canAct: true,
  canSpeak: true
};

// 'silentActor' inherits from 'actor'
var silentActor = Object.create(actor);
silentActor.canSpeak = false;

// 'busterKeaton' inherits from 'silentActor'
var busterKeaton = Object.create(silentActor);
```

In modern browsers, we also have a [new method for reliably inspecting the prototype chain](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Object/GetPrototypeOf):

``` js
Object.getPrototypeOf(busterKeaton); // silentActor
Object.getPrototypeOf(silentActor); // actor
Object.getPrototypeOf(actor); // Object
```

In this simple example, we've been able to set up a prototype chain without using a single constructor or "new" keyword.

## Walking the chain

So how does a prototype chain work? When we try to read a property from our new *'busterKeaton'* object, it checks the object itself and, if it didn't find the property, it traverses up the prototype chain, checking each of its prototype objects in order until it finds the first occurence of the property.

All this happens when we ask for the value of a property from an object.

``` js
busterKeaton.canAct;
```

In order to properly evaluate the value of *'canAct'* on *'busterKeaton'*, the JS engine does the following:

1. Checks *'busterKeaton'* for the *'canAct'* property, but finds nothing.
2. Checks *'silentActor'*, but doesn't find the *'canAct'* property.
3. Checks *'actor'* and finds the *'canAct'* property, so returns its value, which is 'true'.

## Modifying the chain

The interesting thing is that the *'actor'* and *'silentActor'* objects are still live in the system and can be modified at runtime.

So, for a contrived example, if all silect actors lost their jobs, we could do the following:

``` js
silentActor.isEmployed = false;

// So now...
busterKeaton.isEmployed; // false
```

## Where's "super"?

In classical languages, when overriding methods, you can run the method from the parent class in the current context.

In JavaScript we have even more options: *we can run any function in any context*.

How do we achieve this? Using JavaScript's [*'call'*](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Function/call) and [*'apply'*](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Function/apply) methods which are available on all functions. Appropiately enough, these are available because they exist on the 'Function' prototype.

They both allow us to run a function in a specific context which we provide as the first parameter, along with any arguments we wish to pass to the function.

Using our *'silentActor'* example, if we want to achieve the equivalent of calling *'super'*, it looks something like this:

``` js
actor.act = function(line) {
    alert(this.name + ': ' + line);
}

silentActor.act = function(line) {
    // Super:
    actor.act.call(this, line);
};
```

If it took multiple arguments, it would look like this:

``` js
silentActor.act = function(line, emotion) {
    // Using 'call':
    actor.act.call(this, line, emotion);

    // Using 'apply':
    actor.act.apply(this, [line, emotion]);
};
```

## Where are the constructors?

With this setup, it is quite simple for us to create our own DIY constructor:

``` js
function makeActor(name) {
    // Create a new instance that inherits from 'actor':
    var a = Object.create(actor);

    // Set the properties of our instance:
    a.name = name;

    // Return the new instance:
    return a;
}
```

## Understanding prototypes with a touch of class

Now that we've seen how simple object-to-object inheritance can be, it's time to look at our original example again with fresh eyes:

``` js
var charlie = new Actor();
```

*'Actor'* is, obviously, not a class. It is, however, a function.

It could be a function that does nothing:

``` js
function Actor() {}
```

Or, it could set some properties on the new instance:

``` js
function Actor(name) {
    this.name = name;
}
```

In our example, the *'Actor'* function is being used as a constructor since it is invoked with the "new" keyword.

The funny thing about JavaScript is that *any* function can be used as a constructor, so there's nothing special about our *'Actor'* function that makes it different from any other function.

## Clarifying "constructors"

Since any function can be a constructor, all functions have a *'prototype'* property just in case they're used as a constructor. Even when it doesn't make sense.

A perfect example is the [*'alert'*](https://developer.mozilla.org/en-US/docs/DOM/window.alert) function, which is provided by the browser. Even though it's not meant to be used as a constructor (in fact, the browser won't even let you), it still has a *'prototype'* property:

``` js
typeof alert.prototype; // 'object'

new alert(); // TypeError: Illegal invocation
```

When *'Actor'* is used as a constructor, our new *'charlie'* object inherits from the object sitting at *'Actor.prototype'*.

## Functions are objects

If you missed that, let me re-iterate. *[Functions are objects](http://my.safaribooksonline.com/book/programming/javascript/9780596517748/functions/function_objects), and can have arbitrary properties*.

For example, this is completely valid:

``` js
function Actor(){}

// We can set any property:
Actor.foo = 'bar';
Actor.abc = [1,2,3];
```

However, the *'prototype'* property of a function is where we store the object that all new instances will inherit from:

``` js
function Actor(){}

// Used for constructors:
Actor.prototype = {foo: 'bar'};

var charlie = new Actor();
charile.foo; // 'bar'
```

## Creating the chain

Setting up multiple inheritance with our class-like syntax should now be a bit easier to grasp.

It's simply a case of making a function's *'prototype'* property inherit from another function's *'prototype'*:

``` js
// Set up Actor
function Actor() {}
Actor.prototype.canAct = true;

// Set up SilentActor to inherit from Actor:
function SilentActor() {}
SilentActor.prototype = Object.create(Actor.prototype);

// We can now add new properties to the SilentActor prototype:
SilentActor.prototype.canSpeak = false;

// So instances can act, but can't speak:
var charlie = new SilentActor();
charlie.canAct; // true
charlie.canSpeak; // false
```

## Construction: Just 3 simple steps

At this point, it's important to point out what's going on inside a constructor. Instantiating a new object with a "constructor" performs three actions and, as an example, let's take a look at what happens with our *'Actor'* constructor:

``` js
var charlie = new Actor();
```

This one line of the code does the following:

1. Creates a new object, *'charlie'*, that inherits from the object sitting at *'Actor.prototype'*,
2. Runs the *'Actor'* function against *'charlie'*, and
3. Returns *'charlie'*.

## Coming full circle

You might think that this sounds awfully familiar because this precisely mirrors the steps in our "DIY constructor" from earlier, which looked like this:

``` js
function makeActor(name) {
    // Create a new instance that inherits from 'actor':
    var a = Object.create(actor);

    // Set the properties of our instance:
    a.name = name;

    // Return the new instance:
    return a;
}
```

Just like our earlier example, asking our *'busterKeaton'* object for the *'canAct'* property will walk up the prototype chain, except this time the JS engine will act slightly differently:

1. Checks *'busterKeaton'* for the *'canAct'* property, but finds nothing.
2. Checks *'SilentActor.prototype'*, but doesn't find the *'canAct'* property.
3. Checks *'Actor.prototype'* and finds the *'canAct'* property, so returns its value.

You'll notice that the chain now consists of objects sitting at the *'prototype'* property of functions, each of which had been used as the object's constructor.

## Keeping it classic

Even after understanding the syntax, it's still common for people to want to get as far away from it as possible.

There are implementations of "classical" inheritance built on top of JavaScript's prototypal inhertance, the most notable of which is [John Resig's famous "Class" function](http://ejohn.org/blog/simple-javascript-inheritance/).

You'll also find a lot of languages that compile to JavaScript, such as [CoffeeScript](http://coffeescript.org/) or [TypeScript](http://www.typescriptlang.org/), offer standard class syntax. TypeScript in particular ensures it closely resembles the [draft ECMAScript 6 class specification](http://wiki.ecmascript.org/doku.php?id=harmony:classes).

Yes, the ES6 specification draft contains "class" sugar which, like CoffeeScript and TypeScript, is really prototypal inheritance under the hood.

## Clearing the air

Of course none of the classical misdirection and shim layers can really replace a true understanding of prototypal inheritance and the power it affords you.

Hopefully this article has helped clear things up for you and, if you still find yourself puzzled, I hope at the very least that the next time you dig into prototypal inheritance, your mind will have been sufficiently prepared for thinking like Brendan Eich did all those years ago.

If not, there's always Java applets.

### Slides from Web Directions South 2012

This article is based on a presentation I gave on 18 October, 2012 at [Web Directions South](http://south12.webdirections.org/) in Sydney, Australia. The [slides are available online](http://markdalgleish.com/presentations/atouchofclass).