---
layout: post
title: Using 3D CSS Transitions with Fathom.js
date: 2012-03-10 00:37
comments: true
categories: []
---
<p>If you've started making presentations using the browser, your work might still resemble a standard Keynote or Powerpoint presentation. The main benefit of creating presentations in HTML is that you have all the power of modern browsers available to you. This article will focus on just one aspect of this: CSS 3D transforms and transitions.</p>

<h3>Presenting Fathom.js</h3>

<p><a href="http://markdalgleish.com/projects/fathom" target="_blank">Fathom.js</a>, the jQuery-based presentation framework I created for <a href="http://melbjs.com/" target="_blank">MelbJS</a>, might look fairly simple but its simplicity is deceptive. For example, you can build fairly powerful yet straightforward 3D transitions purely with CSS. It's easier than you might think.</p>

<p>It's important to note that Fathom.js uses CSS classes to indicate which slides are active or inactive. By default, all slides have the class of 'slide'. When a slide is active it has the class 'activeslide', and it has the class 'inactive' when (you guessed it) it's inactive. This allows us to add different styles to each state and a transition between them with a few lines of CSS. Now that we have 3D transforms at our disposal, let's create an example 3D transition.</p>

<h3>Let's Get Started</h3>

<p>First of all, we'll create a standard Fathom.js presentation.</p>

``` html
<!DOCTYPE html>
<html>
  <head>
    <title>Fathom.js 3D CSS Transitions</title>
    <script src="jquery.min.js"></script>
    <script src="fathom.js"></script>
    <script>
      $(function(){
        $('#presentation').fathom();
      });
    </script>
    <style type="text/css">
      .slide {
        width: 800px;
        height: 600px;
      }
      .slide .content {
        width: 800px;
        height: 600px;
        background-color: #202020;
        border-radius: 10px;
        font-family: Helvetica, Arial, sans-serif;
        font-size: 20px;
      }
      h1 {
        font-family: Georgia, Times New Roman, serif;
        font-size: 24px;
      }
    </style>
  </head>
  <body>

    <div id="presentation">

      <div class="slide">
        <div class="content">
          <h1>Hello there</h1>
          <p>This is a test slide. In 3D.</p>
        </div>
      </div>

      <div class="slide">
        <div class="content">
          <h1>Hello there</h1>
          <p>This is a test slide. In 3D.</p>
        </div>
      </div>

      <div class="slide">
        <div class="content">
          <h1>Hello there</h1>
          <p>This is a test slide. In 3D.</p>
        </div>
      </div>

    </div>

  </body>
</html>
```

<p>At the moment, this is a fairly run-of-the-mill presentation, except that we've added an extra 'content' div as a container inside the 'slide' div. This is so that Fathom.js can use the positioning of 'div.slide' for its calculations, while we're free to modify any elements nested inside it.</p>

<h3>Getting Some Perspective</h3>

<p>Knowing this, our first step is to add a perspective to the 'slide' class, which is required before the browser will render transforms in 3D, and a 3D transform to the inactive slide's 'content' class.</p>

``` css
.slide {
  -webkit-perspective: 600;
  -moz-perspective: 600;
  -ms-perspective: 600;
  -o-perspective: 600;
  perspective: 600;
}

.inactiveslide .content {
  -webkit-transform: rotateY(180deg) rotateX(-60deg) rotateZ(20deg) scale(0.1);
  -moz-transform: rotateY(180deg) rotateX(-60deg) rotateZ(20deg) scale(0.1);
  -ms-transform: rotateY(180deg) rotateX(-60deg) rotateZ(20deg) scale(0.1);
  -o-transform: rotateY(180deg) rotateX(-60deg) rotateZ(20deg) scale(0.1);
  transform: rotateY(180deg) rotateX(-60deg) rotateZ(20deg) scale(0.1);
}
```

<p>Now our slides are facing backwards, tilted back, rotated left, and scaled down to one-tenth size when inactive. This is a pretty good start, especially considering it's only two extra style declarations. Obviously, we're not finished yet since we need to transition between the two states. Let's expand this a little:</p>

