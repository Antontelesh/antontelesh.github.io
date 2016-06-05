---
layout: post
title:  "Service Workers Stress Test"
date:   2016-06-05 16:50:00 +0300
categories: test
---

Ever wondered how much data can Service Workers cache?

Every time I see an article about Service Workers I don't see any limits they have.
But knowing your limits can make you feel confident about the tools you use.

So I decided to put Service Workers under a stress test.
How to perform such test?

I decided to cache images.
For that, I've written [a sample application](https://github.com/Antontelesh/sw-stress-test) which uses ReactJS for rendering.
ReactJS was chosen only for its simplicity.
I know that I can sketch up something quickly using it without thinking about the implementation.

I've chosen a library called [sw-toolbox](https://github.com/GoogleChrome/sw-toolbox) that allows setting up service workers cache with 10 lines of code, which is extremely cool!

My application allows viewing some pictures.
The UI consists of two parts: a list of pictures and a currently opened picture. The list of pictures displays info about the url of each picture and its loading status. This is useful for tracking loaded images. Click on the image opens it in the presenter panel.

On page load, all images start to download.
In order to be able to manage the priority of download, I created a queue.
It allows to download up to 4 pictures simultaneously and preserves resources to request some pictures out of order.
This provides us with the ability to preview any picture even at the end of the list when the queue is extremely long.

So the time for actual tests has come.
I put the first hundred files to the directory and run the application.
After all the images had been downloaded I turned off my web-server, reloaded the page and saw no corruption in app behavior. How happy I was! All the images were there, reachable and viewable.

So then I decided to increase the number of images.
I've doubled the size and run the app again.
And again all images have been cached.

I doubled the number of pictures again. And again all the same.
Service workers did their best and returned me cached responses for all the images. This time, there were more that 400 of them.

After increasing the number of images up to 900 I finally noticed network errors. This time, Service Workers managed to cache only 431 of my images. Fortunately, they were the latest requested ones.

BTW, it took my notebook around 20 minutes to fetch 900 pictures from my local web-server without any network throttling, but with the limit to 4 requests at a time. During all that time it was consuming up to 95% of CPU. But the memory allocation remained the same — around 600 Mb.

So, I've got 430 images 4.5 Mb each (approximately). It's around 2 Gb of space. That was the limit in my case.
I'm curious to check on other environments.
So if you're curious too you can [clone the repo](https://github.com/Antontelesh/sw-stress-test) and try it yourself.
I will be glad to see your results in the comments section below.

-------

This exercise helped me to conclude that Service Workers are extremely powerful and very easy to set up thanks to the libraries like [sw-toolbox](https://github.com/GoogleChrome/sw-toolbox).

If you want to cache your statics and assets Service Workers are a nice fit for you. They perform very well.

If you need to cache really big amount of data (not a usual case, but happens) try using `navigator.persistentStorage`, which is not so cool as Service Workers but does its job.
