---
title: "The Case of the Crashing Checkout Flows: My Strangest Debugging Story"
date: 2024-08-18 15:54:34
keywords: binary search debugging, memory leak debugging, chrome devtools profiling
tags:
  - Development
  - Testing
---

The year was 2015, and I had just joined RedMart.

I was very impressed by the importance RedMart placed on addressing customer feedback. We had a customer feedback form, and the responses were sent to everyone's email inbox. We would triage the feedback, and if there were any software related issues, my team would pick them up.

At the end of that year, we started seeing a strange pattern in these feedback messages. Some users complained they couldn't complete checkout, while others reported their browsers crashing during the process.

*Note: If you're not familiar with the term "checkout flow", it's the series of pages you need to go through when you're ordering from an ecommerce shop. Like filling up your address, choosing your payment method etc.*

## Memory Leaks?

I started investigating this issue. One thing the feedback had in common was that they were all from MacBooks via Chrome browser (detected using useragent string). The [base model MacBook Air](https://support.apple.com/en-us/111956) at that time had 4GB of memory. I started suspecting it might be a memory leak.

In 2015, RedMart's desktop webapp (aka "Golden Grocer" or "GG" internally) was written in Backbone.js and jQuery. We had started incrementally migrating the pages to React, but the checkout flow was still in Backbone and jQuery.

I had some past experience debugging memory leaks in JVM, so I fired up the Chrome Devtools and [started profiling](https://developer.chrome.com/docs/devtools/memory-problems). I went through the checkout flow and found that there were about ~16MB of objects leaking memory. These were likely JS closures or Backbone/jQuery event handlers not cleaned up properly. But 16MB shouldn't be enough to crash an entire browser. 

In software debugging, there's a saying:

> If you can reproduce a bug consistently, you’ve already solved half the problem.

I’m not sure where I first heard this, but if you're not able to reproduce a bug, then you can't fix it properly. In this situation, I know it happened during checkout flow but I wasn't able to get the browser to crash. For a couple of days, I use the Chrome Devtools to go through the checkout flow again and again trying to see if I could get it to produce a big memory leak.

This didn't work out, so I wanted to try a different approach. Instead of using the fine-grained Chrome Devtools memory profiler, I opened the Chrome Task Manager. I noted down GG's memory consumption on the home page - it was around 300MB. I went through the checkout flow. The first page was address selection, the memory didn't budge much. The second was slots selection, memory remained the same. The third was payment selection, and the memory jumped to more than 1.5GB. 

*Bingo!*

Something in payment page was causing the memory to jump from 300MB to more than 1.5GB. But there's nothing special there in terms of code.

## Binary Search Debugging

Since there's nothing wrong with the code, we can't fix this issue using normal debugging techniques. We need to use a different approach. I've used this approach called "Binary Search Debugging" in the past and it's very effective at locating the bugs. 

Here’s how it works: We first divide our codebase into two parts. We comment out one part and check if we can reproduce the bug. If we can’t reproduce the bug, we then comment out the other part while uncommenting the first part. We then check if we can reproduce the bug. The bug is in the part that remains uncommented. Once we identify the problematic part, we further divide it into two and repeat the process. Eventually, we will pinpoint the specific line of code causing the bug.

This is very effective method and I've successfully used it many times in the past. But this time, I was not able to find the code that caused this bug. 

I have been using this approach on the JS code, but I wondered if it's a CSS issue. It sounds ridiculous, but the whole situation is ridiculous.

I did the same for CSS and I was actually able to find the bug! When a payment method is selected, we show a pulsing blue glow effect around that option. This was done using `box-shadow` and `animation`, and once I comment out the animation, the memory increase didn't happen. I searched the web for related issues but I was not able to find it.

Finally, I was able to fix the memory issue that blocked checkout by just disabling a CSS glow animation. It wasn't a code issue, it was a browser issue!

## Conclusion

It was an interesting experience that required a combination of both skill and luck to fix the bug. I had the memory profiling skill and knowledge about the binary search debugging technique. However, it was also random luck that made me open the Chrome Task Manager and apply the debugging technique to CSS files.

Thanks for reading! :)