``` css
.inactiveslide .content {
  -webkit-transition: all 1000ms ease;
  -moz-transition: all 1000ms ease;
  -ms-transition: all 1000ms ease;
  -o-transition: all 1000ms ease;
  transition: all 1000ms ease;

  -webkit-transform: rotateY(180deg) rotateX(-60deg) rotateZ(20deg) scale(0.1);
  -moz-transform: rotateY(180deg) rotateX(-60deg) rotateZ(20deg) scale(0.1);
  -ms-transform: rotateY(180deg) rotateX(-60deg) rotateZ(20deg) scale(0.1);
  -o-transform: rotateY(180deg) rotateX(-60deg) rotateZ(20deg) scale(0.1);
  transform: rotateY(180deg) rotateX(-60deg) rotateZ(20deg) scale(0.1);

  opacity: 0;
}
.activeslide .content {
  -webkit-transition: all 2500ms ease;
  -moz-transition: all 2500ms ease;
  -ms-transition: all 2500ms ease;
  -o-transition: all 2500ms ease;
  transition: all 2500ms ease;
}
```

<p>With a few extra lines (excluding vendor prefixing), we've managed to set up two different transitions for activating and deactivating slides. Obviously, cleaning up this CSS with a pre-processor like <a href="http://sass-lang.com/" target="_blank">SASS</a> or <a href="http://lesscss.org/" target="_blank">LESS</a>, or <a href="http://lea.verou.me/" target="_blank">Lea Verou</a>'s <a href="http://leaverou.github.com/prefixfree/" target="_blank">-prefix-free</a> would be a good idea.</p>

<h3>Easing Into It</h3>

<p>Those of us who are used to the custom easing options when animating with jQuery might find the standard easing options with CSS transitions a little lacking. If we want to use one of the classic <a href="http://www.robertpenner.com/easing/" target="_blank">Penner easing methods</a>, made famous with Flash and jQuery (with <a href="http://gsgd.co.uk/sandbox/jquery/easing/" target="_blank">jQuery Easing Plugin</a>), one option is to use one of my favourite online developer tools, <a href="http://matthewlein.com/ceaser/" target="_blank">Ceaser</a>, which allows you to create a custom easing method or select from a set of existing well known easing equations. By selecting 'easeInOutExpo' from the drop down list, we can grab the CSS which provides us with our custom easing bezier curve.</p>

<p>If we use the custom easing in our code, it looks like this:</p>

``` css
.inactiveslide .content {
  -webkit-transition: all 1000ms cubic-bezier(1.000, 0.000, 0.000, 1.000);
  -moz-transition: all 1000ms cubic-bezier(1.000, 0.000, 0.000, 1.000);
  -ms-transition: all 1000ms cubic-bezier(1.000, 0.000, 0.000, 1.000);
  -o-transition: all 1000ms cubic-bezier(1.000, 0.000, 0.000, 1.000);
  transition: all 1000ms cubic-bezier(1.000, 0.000, 0.000, 1.000);

  -webkit-transform: rotateY(180deg) rotateX(-60deg) rotateZ(20deg) scale(0.1);
  -moz-transform: rotateY(180deg) rotateX(-60deg) rotateZ(20deg) scale(0.1);
  -ms-transform: rotateY(180deg) rotateX(-60deg) rotateZ(20deg) scale(0.1);
  -o-transform: rotateY(180deg) rotateX(-60deg) rotateZ(20deg) scale(0.1);
  transform: rotateY(180deg) rotateX(-60deg) rotateZ(20deg) scale(0.1);

  opacity: 0;
}
.activeslide .content {
  -webkit-transition: all 2500ms cubic-bezier(1.000, 0.000, 0.000, 1.000);
  -moz-transition: all 2500ms cubic-bezier(1.000, 0.000, 0.000, 1.000);
  -ms-transition: all 2500ms cubic-bezier(1.000, 0.000, 0.000, 1.000);
  -o-transition: all 2500ms cubic-bezier(1.000, 0.000, 0.000, 1.000);
  transition: all 2500ms cubic-bezier(1.000, 0.000, 0.000, 1.000);
}
```

<h3>See It In Action</h3>

<p>You can <a href="http://markdalgleish.com/examples/fathom3d/" target="_blank">see the final result here</a>. Obviously this is fairly basic, but works as a great starting point to expand on the idea further.</p>

<p>To see another example, I recently gave a presentations at <a href="http://www.webdirections.org/" target="_blank">Web Directions</a>' <a href="http://whatdoyouknow.webdirections.org/" target="_blank">What Do You Know</a> event in Melbourne using these techniques called <a href="http://markdalgleish.com/presentations/embracingtouch" target="_blank">'Embracing Touch: Cross Platform Scroll'</a>.</p>

<h3>Wrapping Up</h3>

<p>As you can see, it's really quite simple. A presentation makes for a great way to introduce yourself to the wonders of 3D positioning using CSS. The next time you're working on a Fathom.js presentation, why not get your hands dirty and try out some 3D transitions for yourself?</p>
