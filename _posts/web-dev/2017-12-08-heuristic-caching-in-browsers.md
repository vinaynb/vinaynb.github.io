---
layout: post
title: Heuristic Caching in Browsers
description: "Relatively unknown fact about how browsers may cache your assets without you explicitly instructing them to."
modified: 2017-12-09
category: articles
tags: [web-dev,browsers,caching]
comments: true
share: true
---

Ever found yourself in a situation when your assets are being cached by browsers without you explicitly instructing them to ? Read on this article is for you then.

Normally there are 3 scenarios possible when working with browser caches and your static assets.

- First is when you instruct browser to cache your assets for certain amount of time via [cache-control](https://devcenter.heroku.com/articles/increasing-application-performance-with-http-cache-headers#cache-control) or [Expires](https://devcenter.heroku.com/articles/increasing-application-performance-with-http-cache-headers#expires) headers in response from your server . Browser keeps serving requests for those assets from its cache without hitting the network.
- Second is when you wish to use browser cache for certain amount of time as mentioned above but when that time expires you wish to re-validate the copy of resource in browser cache with your origin server. This happens with [etags headers](https://devcenter.heroku.com/articles/increasing-application-performance-with-http-cache-headers#cache-control) and it's fallback [last Modified](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Last-Modified) headers.
- Third is when you do not wish to cache your assets at all at any point of time and you wish to hit the network for all your requests. This happens with <code>cache-control:no-store</code> header.

We were recently working with a application that was being served from Aws's CDN service Cloudfront at work and while performing regular deployments i noticed that the clients at times did not receive newer versions of files as intended. We use dockers for deployment and we checked if all frontend dockers had the latest versions which it had. The Cloudfront cache invalidation was also being perfectly invalidated and it we tested it by CURLing one of the static assets. So as all seemed good at the backend, it was time to visit the client and open devtools to see what was going on here.

When inspecting the network pane for requests made by browser for the stale resource in question i found a strange behaviour. After fetching it once the browser was fetching the file from its own cache without even validating with the server if the copy it had was fresh. This was not what we expected the browser to behave as we did not instruct it in any way to cache the response by providing the requested headers it needs from server. Below is the screenshot showing the first request on right and the subsequent request served from cache on left.

![Request headers side by side]({{ site.url }}/images/posts/heuristic--caching-1.jpg){: .article-content-img }

As visible in the image above on the left is the request with <code>200 OK</code> status code and i made that request using no-cache option on from devtools so as to hit my origin server. It fetches the original file bypassing the cache as expected. Notice the <code>Last-modified</code> header in its response.

On the right is the request which i made immediately after few seconds of the previous request for the same resource and chrome weirdly fetches it from disk cache without hitting the network at all.

This can prove disastrous for us in production as we are not guaranteed that users will see a updated version of our application once we perform a deployment. A horror story plot if you are publishing a critical hotfix !

I then decided to re-read about how the browser caches assets and the headers required in search of some wisdom as i felt there was something that was being sent from our origin servers which was instructing browser to cache assets. But i did could point it out (turns out i should have read the [w3c spec](https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html) for caching more keenly).

Then after meddling for about an hour on Stackoverflow it struck ! I read about something called Heuristic Expiration mentioned in the [HTTP Caching Spec](https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html) and that was it. It was the reason why Chrome was magically caching my assets without me instructing it to do so. Basically what the spec says that if your servers do not explicitly tell the browser (by not setting appropriate response headers) when a resource expires the browser calculates a certain time based on a heuristic algorithm and decides to serve subsequent request for that resource from its cache until that time expires.

Let's revisit the example of my two requests above it better understand it. The first request on right has a response header <code>Last-modified</code> with value of 08 Dec 2017 11:27:23. The request on right has date header with value of 09 Dec 2017 13:35:12. The heuristic algo that chrome uses to calculate the time is **(date-modified - date) * .10**

Using above formula if we substitute the values we get time duration of 156.8 minutes. This means that for the next 156 minutes any request that browser receives for that request, it will instantly serve it from the cache. On the 157<sup>th</sup> minute browser will go to the server to fetch the resource again as the copy of resource it has in it's cache is stale now.

Point worth noting is that browsers will apply generally apply heuristic caching only if there is <code>Last-modified</code> header from your server (i have not verified this fact practically). If you are concerned about the formula different browsers use according to various forums and SO threads such as [this](https://stackoverflow.com/questions/14345898/what-heuristics-do-browsers-use-to-cache-resources-not-explicitly-set-to-be-cach) most of the popular browsers work on the same formula that we saw above. In that SO question there are links to actual browser source code where you can verify this fact.

One another fact that about [Date](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Date) header that i initially didn't understand initially is that Date header stands for the date and time when the resource was fetched. It does not mean that it will necessarily have the value of date time when you make the request for a resource from your browser. In my case as i used a CDN the date value in request/response is the date time when Cloudfront actually made the request to my origin (in my case S3). Hence once Cloudfront caches a resource all my next subsequent requests will hit cloudfront cache and all those requests will have the same <code>Date</code> header value as Cloudfront is actually not making any requests to my origin.

### Conclusion

In order to avoid forcing browsers invoke heuristic freshness value calculation it is recommended you set up your origin servers to have either <code>Cache-control:max-age value-in-seconds</code> or <code>Expires: value-in-seconds</code> header so that browsers know exactly when they should consider a resource stale and hit the network. If you are working with S3 and using Cloudfront like me, you will need to add required headers as [meta-data to your S3 objects](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingMetadata.html#object-metadata) a guide to which can be found [here](http://docs.aws.amazon.com/AmazonS3/latest/user-guide/add-object-metadata.html).

Still not working for you ? Trouble understanding or implementing caching ? Hit me up in comments or on twitter([@vinaynb](https://twitter.com/Dilemmist)). Thanks for reading :)
