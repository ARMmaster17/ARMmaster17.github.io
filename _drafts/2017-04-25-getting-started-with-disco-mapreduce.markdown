---
layout: post
title:  "Getting Started With Disco MapReduce"
date:   2017-02-17 13:00:17 -0500
categories: disco tutorial
---
A while back, I wanted to learn how to use Hadoop. I prefer to self-host my stuff as it's free (as in pizza), and free (as in speech).

I almost immediately ran into issues getting it set up. First off, it turns out Hadoop uses Java, which I hate. It also has insane memory requirements, which my homelab couldn't handle. To top it off, the installation docs were too confusing for something that I would use for 10 minutes and then never touch again.

Then I found [Disco](http://discoproject.org).

Unlike Hadoop, Disco has libraries for use in Python, which I find slightly more acceptable than Java. It also has an extrememly small footprint. I was able to run a testing worker with 256mb of RAM, 1 CPU, and 1 GB of hard drive space (including all of its dependencies).

The one remaining problem, is that following the installation instructions for Disco has probably been one of the most frustrating pieces of software I've ever had to install.
