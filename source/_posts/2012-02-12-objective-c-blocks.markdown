---

published: false

layout: post
title: "Objective-C Blocks"
date: 2012-02-12 21:47

comments: true
sharing: true
footer: true

categories: 
- Objective-C
- Blocks

---

Objective-C has seen a lot of great improvement over the years: `properties`, `arc` and most revolutionary `blocks`! More than any other addition blocks have totally changed the way I write code. When I first learned about blocks I wasn't sure what they could be used for, and the exiting blog posts didn't help much. That's why I decided to put this post together, most posts I found focused on using blocks in `c` and not much on using them in `objective-c`.

### What are blocks useful for?

I've coded a fair amount of JavaScript in my day and so I was familiar with closure style programing, due to JavaScript's popularity I'm guessing most people are familiar with this, even if they don't know it by name.

In JavaScript with jQuery you'd writing something like this to get a twitter feed:

    $.getJSON('http://api.twitter.com/1/statuses/user_timeline.json?&callback=?&trim_user=true&screen_name=_kgn', 
    function(tweets){
        // do something with the tweets
    });

In cocoa the same request would be written like this:

    http://api.twitter.com/1/statuses/user_timeline.json?&callback=?&trim_user=true&screen_name=_kgn
