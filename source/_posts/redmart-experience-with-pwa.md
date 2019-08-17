---
title: RedMart’s experience with PWA
description: RedMart’s experience with PWA
keywords: PWA, Progressive Web App, Service Workers, Mobile Web
date: 2018-10-25 12:39:35
tags:
  - JavaScript
---

## What is a PWA?

Progressive Web App (PWA) is a term used for websites that behave like an app.

The phrase "behaves like an app" is different for different people. Some say an "app" is something that's installed on your device, for some it's the smooth and fluid user experience, for some it's the ability to work offline, for others it's the tight integration with OS like share menu, push notifications, widgets, 3d touch etc

Since the definition of an "app" is subjective, it's better to have some baseline criteria for PWAs so we know everyone's talking about the same thing. According to [Alex Russell](https://twitter.com/slightlylate), one of the people who coined the term PWA, a website is a PWA if it satisfies the following [criteria](https://infrequently.org/2016/09/what-exactly-makes-something-a-progressive-web-app/):

- Served over HTTPS
- Loads offline even without internet connectivity
- Contains a [Web App Manifest](https://developer.mozilla.org/en-US/docs/Web/Manifest)

## How it all started

At RedMart, we embarked on our PWA journey 2 years back. We didn't exactly set out to build a PWA since we had no idea what the benefits were. We started by adding [service workers](https://developers.google.com/web/fundamentals/primers/service-workers/) to cache some network requests in our mobile site. After this we realized that we actually satisfy 4 out of 5 [criteria](https://developers.google.com/web/fundamentals/app-install-banners/#criteria) for installable webapps. The only thing missing was a Web App Manifest. We quickly added that in 1-2 hours and bam, our mobile site is now a PWA!

## How does it work?

![](/images/2018-pwa/image-1.png)

## What were the benefits?

When we first launched this to public, we didn't know what to expect. Since the only extra cost was the development time to add the manifest, we weren't overly concerned about how it performed. When we looked at the data, the results were unbelievable:

> The ecommerce conversion rate and the time spent on the installed PWA is almost as good as our native Android app and many times higher than the same mobile site used in browser!

## How can we make this even better?

We still have a lot of room for improvement. The current mobile site is a lightweight version of the RedMart app experience and we're planning to add more features to improve the parity. This would increase the conversions and engagement. Adding push notifications for price drops, stock changes etc would help even more.

## Improving the install experience

The ability to install websites as apps is an interesting development in the last couple of years.

Browsers prompt users to install the website by showing a small [infobar](https://developers.google.com/web/fundamentals/app-install-banners/#the_mini-info_bar) or an [icon in the address bar](https://developer.mozilla.org/en-US/docs/Web/Apps/Progressive/Add_to_home_screen#How_do_you_use_it). While this is much better than asking users to go to settings and select "Add to Home Screen", this makes it harder for marketing to promote PWAs.

Native apps are usually promoted by having app install banners or popups in mobile site that when clicked on redirects the user to appstore to download the app.

It would be helpful to have apis that allows us to install the PWA. This seems to be a sensitive topic with a lot of [back and forth](https://github.com/w3c/manifest/issues/627) between developers and browser makers. Unfortunately, the app install popups look like they're here to stay, so it might be useful to provide the apis so we can decide to install PWAs if the device supports it and if not redirect the user to app store.

## Conclusion

The future is pretty exciting for PWAs! As of this writing, Chrome has released support for [desktop PWAs](https://developers.google.com/web/updates/2018/10/nic70#dpwa-windows) and Microsoft allows [submitting PWAs to Windows store](https://docs.microsoft.com/en-us/microsoft-edge/progressive-web-apps/microsoft-store#submitting-your-pwa-manually)

Each industry and userbase is different and you might see different results compared to us, but the barrier to entry is low and this can be quickly validated in a couple of days if you already have a mobile site. I hope this post was able to inspire you to adopt PWAs :)

Originally published in [geeks.redmart.com](http://geeks.redmart.com/2018/10/25/redmarts-experience-with-pwa/)
