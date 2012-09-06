---
layout: post
title: tmpload - Asynchronous Template Loading and Caching for jQuery Templates
date: 2011-04-06 08:26
comments: true
categories: []
---
<p>I'm very pleased to announce the release of my new jQuery plugin, <a href="/projects/tmpload">tmpload</a>.</p>

<p>It's no secret that I'm a big fan of client-side templating and the official <a href="https://github.com/jquery/jquery-tmpl">jQuery Templates</a> (aka tmpl) plugin is really quite fantastic.</p>

<p>I especially like the idea of templates in external files. Generally there are two ways that external templates are implemented:</p>

<ul>
<li>Including them in a script block, downloading them when the page loads; or</li>
<li>Downloading them in parallel when the matching AJAX request is made.</li>
</ul>

<p>Both of these are absolutely fine. However, I wanted more fine grained control over when the downloading of the templates happens. For example, I would sometimes like to download a template as soon as the user focuses on a search form. That way it's never downloaded if they don't use the search form, and if they perform a search, the template is already downloaded and cached as soon as they need it rather than downloading after they submit their search.</p>

<p>Templates can also be compiled and saved to a variable using the $.template function and, seeing as the biggest performance hit with client side templates is the initial parsing of the template, I really wanted template caching to be taken care of automatically.</p>

<p>Solving these two problems inline requires quite a bit of messy 'plumbing' code, decreasing readability and maintainability while discouraging best practices due to increased complexity.</p>

<p>This is where <a href="/projects/tmpload">tmpload</a> comes in.</p>

<p><a href="/projects/tmpload">Tmpload</a> aims to provide a dead-simple way to implement external templates that can be loaded and cached at will, with easily understandable calls to the $.tmpload function.</p>

<p>Declaring a template:</p>

``` js
$.tmpload('search', 'path/to/search.tmpl');
```

<p>Declaring groups of templates:</p>

``` js
$.tmpload([
  {
    name: 'search',
    url: 'path/to/search.tmpl'
  },
  {
    name: 'comments',
    url: 'path/to/comments.tmpl'
  }
]);
```

<p>Loading the template and caching it, or referencing it in an AJAX 'when' block:</p>

``` js
$.tmpload('search');
```

<p>That's all there is to it! Grab a copy and try it out yourself. I'd love to hear your feedback and suggestions.</p>

<ul>
<li><a href="http://github.com/markdalgleish/tmpload/archives/master">Download tmpload v1.0 from GitHub.</a></li>
<li><a href="/projects/tmpload">Read the beginner's guide at the official tmpload project page.</a></li>
<li><a href="http://github.com/markdalgleish/tmpload">If you'd like to contribute, fork me on GitHub.</a></li>
</ul>
