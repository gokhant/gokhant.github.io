---
layout: post
title:  "51-Do you love Spring or hate it?"
date:   2016-02-22 06:02:05 +0200
categories: 
- programming
- java
- springframework
- batch

tags:
- spring
- batch
- java
- csv

---
Greetings! Recently I required to maintain huge (not that huge, approximately 500 MBs each) csv files; 
like reading them and doing something with them and creating some other files.
\\
\\
That's what we call [batch processing](https://en.wikipedia.org/wiki/Batch_processing). 
To some extend, I always try to stay away from [reinventing the wheel](https://en.wikipedia.org/wiki/Reinventing_the_wheel)
That's why I searched for some alternatives and [spring batch](http://projects.spring.io/spring-batch/) was one of them.
\\
\\
I gave it a shot and for the next few weeks, I will try to share my practice here. I last used 
spring like many years ago, where annotations were newly introuced (I think it was 2009 or 2010) 
and we were dealing with many xml files to configure our application. Even then, I liked the idea 
of dependency injection.

{:.center-image}
![Spring](/images/51/spring_logo.png "Spring Logo")

Before starting; let's first read from Sam Atkinson [why he hates spring?](http://samatkinson.com/why-i-hate-spring/)
And then, let's see what has Phil Webb said about [how not to hate spring?](https://spring.io/blog/2015/11/29/how-not-to-hate-spring-in-2016)
\\
\\
Spring has extensive guides for each of their projects in [here](https://spring.io/guides/) 
and you can always use [initializr](http://start.spring.io/) to create a maven project easily (*Switch to full version*)
\\
\\
There is also a nice guide for batch processing [here](https://spring.io/guides/gs/batch-processing/) and an introduction [here](http://docs.spring.io/spring-batch/reference/html/spring-batch-intro.html), 
so i will try to share the things which are not there for the next few weeks; 
such as multiple readers, writers and processors and additionally some performance figures.
\\
\\
Then, I will use a simple template engine to create some html pages which uses [google maps api](https://developers.google.com/maps/) 
and there will be [JSON](http://www.json.org) too.
\\
\\
How come I feel like campaign advertising like politicians?
\\
\\
Stay tuned!

