---
layout: post
tags: project
title: AWS Jobs Environment
---

[This tool](https://github.com/connormurray7/aws-jobs-environment) offers an easy lightweight way to submit "jobs" from your local machine to an Amazon EC2 instance. This is not meant for queuing jobs in a production environment, but rather is a convenient tool for submitting jobs to another instance. Written in Python, this sets up a job queue on the AWS instance and has a simple way of submitting jobs from your local machine.

### _Why I made this_
Last week, I wanted to do some basic benchmarking between two versions of software. I wrote some tests that ran for a non-trivial amount of time and I didn't want to sit there all day and wait for them to finish. I also didn't want to have any of the other programs or things I'm doing on my laptop to interfere with the benchmark.

So I decided to create something that would allow me to automatically queue jobs on an AWS box so that it would just be one startup cost.

### _How it is implemented_
All of the code is written in python. There is a class that does a double-fork to daemonize the `run` process, and that run process is implemented to monitor to directory and then run any new jobs that are added to it.

Currently, all of the code is written to run sequentially, but the `ApplicationRunner` class that is in charge of queuing and executing jobs is written so that it would be very easy to implement this with multiple threads.
