+++
title = "Amazon Builder's Library - An overlooked goldmine of System Design knowledge"
date = 2024-08-10T11:06:35-07:00
draft = true
+++

I've recently been spending some time improving my understanding of distributed systems.

YouTube is now the de facto way I learn new things, so that's where I naturally started. There are definitely some great channels on YouTube (see my recommendation list below), but the vast majority of videos related to this topic on YT are quite terrible. In general, they tend to be focussed on people who are preparing for System Design interviews and not for people who simply want to learn how to build better systems. This leads to videos that are shiny and appear useful, but barely scratch the surface in terms of what you need to actually implement those concepts in practice. In fact, even if your goal is to crack a bunch of interviews, a vast majority of these videos will only help you crack a SDE-2 level interview. They lack any kind of depth or nuance necessary for senior or staff level interviews.

There's another [resource](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/) that pops up a lot in these discussions: Designing Data-Intensive Applications by Martin Kleppmann or "DDIA" as it's fondly called. While this book comes up in the context of preparing for interviews, this book clearly doesn't fall into the same trap as those YT videos. As a senior or staff engineer, you'll find the content in this book very pragmatic and practical. I started it with a bit of skepticism, but ended up reading it cover-to-cover in the course of a week. This is no doubt a great way to learn about distributed systems in greater depth, but it requires a sustained time commitment over the course of a couple of weeks. I was still on the lookout for more bite-sized resources on these topics, so that I could consume them whenever I got a bit of free time.

Then, a couple of months ago, I came across the [Amazon Builders' Library](https://aws.amazon.com/builders-library/). I actually had it bookmarked many months ago with the intention of reading it later, but somehow I never did. I guess, the way it is organized doesn't really work for me: its a collection of articles on random topics (some of them interrelated), and it's not laid out in a logical order or sequence like a book.

This time, I decided to try and overcome my inertia to read a few articles in there. And now, I can't believe I did not do this sooner. These articles are a goldmine of practical knowledge you can apply day-to-day. The topics can be simple like dealing with timeouts/retries, to more complex ideas like dependency isolation. Not only are the articles available as a webpage, but also as PDFs that you can stash away with other papers and books in your collection. Some are even available in the Kindle format!.

Some of my favorite articles so far:
1. Using dependency isolation to contain concurrency overload ([Link](https://aws.amazon.com/builders-library/dependency-isolation/))
2. Reliability, constant work, and a good cup of coffee ([Link](https://aws.amazon.com/builders-library/reliability-and-constant-work))

And I've just barely begun to scratch the surface. There's a lot more I'm yet to read.

I suspect this is one of the most under-utilized resources for learning system design. In all my time watching YT videos on the topic and hanging around on Reddit, I've seldom seen the Builders' library mentioned. That's just baffling to me!. If you are a senior engineer or someone aspiring to be one, reading through the articles in there is one of the most effective things you can do to improve and level up.

### Appendix

While there's a lot of terrible system design videos on YT, there are actually a few great channels that are outliers and put out amazing content. Here's a list of my personal favorites:

1. [Asli Engineering, Aprit Bhayani](https://www.youtube.com/@AsliEngineering): What I like is that this channel never approaches any topic from the point of view cracking interviews. It's all about learning neat concepts and figuring out how things are really implemented deep-down. Any new video here instantly goes into my queue irrespective of whether the topic is relevant to me.
2. [Martin Kleppmann](https://www.youtube.com/@kleppmann): You can't go wrong learning from the author of DDIA. There haven't been any new videos in a while, but what already exists is quite useful. I finally understood how Raft works after watching his System Design lecture series.
3. [HelloInterview](https://www.youtube.com/@hello_interview): Obviously, this one is focussed on interviews, but somehow the actual content and depth of discussion is above the rest. It's probably because they try to also cater to staff level engineers.
4. [System Design Fight Club](https://www.youtube.com/@SDFC): Another one that is very clearly interview focussed, but the depth is far above what you find in other similar channels. Unfortunately, it hasn't had new videos in a while.
5. [System Design Interview](https://www.youtube.com/@SystemDesignInterview): This channel put out a bunch of videos a few years ago and then stopped. What I like about the videos that exist is that they teach a lot about tradeoffs between different designs. Especially, checkout the Top-K hitters video.
