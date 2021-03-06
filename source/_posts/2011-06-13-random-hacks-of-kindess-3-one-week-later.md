---
layout: post
title: "Random Hacks of Kindness #3: One Week Later"
date: 2011-06-13 03:18
comments: true
categories: []
---
RHoK #3 finished last Sunday and this week has been a busy week, I figured I'm overdue a rundown of how the weekend went and what I personally got out of it.

If you haven't heard of the event before, according to [the official RHoK website](http://http://www.rhok.org), "Random Hacks of Kindness is a community of innovation focused on developing practical open source solutions to disaster risk management and climate change adaptation challenges. Random Hacks of Kindness was founded in 2009 in partnership between Google, Microsoft, Yahoo!, NASA and the World Bank."

One of my team members, [Tony Milne](http://tonymilne.com.au), has already contributed [a fantastic technical recap of the VicAlert/VicSafe projects](http://tonymilne.com.au/posts/rhok-melbourne-vicalerts-and-vicsafe) on his blog so I won't attempt to write a similar article. In fact, you should probably read his blog post before reading this one. Rather, this post will cover my personal experiences being part of Melbourne's first RHoK event, which also happened to be my first hackathon.

Arriving at Swinburne Hawthorn's wonderful [Advanced Technologies Centre](http://www.swinburne.edu.au/chancellery/mediacentre/research/news/2011/02/atc-opens-for-business), a $140 million beacon of sustainable architecture housed in glass and metal, I felt dwarfed by the scale of the building around me, but equally welcomed by the friendly people working the front desk. After joining the other RHoKers in the auditorium, I wasted no time in opening up my laptop and connecting to the free wifi. Once the room started to fill up, we were given a briefing on how the day would commence and what to expect from the rest of the weekend.

Tables were arranged for the "four horsemen of the apocalypse", as they were dubbed: fire, earthquake, flood and 'general'. I, along with a couple of my fellow [MelbJS](http://groups.google.com/group/melbourne-javascript-jquery) attendees, [@benjaminpearson](http://twitter.com/benjaminpearson) and [tonymilne](http://twitter.com/tonymilne), walked over to the 'flood' table and began the initial discussion on the [problem definition](http://www.rhok.org/problems/confirm-flood-warning-safe-action-0) we had seen on the RHoK website.

After speaking with Stefan Willoughby from the Bureau of Meteorology and, particularly after hearing from some people from the ABC who have dealt first-hand with disaster response through community radio, we quickly realised that the problem of getting relevant information to those in need during an emergency was much bigger than we initially expected. Needless to say, the scope of our work extended quite a bit and we found our project split in two: VicAlert, a central alerting API based on the [Common Alerting Protocol](http://en.wikipedia.org/wiki/Common_Alerting_Protocol), and VicSafe, a sample JavaScript application for consuming those alerts. VicSafe was targeted at solving the problem definition, which was giving people a visual indicator of the danger they were facing as flood waters were rising around them.

The work environment at a hackathon was something completely new and intimidating to me. Tasked with creating a solution over two caffeine-fueled days with a team of people you haven't met before, especially when everybody's skills might not entirely mesh, is quite a tall order. In fact, early into the first day we lost two of our team members due to differing technical backgrounds, leaving them feeling like they wouldn't be able to make a worthwhile contribution.

Our project was mostly written in JavaScript, with the back end being written in Node.js. The front end was vanilla JavaScript interacting with the Google Maps API, with a healthy dose of jQuery/jQuery UI for interactivity and [Socket.io](http://socket.io/) for real time communication with the server. I became the de facto front end coordinator for VicSafe, ensuring that the disparate parts of code being created by myself and others on the team would coalesce into a usable whole. It must be said that such a working environment doesn't result in the highest quality of code, but I suppose code quality is of a lower importance when your projects have to be done in what basically amounts to a single day of coding.

[We coded through the day, winding down later in the night.](http://www.youtube.com/watch?v=gvKbHDF-Iko) To my surprise, at least in my case, the real breakthroughs seem to happen just as you're getting ready to leave. On the bright side, upon returning the next morning, the development speed completely picked up and I found that a good night's sleep did wonders for my mental clarity. With only hours until our code had to be presented to the other hackers, the final front end code and a decent amount of spit and polish created the presentation-ready version of VicSafe, which at this stage was interacting with the presentation-ready version of VicAlert using Socket.io.

Presenting to a crowd of 60+ developers was a daunting task. Public speaking isn't something I normally have too much difficulty with, but talking to an auditorium of developers was a little more nerve wracking than I was expecting. Luckily [@benjaminpearson](http://twitter.com/benjaminpearson) did most of the presenting and he did a great job. The VicAlert/VicSafe presentation was very well-received and [the slides are available now on SlideShare](http://www.slideshare.net/BenPearson/vicalert-vicsafe).

The projects created by all teams over the weekend were really impressive:

 * [QuakeFelt](http://www.rhok.org/solutions/quakefelt): Android app/jQuery Mobile website that allows users to report earthquake intensity and building damage.
 * [MyPlan](http://www.rhok.org/solutions/myplan): Website for creating a customised disaster response plan, with particular attention paid to preparing a medicine kit for those who require medication - "Prepare, Print, Survive".
 * [Bushfire Connect](http://www.rhok.org/problems/bushfire-connect): Upgrades to usability of website and maps, particularly full screen maps and automatic geolocation.

The response for VicAlert/VicSafe was especially positive and, even though the focus of the event wasn't on teams trying to win, the judges awarded our team with the top prize, saying that a centralised alerting system for Victoria has the potential to be a "game changer".

One week on from the event and some positive things have come from this for me, personally. I've made some new friends, been able to work alongside existing friends for the first time, and I've been able to contribute to a project that has the potential to be a vital tool for all Victorians. We're all still waiting with bated breath for the opportunity to present our proof of concept to those who will be able to give it the proper attention and funding to move the project forward. While what we created over the course of a single weekend was certainly a powerful reminder of what can be done when you put a group of experts in a room with a short time limit, the VicAlert and VicSafe projects merely scratch the surface of what needs to be done to turn this important idea into a reality.

In fact, shortly after the RHoK event, [ABC published a story covering the VicAlert/VicSafe projects](http://www.abc.net.au/local/stories/2011/06/06/3236750.htm) and recognising the potential for a future where the alerting systems were unified around a central platform.

I'd like to say a special thank you to [@benjaminpearson](http://twitter.com/benjaminpearson), [@tonymilne](http://twitter.com/tonymilne), [@wolfeidau](http://twitter.com/wolfeidau) and [@sivaone](http://twitter.com/sivaone) for making the weekend so enjoyable. You're truly a smart bunch of people and I look forward to seeing what you guys have to contribute at RHoK #4.

The code for VicSafe is available on GitHub: [github.com/wolfeidau/VicSafe](http://github.com/wolfeidau/VicSafe).
