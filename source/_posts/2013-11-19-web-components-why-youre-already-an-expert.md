---
layout: post
title: "Web Components: Why You're Already An Expert"
date: 2013-11-19 14:05
comments: true
categories: 
---

[Web Components](http://w3c.github.io/webcomponents/explainer/) are going to fundamentally change the nature of HTML.

At first glance, they may seem like a complicated set of new technologies, but Web Components are built around a simple premise. Developers should be free to act like browser vendors, extending the vocabulary of HTML itself.

If you're intimidated by these new technologies or haven't experimented with them yet, this post has a very simple message for you. If you're already familiar with HTML elements and DOM APIs, you are already an expert at Web Components.

## "Components" today

To understand why Web Components are so important, we need look no further than how we've hacked around the *lack* of Web Components.

As an example, let's run through the process of consuming a typical third-party widget.

First, we include the widget's CSS and JavaScript:

```html
<link rel="stylesheet" type="text/css" href="my-widget.css" />
<script src="my-widget.js"></script>
```

Next, we might need to add placeholder elements to the page where our widgets will be inserted.

```html
<div data-my-widget></div>
```

Finally, when the DOM is ready, we reach back into the document, find the placeholder elements and instantiate our widgets:

```js
// jQuery used here for brevity...

$(function() {
  $('[data-my-widget]').myWidget();
});
```

This was a bit of work, but we're *still* not finished.

We've introduced a custom widget to the page, but it is not aware of the browser's element lifecycle. This becomes clear if we update the DOM:

```js
el.innerHTML = '<div data-my-widget></div>';
```

Since this isn't a typical element, we now must manually instantiate any new widgets as we update the document:

```js
$(el).find('[data-my-widget]').myWidget();
```

The most common way to avoid this constant two-step process is to completely abstract away DOM interaction. Unfortunately, that's a pretty heavy-handed solution that usually results in widgets being tied to particular libraries or frameworks.

## Component soup

Once our widgets have been instantiated, our placeholder elements have been filled with third-party markup:

```html
<div data-my-widget>
  <div class="my-widget-foobar">
      <input type="text" class="my-widget-text" />
      <button class="my-widget-button">Go</button>
  </div>
</div>
```

This markup is now sitting in the same context as our application markup.

Its internals are visible when we traverse the DOM, and the styles for this widget exist in the same global context as our styles, leading to a high risk of style clashes. All of their classes must be carefully namespaced with `my-widget-` (or something similar) to avoid naming collissions.

Our code is now all mixed up with the third-party code, with no clean separation between the two. Basically, there is no [encapsulation](http://en.wikipedia.org/wiki/Encapsulation_object-oriented_programming).

## Web Components to the rescue

With Web Components, it becomes clear what we've been missing.

```html
<!-- Import it: -->
<link rel="import" href="my-widget.html" />

<!-- Use it: -->
<my-widget />
```

In this case, we've imported a new custom element with a single import and used it immediately.

More importantly, since `<my-widget />` is an actual element, it hooks into the browser's element lifecycle, allowing us to add a new instance to the page like we would with any native widget:

```js
el.innerHTML = '<my-widget />';
// The widget is now instantiated
```

When we inspect this element, we can see that it is a single element. However, if we [enable Shadow DOM in our developer tools](http://chemicaloliver.net/programming/inspecting-the-shadow-dom-in-google-chrome-inspector/), we see something very interesting.

Hidden inside this element is its private implementation details, in the form of a [document fragment](https://developer.mozilla.org/en/docs/Web/API/DocumentFragment):

```html
#document-fragment
  <div>
    <input type="text" />
    <button>Go</button>
  </div>
```

While these elements are visible to the naked eye, they are hidden from us when we traverse the DOM or write CSS selectors. To the outside world, even when instantiated, our custom widget is still just a single element.

We finally have a simple, encapsulated widget that behaves exactly like a standard HTML element.

## In the interest of time

When we talk about Web Components, we're not talking about a single technology.

We're talking about a suite of new tools that are each useful in their own right: [Custom Elements](http://www.html5rocks.com/en/tutorials/webcomponents/customelements/), [Shadow DOM](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/), [HTML Templates](http://www.html5rocks.com/en/tutorials/webcomponents/template/), [HTML Imports](http://www.html5rocks.com/en/tutorials/webcomponents/imports/), and [Decorators](http://w3c.github.io/webcomponents/explainer/#decorator-section).

The primary goal of Web Components is to give us the encapsulation we've been missing. Luckily, this goal can be achieved purely with Custom Elements and Shadow DOM. So, in the interest of time, we'll begin by focusing on these two technologies.

Rather than immediately jumping into a list of new browser features, I find it helpful for us to first reacquaint ourselves with what we already know about the native elements we've been consuming for years. After all, it doesn't hurt for us to understand what we're trying to build.

## What we already know about elements

We know that elements can be instantiated through markup or JavaScript:

```html
<input type="text" />
```

```js
document.createElement('input');
el.innerHTML = '<input type="text" />';
```

We know that elements are instances:

```js
document.createElement('input') instanceof HTMLInputElement; // true
document.createElement('div') instanceof HTMLDivElement; // true
```

We know that elements perform their own initialisation:

```js
// If we create an input with a value attribute defined...
el.innerHTML = '<input type="text" value="foobar" />';

// ...the value *property* is already in sync
el.querySelector('input').value;
```

We know that elements can respond to attribute changes:

```js
// If we change the value *attribute*...
input.setAttribute('value', 'Foobar');

// ...the value *property* updates accordingly
input.value === 'Foobar'; // true
```

We know that elements can have hidden internal DOM structures:

```html
<!-- A single 'input' provides a complex calendar  -->
<input type="date" />
```

```js
// Despite its complexity, to us it's still just a single element
dateInput.children.length === 0; // true
```

We know that elements have access to child elements:

```html
<!-- We can provide as many 'option' tags as we like -->
<select>
  <option>1</option>
  <option>2</option>
  <option>3</option>
</select>
```

We know that elements can provide style hooks to their internals:

```css
dialog::backdrop {
  background: rgba(0, 0, 0, 0.5);
}
```

Finally, we know that elements can have their own private styles. Unlike the custom widgets of today, we never need to manually include CSS for the browser's native widgets.

By understanding all of this, we're well on our way to understanding Web Components. With Custom Elements and Shadow DOM, we can now recreate all of this standard behaviour in our widgets.

## Custom elements

Registering a new element can be as simple as this:

```js
var MyElement = document.register('my-element');
// 'document.register' returns a constructor
```

You might have noticed that our element name contains a hyphen. This is an important requirement for Custom Elements to ensure our tag names don't clash with current or future elements.

This element now works like any other native element:

```html
<my-element />
```

Which, of course, means that our element works with all the standard DOM APIs:

```js
document.create('my-element');

el.innerHTML = '<my-element />';

document.create('my-element') instanceof MyElement; // true
```

## Breathing life into our custom element

Currently, this is a pretty useless element. Let's give it some content:

```js
// We'll now provide the second argument to 'document.register'
document.register('my-element', {
  prototype: Object.create(HTMLElement.prototype, {

    createdCallback: {
      value: function() {
        this.innerHTML = '<h1>ELEMENT CREATED!</h1>';
      }
    }

  })
});
```

In this example, we've set up the [prototype](http://markdalgleish.com/2012/10/a-touch-of-class-inheritance-in-javascript/) for our custom element, using [Object.create](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create) to make a new object that inherits from the [HTMLElement](https://developer.mozilla.org/en/docs/Web/API/HTMLElement) prototype.

We've defined a `createdCallback` function which will run every time a new instance of the element is created.

We could also optionally define `attributeChangedCallback`, `enteredViewCallback` and `leftViewCallback`.

Inside our callback we can modify our new element however we like. In this case, we've set its `innerHTML`.

So far we're able to dynamically modify the contents of our custom element, but this isn't too different from the custom widgets of today. In order to complete the picture, we need a way to provide encapsulation to our new element by hiding its internals.

## Encapsulation with Shadow DOM

We're going to modify our `createdCallback` a bit.

This time, instead of setting the `innerHTML` directly on our custom element, we're going to do something quite different:

```js
createdCallback: {
  value: function() {

    var shadow = this.createShadowRoot();
    shadow.innerHTML = '<h1>SHADOW DOM!</h1>';

  }
}
```

In this example, you would see the words 'SHADOW DOM!' when looking at the page, but inspecting the DOM would reveal a single, empty `<my-element />` tag. Instead of modifying the containing page, we've created a new shadow root inside our custom element using `this.createShadowRoot()`.

Anything inside of this shadow root, while visible to the naked eye, is hidden from DOM APIs and CSS selectors in the containing page, maintaining the illusion that this widget is only a single element.

If we were writing a custom calendar widget, the shadow root is where our complex calendar markup would go, allowing us to expose a single tag as a simple interface to its hidden complexity.

## Accessing the "light DOM"

So far, our custom element is just an empty tag, but what happens if elements were nested inside our new component? We may want a widget with similar flexibility to the `<select>` tag, which can contain many `<option>` tags.

As a working example, let's assume the following markup.

```html
<my-element>
  This is the light DOM.
  <i>hello</i>
  <i>world</i>
</my-element>
```

As soon as a new shadow root is created against this custom element, its child nodes are no longer visible. We refer to these hidden child nodes as the "light DOM". If we inspect the page or traverse the DOM we can see these hidden nodes, but the end user would have no clue these elements exist at all.

Without shadow DOM, this example would simply appear as 'This is the light DOM. *hello world*'.

When we set up the shadow DOM inside the `createdCallback` function, we can use the new `content` tag to distribute elements from the light DOM into the shadow DOM:

```js
createdCallback: {
  value: function() {

    var shadow = this.createShadowRoot();
    // The child nodes, including 'i' tags, have now disappeared

    shadow.innerHTML =
      'The "i" tags from the light DOM are: ' +
      '<content select="i" />';
    // Now, only the 'i' tags are visible inside the shadow DOM
  }
}
```

With shadow DOM and the `<content>` tag, this now appears as 'The "i" tags from the light DOM are: *helloworld*'. Note that the `<i>` tags have rendered next to each other with no whitespace.

## Encapsulating styles

What's important to understand about Shadow DOM is that we've created a clean separation between our widget's markup and the outside world, known as the shadow boundary.

One powerful feature of Shadow DOM is that styles declared inside do not leak outside of the shadow boundary.

```js
createdCallback: {
  value: function() {

    var shadow = this.createShadowRoot();
    shadow.innerHTML =
      "<style>span { color: green }</style>" +
      "<span>I'm green</span>";

  }
}
```

Even though a very gereric `span` style was defined in the shadow DOM, it has no effect on `<span>` tags in the containing page:

```html
<my-element />
<span>I'm not green</span>
```

If we're distributing elements from the light DOM into the shadow DOM, like in our `<i>` example earlier, it's important to understand that these nodes don't technically belong to our widget.

Distributed nodes still belong to the containing page, meaning that we can't style these elements by simply writing a standard selector.

Instead, we must style these distributed elements with the `::content` pseudo-element:

```css
::content i {
  color: blue;
}
```

Which, in the context of a component, looks something like this:

```js
createdCallback: {
  value: function() {

    var shadow = this.createShadowRoot();
    shadow.innerHTML =
      '<style>::content i { color: blue; }</style>' +
      'The "i" tags from the light DOM are: ' +
      '<content select="i" />';
  }
}
```

## Exposing style hooks

When we hide the internal markup of our custom element, it is still sometimes desirable to allow certain aspects of our element to be re-styled from outside.

For example, if we were writing a custom calendar widget, we might want to allow end users to style the buttons, without giving them access to the entirety of our widget's markup.

This is where the `part` attribute and pseudo element comes in:

```js
createdCallback: {
  value: function() {

    var shadow = this.createShadowRoot();
    shadow.innerHTML = 'Hello <em part="world">World</em>';

  }
}
```

The `::part()` pseudo element lets us style any element with a `part` attribute:

```css
my-element::part(world) {
  color: green;
}
```

This `part` contract is essential in maintaining encapsulation. In the previous example, we styled the word "World", but users of our widget would have no idea that it's actually an `em` tag under the hood.

One important benefit of this system is that we're free to dramatically change the markup inside our widget between versions, so long as the "part" attributes are still in place.

## Just the beginning

Web Components finally give us a way to achieve simple, consistent, reusable, encapsulated and composable widgets, but we're only just getting started. It's a great time to start experimenting.

Before you can begin, you need to make sure your browser has the relevant features enabled. If you use Chrome, head to `chrome://flags` and enable "experimental Web Platform features".

To target browsers that don't have these features enabled, you can use Google's [Polymer](http://www.polymer-project.org/), or Mozilla's [X-Tag](http://www.x-tags.org/).

## Time to experiment

All of the functionality presented in this article is simply an exercise in emulating standard browser behaviour. We've been working with the browser's native widgets for a long time, so taking the step towards writing our own isn't as difficult as it might seem.

If you haven't created a component before, I urge you to open up the console and experiment. Try making a custom element, then try creating a shadow root (against any element, not just Custom Elements).

This experimentation will naturally raise questions about topics not fully discussed in this article. Do we have to use strings to define the markup in our Shadow DOM? No, this is where [HTML Templates](http://www.html5rocks.com/en/tutorials/webcomponents/template/) come in. Can we bundle an HTML template with our component's JavaScript? Yes, with [HTML Imports](http://www.html5rocks.com/en/tutorials/webcomponents/imports/).

Even if [it's too early to use this stuff in production](http://www.polymer-project.org/faq.html#readiness), it's never too early to be prepared for the future of the web.

*This article is based on a 15 minute presentation from Web Directions South 2013 in Sydney, Australia [(Slides)](http://markdalgleish.github.io/presentation-web-components/).*

*Please note: The Web Component APIs have been in constant flux, so if anything in this article is out of date, please let myself and others know in the comments.*