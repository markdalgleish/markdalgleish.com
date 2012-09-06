---
layout: post
title: The Controversial Future of URLs
date: 2011-03-06 05:16
comments: true
categories: []
---
Most notably with upgrades to Twitter and Gawker, the use of "#!", or hashbang, in a website's URL has come to the forefront of emerging front-end development trends. The feedback, even from those who consider themselves on the cutting edge, often lands on the negative side of the argument. It's been urged by many in the industry that the hashbang breaks the basic assumption of how a URL functions.

Take, for example, the following URL for my twitter feed: <a href="http://twitter.com/#!/markdalgleish">http://twitter.com/#!/markdalgleish</a>

Typically, the hash in a URL is purely used as a way to link to a certain section within a single page. Because of this, anything following a hash in the URL is not sent to the server. When you click the link to my Twitter feed, your browser is really making a request to <a href="http://twitter.com/">http://twitter.com/</a>. The server responds with a bunch of HTML and JavaScript that serves as a client side app. If you view the source of the page you won't see a single tweet. Everything following the hash ("!/markdalgleish") is in fact passed into the JavaScript on the client side, which in turn triggers an AJAX request for the corresponding data.

Because of this architecture, any bot, crawler or parser that attempts to read from <a href="http://twitter.com/#!/markdalgleish">http://twitter.com/#!/markdalgleish</a> will not see my tweets. It will, in fact, receive code and markup for an application with no data.

Luckily, there is a bright side.

Twitter is actually adhering to a <a href="http://code.google.com/web/ajaxcrawling/docs/getting-started.html">URL standard supported by Google</a>. When the Google crawler sees the URL to my Twitter feed and detects the hashbang, it instead makes a request for:
<a href="http://twitter.com/?_escaped_fragment_=/markdalgleish">http://twitter.com/?_escaped_fragment_=/markdalgleish</a>.
If you click this, you'll see that is a valid link .

Google's web crawler will, assuming you follow their '_escaped_fragment_' querystring format, successfully traverse your site and index your pages correctly. However, any crawler that hasn't been updated to support the hasbang URL standard will fail to navigate standards-based, AJAX-reliant sites.

So where do I stand on this issue?

Over the past decade, web "sites" have quickly turned into web "apps" and the concept of a site being a series of pages is no longer appropriate for many of the most popular sites on the internet. As long as <a href="http://code.google.com/web/ajaxcrawling/index.html">Google's AJAX-crawlable URL standard</a> is followed correctly, I feel that the hashbang URL format is a major breakthrough that allows sites to finally embrace the benefits of AJAX to their full extent. No longer will we need to re-download core parts of a site's user interface every time we click a link, watching the site disappear only to reappear seconds later.

However, it's also clear that migrating to a purely client-side navigation can alienate large portions of a site's audience and, if implemented incorrectly, can render the site useless for anyone with JavaScript disabled. <a href="http://www.reddit.com/r/programming/comments/fhzyk/breaking_the_web_with_hashbangs_lifehacker_along/">Gawker's redesign was particularly notorious</a>. It's important that the site works without JavaScript so that those with legacy browsers, or those who are running plugins such as NoScript, can access your site correctly.

Sometimes upgrades can be unnerving for users and cause problems for legacy software, and this is always followed by outcry from the more steadfast among us. Changes of this nature will only become more prevalent, especially as the browser wars are heating up and JavaScript performance is going through the roof. Ever since Microsoft introduced the <a href="http://en.wikipedia.org/wiki/XMLHttpRequest">XMLHttpRequest</a>, this has been an inevitability and one I think is long overdue.
