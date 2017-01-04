---
layout: post
tags: project
title: Link Aggregator
---

Link Aggregator is a website that lives at [link-aggregator.top](http://link-aggregator.top). It takes a search term, and displays the top results from Github, StackOverflow, and Hacker News (more integrations in the future). It is written in Python with Flask, Gunicorn, and Nginx. See [this article](http://connormurray.me/Deploying-Python/) for how it is deployed. The code is all available [here](https://github.com/connormurray7/link-aggregator).

### _What is it for?_
When trying to learn about new topics in software engineering I always would be visiting a bunch of the same sites. Github to see some projects, Hacker News to find some good blog posts/articles on the subject, and Stack Overflow to see some small examples. Instead of Googling around every time, I used the APIs from each of sites to aggregate all of the information.

### _Logging_
Logging is a first class citizen in Link Aggregator. The more you know about the requests you're getting, the better you can trouble shoot and identify your problems. Link Aggregator uses a RotatingFileHandler if the files start to get too large. As discussed in [my other post](http://connormurray.me/logging/), Link Aggregator passes around a handler in order to pipe all of the logging to the same file without rewriting the same code.

### _Caching_
Link Aggregator uses a simple LRU cache to store the most recent searched items. It holds the responses in memory because it doesn't need to write anything to a database, the application can always make another API request if it goes down and then has to start back up.

The cache data structure inherits from  `collections.OrderedDict`  in the Python Standard Library. This allows for efficient eviction without needed to loop through every item in the cache.

### _Rate Limiting_
Sometimes caching is not enough, Link Aggregator also limits the amount of requests it makes to the API's it calls. If there are more than x/sec (20 as a default based on the API with the least allowed/sec) then it will not execute the request.

This does not apply to items that are already in the cache (100 requests for "eventual consistency" per second will not be limited because the cache can serve those requests without API calls).

### _Configuration_
