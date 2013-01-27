---
layout: post
title: "Mobile Parallax with Stellar.js"
date: 2012-10-02 20:41
comments: true
categories: 
---

Parallax effects can be an extremely effective way to engage users during the simple act of scrolling. What is normally a simple process of moving static content along the screen can become an act of moving through a makeshift, HTML world. While the effect can certainly be abused, when applied with a light, elegant touch, it can breathe new life into your site.

Unfortunately this effect is hampered on touch devices, where tactile feedback and intertial scrolling would perfectly suit parallax animation. Mobile browsers place a limit on script execution during scroll, so your carefully crafted parallax effects won't happen until scrolling has come to a *complete* halt. As you can imagine, this is far from impressive.

As a personal goal to not only make parallax effects as simple as possible, but to make them work on mobile browsers where these effects are notoriously difficult, I created a scrolling parallax library called [Stellar.js](http://markdalgleish.com/projects/stellar.js/).

If you're feeling impatient, you can [skip straight to the demo](/examples/mobileparallax/). At this point, you might be wondering what the issue with mobile browsers happens to be, and how Stellar.js lets us overcome this.

## The overflow problem

When we got our first taste of HTML5-capable mobile browsers, one of the first things web designers and developers noticed was their lack of scrolling overflow support.

On desktop we took it for granted that we could add a scroll bar to any element. This ability was taken away from us on these new platforms which, inside their own native applications, commonly made use of small, nested scrolling panes in order to better fit more content on such relatively tiny screens.

Plenty of people noticed this shortcoming, and some library authors weren't ready to be defeated.

## Enter touch scrolling libraries

It didn't take long for some smart developers to find ways around this limitation by creating libraries which emulate native scroll using [CSS3 transforms](https://developer.mozilla.org/en-US/docs/CSS/transform).

While their need is somewhat dimished with the *[overflow-scrolling](http://johanbrook.com/browsers/native-momentum-scrolling-ios-5/)* property, they still prove vital for overcoming our JavaScript issues. By emulating native scroll, the browser no longer throttles our script execution.

There's quite a few to choose from, and all of these will integrate with Stellar.js:

* [iScroll](http://cubiq.org/iscroll-4) by [Matteo Spinelli](http://cubiq.org/)
* [Zynga Scroller](http://zynga.github.com/scroller/) from [Zynga](http://zynga.com/)
* [Scrollability](http://joehewitt.github.com/scrollability/) by [Joe Hewitt](http://joehewitt.com/)

Before we use one of these libraries, we first need to create a standard, non-mobile parallax example.

## Marking up a simple desktop-only demo

To get started, we need a basic HTML page with jQuery, [Stellar.js](http://markdalgleish.com/projects/stellar.js/) and some parallax elements:

``` html
<!doctype html>
<html>
	<head>
		<title>Mobile Parallax with Stellar.js - Demo</title>
		<link rel="stylesheet" type="text/css" href="style.css" />
	</head>
	<body>

		<h1>Stellar.js</h1>
		<h2>Mobile Parallax Demo</h2>
		<p>Scroll down to see Stellar.js in action.</p>
				
		<section>
			<div class="pixel" data-stellar-ratio="1.1"></div>
			<div class="pixel" data-stellar-ratio="1.2"></div>
			<div class="pixel" data-stellar-ratio="1.3"></div>
			<div class="pixel" data-stellar-ratio="1.4"></div>
			<div class="pixel" data-stellar-ratio="1.5"></div>
			<div class="pixel" data-stellar-ratio="1.6"></div>
			<div class="pixel" data-stellar-ratio="1.7"></div>
			<div class="pixel" data-stellar-ratio="1.8"></div>
			<div class="pixel" data-stellar-ratio="1.9"></div>
			<div class="pixel" data-stellar-ratio="2.0"></div>
		</section>

		<script src="jquery.min.js"></script>
		<script src="jquery.stellar.min.js"></script>
		<script src="script.js"></script>
	</body>
</html>
```

A couple of files are referenced here which haven't been created yet: *"script.js"* and *"style.css"*.

## Scripting and styling

The code in *script.js* initialises Stellar.js, which activates all elements with *data-stellar-ratio* data attributes:

``` js
$(function(){
  $.stellar({
    horizontalScrolling: false,
    verticalOffset: 150
  });
});
```

In *style.css* we specify how the content should look, paying particular attention to the styling of our *"pixel"* elements:

``` css
* {
  margin: 0;
  padding: 0;
}

body {
  background: #2491F7;
}

section {
  position: relative;
  top: 400px;
  width: 300px;
  height: 3000px;
  margin: 0 auto;
}

h1, h2, p {
  color: white;
  font-family: helvetica, arial;
  text-shadow: 0 1px 2px rgba(0,0,0,0.5);
  position: absolute;
  left: 50%;
  width: 300px;
  margin-left: -150px;
}

h1 {
  font-size: 68px;
  top: 85px;
}

h2 {
  font-size: 27px;
  top: 170px;
}

p {
  font-size: 17px;
  top: 210px;
}

.pixel {
  position: absolute;
  width: 15px;
  height: 15px;
  border-radius: 15px;
  background: white;
}

  .pixel:nth-child(2)  { left: 30px;  }
  .pixel:nth-child(3)  { left: 60px;  }
  .pixel:nth-child(4)  { left: 90px;  }
  .pixel:nth-child(5)  { left: 120px; }
  .pixel:nth-child(6)  { left: 150px; }
  .pixel:nth-child(7)  { left: 180px; }
  .pixel:nth-child(8)  { left: 210px; }
  .pixel:nth-child(9)  { left: 240px; }
  .pixel:nth-child(10) { left: 270px; }
```

We now have [a very basic, desktop-only parallax demo](/examples/mobileparallax/index.pre.html) but, as you'd expect, we are going to make this work cross-platform.

## Adding mobile support

We first need to make some adjustments to the markup. We need to add a [mobile-friendly *meta* tag](http://chriscoyier.net/2009/01/25/iphone-meta-tag-for-viewport-maintaining-scale-on-orientation-change/):

``` html
<meta name="viewport" content="width=device-width; initial-scale=1.0; maximum-scale=1.0;" />
```

We next need to add a couple of divs for [iScroll](http://cubiq.org/iscroll-4), namely a *"wrapper"* div and a *"scroller"* div:

``` html
<div id="wrapper">
  <div id="scroller">
    
    <section>...</section>
  
  </div>
</div>
```

Finally, we need to include the JavaScript for iScroll:

``` html
<script src="iscroll.js"></script>
```

The complete HTML file now looks like this:

``` html
<!doctype html>
<html>
  <head>
    <meta name="viewport" content="width=device-width; initial-scale=1.0; maximum-scale=1.0;" />
    <title>Mobile Parallax with Stellar.js - Demo</title>
    <link rel="stylesheet" type="text/css" href="style.css" />
  </head>
  <body>

    <div id="wrapper">
      <div id="scroller">

        <h1>Stellar.js</h1>
        <h2>Mobile Parallax Demo</h2>
        <p>Scroll down to see Stellar.js in action.</p>

        <section>
          <div class="pixel" data-stellar-ratio="1.1"></div>
          <div class="pixel" data-stellar-ratio="1.2"></div>
          <div class="pixel" data-stellar-ratio="1.3"></div>
          <div class="pixel" data-stellar-ratio="1.4"></div>
          <div class="pixel" data-stellar-ratio="1.5"></div>
          <div class="pixel" data-stellar-ratio="1.6"></div>
          <div class="pixel" data-stellar-ratio="1.7"></div>
          <div class="pixel" data-stellar-ratio="1.8"></div>
          <div class="pixel" data-stellar-ratio="1.9"></div>
          <div class="pixel" data-stellar-ratio="2.0"></div>
        </section>
        
      </div>
    </div>

    <script src="jquery.min.js"></script>
    <script src="jquery.stellar.min.js"></script>
    <script src="iscroll.js"></script>
    <script src="script.js"></script>
  </body>
</html>
```

## Detecting mobile devices

Now that we have iScroll available, and the required markup, we need to update our *script.js* file.

We're going to be introducing some local variables that we don't want leaking into the global scope, so our first step is to wrap our code in an [immediately invoked function expression (IIFE)](/2012/07/getting-closure-in-javascript/):

``` js
(function(){
  // code goes here...
})();
```

In order to activate iScroll only when we're on a mobile WebKit browser, we need to do some dreaded user agent sniffing:

``` js
var ua = navigator.userAgent,
  isMobileWebkit = /WebKit/.test(ua) && /Mobile/.test(ua);
```

In order to style the mobile version differently, we can now use our *isMobileWebkit* flag to add a class to our HTML element:

``` js
if (isMobileWebkit) {
  $('html').addClass('mobile');
}
```

Finally, we need to initalise iScroll if we're on mobile:

``` js
var iScrollInstance;
if (isMobileWebkit) {
  iScrollInstance = new iScroll('scroller');
}
```

Now we can initalise Stellar.js differently whether we're on desktop or mobile.

## Understanding Stellar.js on mobile

Before we continue, it's important to try to understand how Stellar.js is able to work with a touch scrolling library like iScroll.

Stellar.js has the concept of *scroll properties* which are adapters that tell it how to read the scroll position. The default scroll property is, appropriately enough, *"scroll"*, which covers the standard *onscroll* style animation.

When using Stellar.js' default settings, it's equivalent to writing this code:

``` js
$(window).stellar({
  scrollProperty: 'scroll'
});
```

Here you can see that we've specified that *"window"* is the source of our scrolling, and we read its scroll position using the *"scroll"* scroll property adapter.

Once we use iScroll, there is no longer a *scroll position* in the traditional sense. Native scroll is being simulated by moving the *"scroller"* div inside the *"wrapper"* div using CSS3 transforms.

Thankfully, Stellar.js has a *"transform"* adapter to cover this exact case. All we have to do is point Stellar.js at the div being moved, and specify which *"scrollProperty"* to use:

``` js
$('#scroller').stellar({
  scrollProperty: 'transform'
});
```

## Performant positioning in Mobile WebKit

Repositioning elements rapidly is fairly demanding on mobile devices. In order to achieve a seamless effect, we need to ensure that our parallax effects are hardware accelerated.

In WebKit, [hardware acceleration is triggered by using "translate3d"](http://davidwalsh.name/translate3d). Stellar.js provides support for this method of positioning via the "transform" position property adapter:

``` js
$('#scroller').stellar({
  scrollProperty: 'transform',
  positionProperty: 'transform'
});
```

By using this adapter, our parallax effects should now be as smooth as possible.

## Hooking Stellar.js up to iScroll

Now that we've seen all the individual pieces needed to make our JavaScript work, it's time to see the final *script.js* in its entirety:

``` js
(function(){
  var ua = navigator.userAgent,
    isMobileWebkit = /WebKit/.test(ua) && /Mobile/.test(ua);

  if (isMobileWebkit) {
    $('html').addClass('mobile');
  }

  $(function(){
    var iScrollInstance;

    if (isMobileWebkit) {
      iScrollInstance = new iScroll('wrapper');

      $('#scroller').stellar({
        scrollProperty: 'transform',
        positionProperty: 'transform',
        horizontalScrolling: false,
        verticalOffset: 150
      });
    } else {
      $.stellar({
        horizontalScrolling: false,
        verticalOffset: 150
      });
    }
  });

})();
```

Of course, this won't work until we've updated our style sheet.

## Styling for mobile

Finally, we add our mobile-specific styles which act on the *body* tag and our new iScroll containers:

``` css
.mobile body {
  overflow: hidden;
}

.mobile #wrapper {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  width: 100%;
}

.mobile #scroller {
  height: 3000px;
}
```

## Seeing the end result

If you [take a look at the final page](/examples/mobileparallax/), you'll see we now have a page with scrolling parallax which works on desktop and mobile.

From here we have a base to play with our parallax site, while ensuring it works correctly cross-platform. At this point, if you haven't already, it's a good idea to read through the [Stellar.js documentation](markdalgleish.com/projects/stellar.js/docs/) to get a better idea of the control you have over the effect.

Have you made a cross-platform parallax site with Stellar.js? [Let me know on Twitter](http://twitter.com/markdalgleish), or post a link in the comments.

*Update (27 Jan 2013): This article now reflects changes made in Stellar.js v0.6*